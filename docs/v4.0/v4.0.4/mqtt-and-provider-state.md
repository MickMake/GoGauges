# MQTT and provider state

## Contract authority

Topic structure, payload fields and delivery semantics come from the authoritative files under:

```text
docs/v4.0/v4.0.2/contracts/
```

This document defines consumer implementation behaviour. It does not duplicate or replace the contract.

## MQTT client ownership

One MQTT consumer component owns:

- client construction
- connection and reconnect
- subscription setup
- topic parsing
- payload-size limits
- JSON decoding
- contract validation
- delivery into Core

The consumer does not publish command, control, raw-CAN or RPC topics.

## Subscriptions

GoGauges subscribes to the existing wildcard topics:

```text
gogauge/v1/providers/+/status
gogauge/v1/providers/+/catalog
gogauge/v1/providers/+/state
```

Status and catalogue are retained. State is non-retained and contains complete provider state.

## Consumer Last Will

GoGauges configures no application-level Last Will in version 1. The shared contract defines provider presence, not consumer-presence topics.

## Message validation and isolation

Before a message reaches mutable Core state:

- the topic must match one of the three known forms
- the topic `provider_id` must equal the payload `provider_id`
- `contract_version` must be supported
- required fields and value types must validate
- external health must be one of `ok`, `degraded`, `error`, `unknown`
- identifiers and payload size must satisfy implementation bounds

A malformed message is rejected in isolation. It does not partially update Core and does not affect another provider.

## Bounded handoff

### Status and catalogue

Validated status and catalogue messages use a bounded FIFO control queue. Queue pressure is diagnosed. The implementation must not silently create an unbounded queue.

### State

Validated state messages are stored as the latest pending snapshot for each:

```text
provider_id + instance_id
```

A newer sequence replaces an older pending snapshot. A capacity-one wake notification tells Core that state is ready. Intermediate complete snapshots may be coalesced.

## Provider registry

The provider registry is keyed by durable `provider_id`.

For each provider it tracks:

- descriptive provider metadata
- reported online and enabled state
- external health and message
- current accepted `instance_id`
- candidate runtime instances
- last status receive time
- last state activity time
- conflict state
- last accepted state sequence
- whether live catalogue evidence exists
- whether cached catalogue metadata exists

Cached metadata never proves that a provider is online.

## Provider instance changes

A clean instance transition occurs when the old instance is offline or expired and a new instance becomes the accepted live instance.

On acceptance of a new `instance_id`:

- reset state-sequence tracking
- preserve durable provider identity and user bindings
- preserve compatible catalogue metadata
- clear previous live signal values to unobserved
- wait for a valid complete snapshot from the new instance

State from different instances is never merged.

## Conflicting instances

Overlapping live activity from different `instance_id` values using one `provider_id` creates a conflict.

While conflicted:

- mark the provider conflicted
- preserve the last accepted signal snapshot only as historical current-state memory
- expose conflict to bindings and UI snapshots
- stop applying state from all conflicting instances
- do not choose a winner by sequence, UUID, arrival time or connection order

A conflict clears:

- immediately when all but one instance explicitly report offline; or
- after only one instance remains active for five seconds

When conflict clears, accept the surviving instance, reset sequence tracking and wait for or apply only its current valid complete snapshot. Snapshots received during conflict are not replayed as a backlog.

## Catalogue registry and cache

A catalogue is one complete provider-owned metadata document.

For the same provider:

- higher revision: validate and replace
- same revision and identical content: treat as duplicate
- same revision with different content: reject as inconsistent
- lower revision: reject as rollback unless a future contract revision explicitly permits it
- new instance with same revision and content: accept as provider restart

When a catalogue removes a signal:

- remove it from the active catalogue
- mark its signal unavailable
- preserve user bindings
- allow automatic resolution if the same stable signal returns compatibly

## State arriving before status or catalogue

Retained delivery order is not assumed.

If state arrives before matching status and catalogue:

- keep only the newest pending state for that `provider_id + instance_id`
- do not apply it
- apply it once matching status and catalogue establish the instance
- reject it if it does not validate against the catalogue
- discard it if the instance becomes conflicted, offline or superseded

## Complete-snapshot processing

The currently authoritative contract says state messages are complete snapshots. During Chat 4, an exact key-set rule was accepted as a proposed clarification pending final consolidation; see `OPEN-QUESTIONS.md`.

Core processing assumes replace-not-patch semantics:

- validate provider and accepted instance
- validate sequence
- validate every state entry and signal value type
- validate the snapshot against the current catalogue
- replace provider signal state atomically

If any required validation fails, reject the entire snapshot. Never partially apply a provider snapshot.

## Duplicate and out-of-order state

Within one accepted `instance_id`:

- sequence greater than last accepted: accept
- sequence equal to last accepted: duplicate, ignore and count
- sequence lower than last accepted: reject as old/out of order

Sequences are never compared across different instances.

`published_at` is retained for diagnostics only. Signal freshness uses each retained value's `observed_at`.

## Signal store

The signal store is keyed by:

```text
provider_id + signal_id
```

Each signal view preserves:

- catalogue availability
- retained typed value, if any
- `observed_at` associated with that value
- `valid`
- `enabled`
- external signal health and message
- effective provider online state
- provider conflict state
- accepted provider instance
- last accepted snapshot sequence
- snapshot `published_at` for diagnostics
- consumer-calculated freshness

No widget animation, rendering cadence or gauge physics belongs in the signal store.

## Invalid retained values

When a signal is invalid but includes the last good value:

- retain the supplied value
- retain its original `observed_at`
- expose `valid: false`
- expose the provider health and diagnostic message
- calculate freshness from the original value timestamp

An invalid observation must not make an old value appear fresh.

## Staleness

Freshness is a separate consumer-calculated value:

```text
unknown
fresh
stale
```

Rules:

- never observed: `unknown`
- disabled: freshness remains `unknown`
- no `stale_after_ms` while provider is online: `unknown`
- age exceeds `stale_after_ms`: `stale`
- provider offline or MQTT connection unavailable:
  - retained values become `stale`
  - unobserved values remain `unknown`
- invalid retained values still age from their original `observed_at`

### Timer strategy

One Core timer scans freshness deadlines. There is no timer or goroutine per signal.

On state receipt, Core compares wall time with `observed_at`, derives the remaining freshness duration and records a monotonic deadline. Later checks use monotonic time.

A provider timestamp materially in the future is clamped for scheduling and reported as clock skew. It must not remain fresh indefinitely.

Tests use an injected clock and controlled timer advancement.
