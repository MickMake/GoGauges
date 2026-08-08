# GoGauges Core accepted decisions

Chat 4 decision identifiers use `GG-C###` and do not reuse identifiers from earlier chats.

## GG-C001 — Single Core state owner

**Decision:** One Core goroutine owns all mutable provider registry, catalogue registry, signal store and binding-resolution state.

**Reason:** Explicit single ownership removes cross-component locking ambiguity while avoiding an event-bus framework or a manager object for every noun.

## GG-C002 — Bounded MQTT handoff

**Decision:** Validated status and catalogue messages enter a bounded FIFO control queue. Complete state snapshots use one coalescing latest-state slot per `provider_id + instance_id`, with a capacity-one wake notification.

The control queue must never grow without bound. When it reaches capacity, GoGauges reports queue pressure prominently and uses the simplest safe bounded handling available during implementation, such as brief cancellable blocking or replacing superseded control messages only where semantic equivalence can be proven. MQTT callback execution must not block indefinitely, and status or catalogue messages must not be silently discarded without diagnostics.

Exact queue capacity, enqueue timing and the chosen simple overflow technique are implementation tuning values rather than architectural constants.

**Reason:** Status and catalogue traffic is low-rate and should be processed coherently. State is a complete replaceable snapshot, so queuing obsolete snapshots wastes memory and latency. A bounded control path must also define failure visibility without introducing a priority or event framework.

## GG-C003 — Offline-capable readiness

**Decision:** GoGauges is ready when configuration, Core state and the UI boundary are available. MQTT connectivity and live provider availability do not block readiness.

**Reason:** An offline dashboard must still start, display cached definitions and preserved bindings, and report provider unavailability.

## GG-C004 — Provider conflict handling

**Decision:** Conflicting live instances using one `provider_id` are never merged and no winner is silently selected. Ordinary state application pauses while the provider is conflicted.

A conflict clears:

- immediately when all but one instance explicitly report offline; or
- after only one instance continues to be observed as live for a configurable conflict-recovery timeout.

Five seconds is a provisional suggested default for that timeout, not a user-selected architectural constant. The value must be easy to tune during implementation without changing the conflict-resolution mechanism.

Sequence tracking resets when the surviving instance is accepted. State received during the conflict is not replayed or merged.

**Reason:** `provider_id` is a durable public identity, while `instance_id` identifies one process. Silent merging would make state origin unknowable. The recovery mechanism matters architecturally; the exact timeout is an operational tuning value.

## GG-C005 — Atomic complete-snapshot processing

**Decision:** A valid newer complete snapshot replaces the previous active provider snapshot atomically. Equal sequences are duplicates and ignored. Lower sequences are rejected as old or out of order. Sequences are never compared across different `instance_id` values.

When a new instance is accepted, previous live values are cleared to unobserved until that instance publishes a valid complete snapshot.

**Reason:** State messages are complete current-state documents, not patches or durable event history.

## GG-C006 — Consumer-owned staleness engine

**Decision:** GoGauges calculates staleness with one Core timer using the retained value's `observed_at`, catalogue `stale_after_ms` and explicit provider availability. There is no goroutine or timer per signal.

Freshness is represented separately as `unknown`, `fresh` or `stale`; validity, enabled state, provider reported online/offline state, MQTT transport connection state, conflict and health remain separate facts.

An explicit provider offline state may force retained values stale. Ordinary freshness otherwise continues to age from `observed_at`. Temporary GoGauges MQTT disconnection is exposed separately as transport state and does not itself rewrite signal freshness.

**Reason:** Collapsing transport connectivity, provider state and signal freshness into one status creates contradictory states and makes deterministic testing difficult.

## GG-C007 — Immutable UI boundary

**Decision:** Later dashboard and widget layers receive immutable Core snapshots and coalesced change notifications. They do not receive mutable Core maps, MQTT payloads or per-message event streams.

**Reason:** UI code needs the latest coherent state, not transport ownership or internal concurrency concerns.

## GG-C008 — Minimal persistence

**Decision:** Version 1 persists only:

- user-owned dashboard configuration and bindings
- the application-managed catalogue cache

Live values, provider online state, `instance_id`, state sequence and calculated staleness are not persisted as authority. No database is introduced.

The catalogue cache uses the platform user-cache directory, flushes dirty state every 30 seconds, flushes on `SIGHUP`, and flushes synchronously on shutdown. Unsupported cache versions are logged, ignored and rebuilt through discovery.

**Reason:** Two ordinary files satisfy offline startup and user ownership without creating a database lifecycle.

## GG-C009 — Durable binding identity

**Decision:** Durable bindings use `provider_id + signal_id`; `instance_id` is never stored in user configuration.

When a provider is offline or a signal disappears, bindings are preserved. A returning signal reconnects automatically only when its stable identity and value type remain compatible. An incompatible type change leaves the binding unresolved and reports the incompatibility.

**Reason:** Runtime process identity must not destroy durable user configuration, but silent type changes are unsafe.

## GG-C010 — Current-state-only signal store

**Decision:** GoGauges Core version 1 stores current signal state only. It does not maintain a general per-signal sample history for graphs, traces or widgets, and later UI code must not depend on Core retaining every MQTT sample.

If graph or history functionality later requires historical samples, it may introduce a separate explicitly bounded history component with its own ownership and retention policy.

**Reason:** Current dashboard state and sample-history retention are different responsibilities. Keeping history out of Core v1 avoids unbounded memory growth and unnecessary coupling to hypothetical graph consumers.
