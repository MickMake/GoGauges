# GoGauge v4.0 documentation changelog

## 2026-08-02

### Added

- Established the repository-local `docs/v4.0/` living design.
- Established GoGauge as the sole owner of the authoritative shared contract.
- Added authoritative signal and MQTT contract documents with validated JSON examples.
- Added provider discovery, catalogue cache, signal-store, binding, configuration and widget-boundary documentation.
- Added repository-split migration documentation.

### Clarified

- Standardised the name `GoGauge` throughout the documentation.
- Separated stable `provider_id` from runtime `instance_id`.
- Separated value, sample timestamp, validity, enabled state, health, provider online state and consumer-calculated staleness.
- Confirmed complete provider snapshots on one non-retained state topic.
- Confirmed catalogue cache and user configuration are different stores.
- Confirmed catalogue removal preserves user bindings.

### Corrected accepted design

An earlier proposal retained a last good value while replacing its sample timestamp with the time of a failed observation. That would falsely refresh stale data. The corrected contract keeps the last valid value and its original `observed_at`, while publishing `valid: false`, `health: error` and an explanation.

Affected documents:

- `DECISIONS.md`
- `contracts/signal-contract.md`
- `contracts/mqtt-contract.md`
- `contracts/examples/provider-state.json`
- `gogauge/signal-store-and-bindings.md`

### Superseded wording

- Replaced the legacy plural product name with `GoGauge`.
- Replaced the earlier per-signal state-topic proposal with one complete provider `state` topic.
- Removed any implication that providers publish an authoritative stale state.
