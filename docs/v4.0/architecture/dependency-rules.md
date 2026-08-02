# GoGauge dependency rules

## Direction

```text
MQTT payloads
     |
     v
contract validation and parsing
     |
     v
provider/catalogue registries
     |
     v
signal store
     |
     v
bindings
     |
     v
widgets
```

## Rules

1. MQTT payloads are parsed and validated once at the boundary.
2. Widgets never subscribe to MQTT or inspect contract JSON.
3. Provider conflict handling occurs before state reaches ordinary bindings.
4. The catalogue registry is provider-owned metadata; user configuration is separate.
5. The catalogue cache is disposable and rebuildable; it is not online-state authority.
6. Bindings use `provider_id + signal_id`, never `instance_id`.
7. Staleness is calculated in GoGauge, not copied from a provider flag.
8. Dashboard and widget code may use engineering limits as hints but owns final visual ranges.
9. GoGauge contains no CAN, OBD, ECU or vehicle-decoder dependencies.
10. The contract package, if implemented in Go, remains small and contains no MQTT client, widget, configuration-loader or application lifecycle code.
