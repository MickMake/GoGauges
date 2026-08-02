# Widget data boundary

## Rule

Every widget receives typed GoGauge state through a binding. No widget subscribes directly to MQTT.

## Widgets may receive

- typed value
- unit and engineering metadata
- validity
- enabled state
- health and message
- stale state
- provider offline or conflicted state

## Widgets do not receive

- MQTT topics or payload bytes
- CAN IDs or OBD PIDs
- provider source configuration
- catalogue-cache persistence details
- reconnect logic

## Presentation ownership

Signal metadata may help initialise a gauge, but it does not dictate:

- gauge family
- visual range
- dashboard position
- realism behaviour
- colour, bezel, needle or theme

Those are user and GoGauge display concerns. A provider must never publish a preferred widget through the shared contract.
