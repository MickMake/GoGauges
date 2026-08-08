# GoGauges Core architecture

## Purpose

GoGauges Core owns the application machinery between validated MQTT messages and later dashboard or widget code.

```text
MQTT callbacks
     |
     +--> bounded status/catalogue queue
     |
     +--> coalescing latest-state slots
                    |
                    v
          one Core owner goroutine
     +--------------+---------------+
     |              |               |
     v              v               v
provider registry  catalogue       signal store
                  registry/cache    + staleness
     +--------------+---------------+
                    |
                    v
             binding resolution
                    |
                    v
          immutable Core snapshot
                    |
                    v
         dashboard/widget boundary
```

Core understands only shared contract concepts: provider identity, catalogue definitions, typed state, validity, enabled state, health, online state, conflict and freshness. It contains no vehicle-acquisition or rendering knowledge.

## Suggested package shape

```text
GoGauges/
├── cmd/
│   └── gogauges/
├── contract/
│   └── Go implementation of the existing authoritative contract
└── internal/
    ├── app/
    ├── config/
    ├── mqttconsumer/
    ├── core/
    │   ├── providers.go
    │   ├── catalogues.go
    │   ├── signals.go
    │   ├── freshness.go
    │   ├── bindings.go
    │   └── snapshots.go
    └── persist/
```

This is a direction, not a requirement to create empty packages or files before implementation needs them.

## Ownership and concurrency

### `internal/app`

Owns process startup, readiness, shutdown and component supervision. It constructs concrete components and starts a small number of goroutines.

### `internal/config`

Loads and validates user configuration. Validated configuration is immutable after construction. Dashboard configuration and bindings remain user-owned.

### `internal/mqttconsumer`

Owns the MQTT client, reconnect, subscription setup, topic parsing, payload bounds, contract validation and delivery into Core.

MQTT callbacks do not mutate Core maps directly.

### `internal/core`

One goroutine owns all mutable provider, catalogue, signal and binding-resolution state. It processes bounded commands and publishes immutable snapshots.

Provider registry, catalogue registry, signal store and bindings remain distinct domain concepts even though they initially share one package and owner.

### `internal/persist`

Owns atomic file writes and cache loading. Persistence never becomes online-state authority.

## Application lifecycle

### Startup

1. Parse startup options and locate user configuration.
2. Load and structurally validate dashboard configuration and bindings.
3. Load the catalogue cache.
   - missing cache: start empty
   - corrupt cache: report and ignore
   - unsupported version: report and ignore
4. Construct Core state.
5. Insert cached provider metadata and catalogues.
6. Mark all cached providers offline and catalogues cached, not live-confirmed.
7. Resolve bindings against known cached definitions.
8. Publish the initial immutable Core snapshot.
9. Start the MQTT consumer.
10. On connection, subscribe to the three authoritative wildcard topics.
11. Process retained discovery messages and subsequent live state.

### Readiness

GoGauges becomes ready after valid configuration, Core construction and availability of the UI boundary. MQTT connection and live providers are not readiness prerequisites.

### No provider available

No-provider operation is normal:

- cached catalogue definitions remain visible
- user bindings remain preserved
- current values are unavailable or unobserved
- no dashboard is created, deleted or rearranged
- diagnostics report zero live providers

### Reconnect

After reconnection:

1. resubscribe to all three wildcard topics
2. process retained status and catalogue messages
3. wait for the next complete non-retained state snapshot
4. do not request or replay a historical state backlog

MQTT connection state is a separate transport fact. A temporary consumer disconnect does not by itself rewrite signal freshness; retained values continue to age from their existing `observed_at` values until provider status and new state are learned again.

### Shutdown

1. mark shutdown in progress
2. stop accepting new MQTT callback work
3. disconnect or stop the MQTT consumer
4. drain accepted status/catalogue commands
5. apply the latest already-accepted coalesced state
6. publish one final immutable Core snapshot
7. flush dirty user configuration if explicitly changed
8. synchronously flush the catalogue cache
9. stop the Core owner goroutine

Shutdown is bounded and idempotent. Cache failure is reported but does not rewrite user configuration or invent live state.

## Current-state-only signal ownership

GoGauges Core version 1 stores current signal state only.

It does not retain a general per-signal sample history for graphs, traces or widget effects, and later display code must not assume every MQTT sample is available from Core.

If a later graph or history feature genuinely needs historical samples, that work may introduce a separate bounded history component with explicit ownership and retention policy. That component is not part of Core v1.

## Immutable Core snapshots

A `CoreSnapshot` is a complete read-only view containing:

- MQTT connection summary
- provider snapshots
- catalogue information
- current signal state
- binding resolution
- diagnostics summary
- monotonically increasing Core revision

Core snapshots contain ordinary copied Go values. Readers do not receive pointers into mutable Core maps.

Change notifications are coalesced signals meaning “a newer Core snapshot is available.” Consumers fetch the latest snapshot rather than processing every intermediate update.

## Persistence

### User-owned configuration

Persist dashboard configuration and bindings in the platform user-config directory. Writes are explicit and atomic. Discovery never silently rewrites this file.

### Catalogue cache

Persist application-managed provider metadata and complete catalogues in the platform user-cache directory.

Cache content may include:

- cache format version
- provider identity and descriptive metadata
- last known complete catalogue
- catalogue revision and optional definition hash
- last discovery time

Do not cache as authority:

- provider online state
- current signal values
- active `instance_id`
- state sequence
- freshness result

Dirty cache state is flushed every 30 seconds, on `SIGHUP`, and synchronously during shutdown. Writes use temp-file replacement. A failed write leaves the cache dirty for retry.

## Health and diagnostics

Core diagnostics remain lightweight and queryable through immutable snapshots and structured logs.

Track at least:

- MQTT connected state and reconnect count
- subscription or resubscription failure
- malformed messages by topic and reason
- provider conflicts and recoveries
- rejected old or duplicate snapshots
- unknown or mismatched signal keys
- stale provider and stale signal counts
- catalogue-cache load and flush failures
- unresolved and incompatible bindings
- control-queue depth, queue-pressure events and state coalescing count

Diagnostics report conditions; they do not create a monitoring framework or a second control plane.
