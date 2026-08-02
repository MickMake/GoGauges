# GoGauge repository-split migration

## Target

GoGauge becomes the generic display repository and the sole owner of the shared signal and MQTT contract.

## Migration sequence

1. **Standardise the product name**
   - Use `GoGauge` in documentation, package references, executable naming and configuration vocabulary.
   - The MQTT namespace remains lowercase `gogauge`.

2. **Own the authoritative contract**
   - Maintain the normative Markdown and JSON examples under `docs/v4.0/contracts/`.
   - Keep any Go contract package as an implementation of these documents, not a second authority.

3. **Remove provider-side ownership**
   - Remove CAN, OBD, ECU discovery, vehicle decoders and recording internals from GoGauge.

4. **Establish the MQTT boundary**
   - Discover status and catalogues through wildcard subscriptions.
   - Validate status, catalogue and complete state snapshots once.

5. **Establish registries and signal store**
   - Track `provider_id` separately from live `instance_id`.
   - Cache catalogue definitions independently of online state.
   - Calculate staleness in the consumer.

6. **Establish bindings and configuration ownership**
   - Bind gauges using `provider_id + signal_id`.
   - Preserve user configuration when providers or signals disappear.
   - Do not silently add, remove or reposition gauges during discovery.

7. **Retain display ownership**
   - Keep widgets, dashboards, rendering, realism and the gauge verification harness in GoGauge.

## Completion criteria

- GoGauge contains no vehicle acquisition or decoding ownership.
- The contract has one authoritative documentation location.
- Widgets do not subscribe directly to MQTT.
- User configuration and catalogue cache are separate.
- Provider conflicts can be detected with `instance_id`.
- Version 1 remains telemetry-only.
