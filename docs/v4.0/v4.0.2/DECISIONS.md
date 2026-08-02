# GoGauge v4.0 accepted decisions

## GG-001 — Authoritative contract ownership

**Decision:** The only authoritative shared signal and MQTT contract lives under `docs/v4.0/contracts/` in the GoGauge repository.

**Reason:** Providers and consumers need one normative source rather than competing handwritten copies.

## GG-002 — Generic provider model

**Decision:** GoGauge works with generic providers and signals. It contains no CAN, OBD, ECU or vehicle-decoder knowledge.

**Reason:** The same display system should work with vehicle, power, weather, industrial and simulated data.

## GG-003 — Public and runtime identity

**Decision:**

- `provider_id` is stable, configured and used in MQTT topics and user bindings.
- `instance_id` identifies one running provider process and is regenerated at startup.
- A signal's stable public identity is `provider_id + signal_id`.
- `instance_id` never appears in user bindings.

**Reason:** Human-readable configuration and duplicate-live-provider detection require different identifiers.

## GG-004 — MQTT topic set

**Decision:** Version 1 uses exactly three provider topics:

```text
gogauge/v1/providers/<provider-id>/status
gogauge/v1/providers/<provider-id>/catalog
gogauge/v1/providers/<provider-id>/state
```

**Reason:** Three topics support discovery, metadata and live telemetry without a per-signal topic forest or RPC layer.

## GG-005 — Delivery semantics

**Decision:**

- `status`: retained, QoS 1
- `catalog`: retained, QoS 1
- `state`: non-retained, QoS 0, complete snapshot

**Reason:** Discovery metadata should survive subscriber arrival; fast telemetry can tolerate loss because the next complete snapshot repairs it.

## GG-006 — Supported value types

**Decision:** Version 1 supports `number`, `boolean`, `string` and `enum`.

**Reason:** These cover the accepted use cases without adding general schemas or a framework.

## GG-007 — Separate state dimensions

**Decision:** Signal value, sample timestamp, validity, enabled state, health, provider online state and consumer-calculated staleness are separate facts.

**Reason:** Collapsing them into one status enum creates contradictions such as an online provider carrying an invalid or stale signal.

## GG-008 — Shared health values

**Decision:** Provider and signal health use the same enum:

```text
ok
degraded
error
unknown
```

`online` and `enabled` remain separate booleans. `valid` remains a separate signal field.

**Reason:** Reusing one small health vocabulary simplifies handling without pretending the dimensions are identical.

## GG-009 — Consumer-calculated staleness

**Decision:** GoGauge calculates staleness using `observed_at`, catalogue `stale_after_ms`, and provider online status. Providers do not publish an authoritative stale flag or stale health value.

**Reason:** There must be one answer to freshness, calculated where current time and provider status are known.

## GG-010 — Catalogue and user configuration separation

**Decision:** Provider-owned catalogue data and user-owned display configuration are separate stores. Discovery may update the catalogue but must not silently rewrite user configuration.

**Reason:** The catalogue describes what is available; configuration describes what the user wants.

## GG-011 — Catalogue cache lifecycle

**Decision:** GoGauge keeps the catalogue in memory and persists an application-managed JSON cache:

- periodic flush when dirty
- forced flush on `SIGHUP`
- final synchronous flush on `SIGTERM` or `SIGINT`
- atomic temp-file replacement

`SIGHUP` means cache flush only unless a later decision explicitly adds another meaning.

**Reason:** Offline startup needs remembered definitions without making the cache a second authority.

## GG-012 — Removed signals preserve bindings

**Decision:** A signal removed from a new catalogue becomes unavailable. GoGauge preserves user bindings and permits rediscovery if the same stable signal ID returns.

**Reason:** Provider changes must not silently destroy user-owned dashboards.

## GG-013 — Widget data boundary

**Decision:** Widgets read typed state through GoGauge bindings and the signal store. Individual widgets never subscribe directly to MQTT.

**Reason:** MQTT parsing, reconnects, conflict handling and staleness belong at one boundary, not in every widget.

## GG-014 — Telemetry-only version 1

**Decision:** Version 1 defines no command topics, vehicle-control topics, raw CAN topics, RPC over MQTT, plugin system or general-purpose framework.

**Reason:** These are separate problems and would turn a small contract into a cathedral.

## GG-015 — Invalid observations preserve sample time

**Decision:** If a provider retains a last valid value after a failed observation, the provider must also retain the timestamp associated with that valid value. It publishes `valid: false`, error health and a message. If no valid value exists, value and `observed_at` are null.

**Reason:** Updating `observed_at` while retaining an older value would defeat GoGauge's staleness calculation.

**Change record:** This corrects an earlier ambiguous proposal that paired a last-good value with the time of a failed observation.

**Affected documents:**

- `DECISIONS.md`
- `CHANGELOG.md`
- `contracts/signal-contract.md`
- `contracts/mqtt-contract.md`
- `contracts/examples/provider-state.json`
- `gogauge/signal-store-and-bindings.md`
