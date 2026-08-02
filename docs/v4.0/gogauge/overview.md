# GoGauge core overview

## Responsibilities

GoGauge Core owns the data path between validated MQTT messages and display bindings:

```text
MQTT boundary
    -> provider registry
    -> catalogue registry and cache
    -> signal store
    -> bindings
    -> widgets and dashboards
```

## Provider independence

Core operates on contract concepts only. It does not know whether a value came from CAN, OBD, a weather station, a power monitor or a simulator.

## Data-state separation

Core must preserve separate representations for:

- provider online/offline
- provider conflict
- signal availability
- signal enabled state
- signal validity
- signal health
- consumer-calculated staleness
- current retained value and its timestamp

Detailed component interfaces and concurrency belong to the GoGauge Core design. They must preserve the contract and dependency rules here.
