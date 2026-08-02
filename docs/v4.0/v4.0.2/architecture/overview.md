# GoGauge architecture overview

## Purpose

GoGauge converts generic provider telemetry into configured visual displays.

```text
MQTT broker
     |
     v
MQTT boundary and provider discovery
     |
     v
Provider registry + catalogue registry
     |
     v
Signal store
     |
     v
Bindings
     |
     v
Widgets and dashboards
```

## Generic design

GoGauge understands provider identity, signal definitions, typed values, validity, health, online state and freshness. It does not understand how a provider acquired the data.

The same architecture can consume:

- `engine.rpm`
- `battery.second.voltage`
- `weather.pressure`
- `inverter.output_power`
- `lathe.spindle_speed`

Domain-specific signal names are data, not hard-coded application architecture.

## Discovery versus configuration

- The provider catalogue says what data is available.
- GoGauge's catalogue registry and cache remember discovered definitions.
- User configuration says what should be displayed and how.
- Discovery never silently rewrites user configuration.

## State ownership

- Providers own values, sample timestamps, validity and explicit health.
- MQTT status determines whether the provider is online.
- GoGauge calculates staleness.
- Widgets receive typed, resolved state through bindings, never MQTT messages.
