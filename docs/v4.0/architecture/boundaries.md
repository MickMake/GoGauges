# GoGauge repository boundaries

## Owned here

- authoritative shared signal contract
- authoritative MQTT topic and delivery contract
- MQTT consumer boundary and reconnect behaviour
- provider discovery and conflict detection
- catalogue registry and cache
- signal store and freshness calculation
- bindings
- dashboard and gauge configuration
- widgets, rendering and realism
- gauge verification harness

## Not owned here

- CAN arbitration IDs or frames
- OBD PIDs or polling
- ECU discovery
- diagnostic sessions
- vehicle-specific decoding
- GoDriveLog recording internals
- provider source lifecycle

## Forbidden coupling

GoGauge must not:

- import GoDriveLog provider internals
- ask widgets to parse MQTT
- infer missing provider acquisition details
- publish vehicle-control or command messages in version 1
- silently regenerate user configuration because catalogues changed
- treat cached catalogue data as proof that a provider is online

A provider written in any language can conform by following the Markdown and JSON contract; importing a Go package is optional.
