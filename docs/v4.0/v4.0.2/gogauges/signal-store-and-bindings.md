# Signal store and bindings

## Signal store

The signal store receives validated complete provider snapshots and exposes typed state to bindings.

For each signal it preserves:

- current retained value
- `observed_at` associated with that value
- validity
- enabled state
- explicit health and message
- provider online/offline state
- provider conflict state
- GoGauge-calculated staleness

## Freshness

When `stale_after_ms` exists, GoGauge compares current time with `observed_at` and also considers provider status. The exact clock implementation belongs to Core, but there is only one consumer-calculated stale result.

A provider `degraded` value remains distinct from stale. Degraded means the provider says data is usable but impaired.

## Invalid last-good state

If a state entry is invalid but contains a last-good value:

- retain the supplied value
- retain its supplied original `observed_at`
- expose `valid: false`
- expose error health and message
- calculate staleness from that original timestamp

## Bindings

A binding identifies a signal using:

```text
provider_id + signal_id
```

It does not bind to `instance_id`.

When a provider or signal disappears:

- preserve the binding
- report missing or unavailable state
- reconnect automatically if the same stable identity returns

## Widget boundary

Bindings translate signal-store state into widget inputs. Widgets never parse MQTT messages or decide provider conflicts.
