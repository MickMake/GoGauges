# GoGauge v4.0 status

## Design status

**Provisionally accepted.**

The repository boundary and shared contract version 1 are defined sufficiently for cross-repository review. The contract is intentionally small and telemetry-only.

## Implementation status

Not asserted by this documentation set. The v4.0 repository began as a renamed copy of pre-split code, so provider-side code may still be present during migration.

## Locked direction

- GoGauge is generic and does not know CAN, OBD, ECUs or vehicle decoders.
- GoGauge owns the authoritative shared contract.
- Providers publish status, catalogue and complete state snapshots.
- User configuration is separate from discovered catalogue data.
- Widgets receive typed signal-store state and never subscribe directly to MQTT.
- Staleness is calculated by GoGauge; it is not a provider-published authoritative state.
- Version 1 is telemetry only.

## Next review checkpoint

Review this contract together with the GoDriveLog provider documentation before implementation. Future GoGauge Core and widget-design chats may refine consumer internals and presentation, but must not silently change this contract.
