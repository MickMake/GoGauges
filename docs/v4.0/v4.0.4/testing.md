# GoGauges Core testing

GoGauges Core uses standard Go tests, deterministic clocks, fake MQTT seams and race detection. Tests verify consumer behaviour without requiring GoDriveLog, vehicle hardware or actual gauge rendering.

## CI baseline

```text
go test ./...
go test -race ./...
go vet ./...
```

Fuzz contract parsing and topic parsing where useful, but do not make a separate production verification service.

## Contract and MQTT boundary tests

Cover:

- valid status, catalogue and state messages
- topic `provider_id` versus payload mismatch
- unsupported contract version
- invalid health values
- malformed JSON and missing required fields
- payload-size limits
- malformed-message isolation between providers
- no command, control, raw-CAN or RPC topics

The tests refer to the authoritative contract under `docs/v4.0/v4.0.2/contracts/`. They do not embed a competing handwritten specification.

## Retained discovery tests

Verify:

- cached providers start offline
- retained status and catalogue discover providers after subscription
- retained catalogue does not imply online state
- retained messages may arrive in either order
- discovery does not create, remove or rearrange dashboards

## Reconnect tests

Verify:

- reconnect and resubscribe to all three wildcard topics
- retained status and catalogue are reprocessed idempotently
- no historical state backlog is expected
- latest complete state restores current values
- GoGauges remains ready while MQTT is disconnected
- MQTT disconnection is exposed separately from provider online/offline state and signal freshness
- temporary MQTT disconnection does not itself force retained values stale
- retained values continue ageing from `observed_at` during transport disconnection

## Provider conflict tests

Cover:

- two live `instance_id` values for one `provider_id`
- immediate conflict marking
- state from both instances blocked while conflicted
- no silent merge or winner selection
- recovery after explicit offline status
- recovery after a configurable single-instance conflict-recovery timeout
- provisional five-second default can be changed without altering conflict semantics
- sequence reset for the surviving instance
- no replay of state accumulated during conflict

Use deterministic clocks. Do not encode five seconds as an architectural invariant in tests.

## Catalogue tests

Cover:

- first catalogue acceptance
- higher-revision replacement
- duplicate same revision and content
- rejection of same revision with different content
- rejection of lower revision
- same revision accepted across a clean instance restart when content matches
- removed signals becoming unavailable
- preserved user bindings after removal
- compatible signal rediscovery
- incompatible returning signal type
- cache load, dirty tracking, 30-second flush and atomic replacement
- corrupt or unsupported cache ignored without blocking startup

## Complete-snapshot tests

Cover:

- atomic provider snapshot replacement
- duplicate sequence ignored
- old sequence rejected
- sequence reset across `instance_id` change
- state received before status or catalogue
- only the newest pending state retained per instance
- pending state discarded on conflict, offline or supersession
- no partial application after any signal validation failure

During the final contract consolidation, add explicit tests for the accepted proposed exact-key clarification:

- state keys exactly match catalogue IDs
- missing key rejects the whole snapshot
- additional key rejects the whole snapshot
- removed catalogue signal disappears from the next snapshot
- unobserved, disabled and invalid signals remain present

Until consolidation, mark these as tests for the proposed clarification rather than claiming an already normative contract rule.

## Signal-state tests

Cover:

- typed number, boolean, string and enum values
- unobserved state
- disabled state
- invalid state without a retained value
- invalid state with a retained last-good value
- invalid state preserving the original `observed_at`
- all four external health values: `ok`, `degraded`, `error`, `unknown`
- provider offline and conflict dimensions remaining separate from signal health
- Core retaining current state without accumulating general per-signal sample history
- widgets or UI consumers not requiring every MQTT sample from Core

## Staleness tests

Use an injected deterministic clock. Do not use arbitrary sleeps.

Cover:

- never-observed freshness `unknown`
- online signal becoming stale at `stale_after_ms`
- omitted `stale_after_ms` producing `unknown` time freshness
- explicit provider offline state making retained values stale where applicable
- MQTT transport disconnection not itself rewriting freshness
- retained values continuing to age from `observed_at` while MQTT is disconnected
- disabled signal remaining freshness `unknown`
- invalid retained value ageing from its original timestamp
- future provider timestamp clamping and clock-skew diagnostics
- one timer servicing many signals
- notification emitted only when freshness state changes

## Binding tests

Cover:

- durable identity using `provider_id + signal_id`
- no `instance_id` in user configuration
- provider absent at startup
- provider offline
- signal removed and binding preserved
- compatible signal return resolving automatically
- incompatible type return remaining unresolved
- discovery not rewriting presentation configuration

## UI-boundary tests

Verify:

- immutable snapshots do not expose mutable Core maps
- monotonically increasing Core revision
- coalesced notifications allow slow consumers to fetch latest state
- slow consumers do not block MQTT callbacks or Core mutation
- no MQTT payloads, topics or GoDriveLog details cross the UI boundary
- Core snapshots expose current state rather than an implicit unbounded sample history

## Queue and pressure tests

Cover:

- bounded status/catalogue control queue
- state update storm coalescing to latest snapshot
- queue-pressure diagnostics when control capacity is reached
- no unbounded memory growth
- MQTT callback execution does not block indefinitely on a full control queue
- status/catalogue messages are not silently discarded without diagnostics
- chosen simple overflow implementation obeys cancellation and boundedness
- superseded control-message replacement occurs only where semantic equivalence is proven
- malformed-message storms remaining isolated

Exact control-queue capacity and enqueue timing are implementation tuning values and should not be hard-coded as architecture tests.

## Persistence tests

Cover:

- platform config and cache path selection
- explicit atomic user-configuration writes
- catalogue cache temp-file replacement
- dirty flag cleared only after successful replacement
- failed cache write leaves dirty state for retry
- synchronous shutdown flush
- no persistence of live values, online state, instance ID, sequence or freshness
- no persistence of general per-signal sample history in Core v1

## Race detector focus

Race tests should focus on:

- MQTT callback handoff
- Core owner command processing
- immutable snapshot publication
- coalesced notification delivery
- persistence snapshot handoff
- startup and shutdown cancellation
