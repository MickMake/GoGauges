# GoGauge MQTT contract version 1

## 1. Topic namespace

Each provider uses exactly three topics:

```text
gogauge/v1/providers/<provider-id>/status
gogauge/v1/providers/<provider-id>/catalog
gogauge/v1/providers/<provider-id>/state
```

No version 1 per-signal topics, command topics, control topics, raw CAN topics or RPC topics exist.

## 2. Delivery rules

| Topic | Retained | QoS | Purpose |
|---|---:|---:|---|
| `status` | yes | 1 | Provider identity, online state, enabled state and health. |
| `catalog` | yes | 1 | Complete signal catalogue for the provider instance. |
| `state` | no | 0 | Complete current provider signal snapshot. |

## 3. Provider discovery

GoGauge subscribes to:

```text
gogauge/v1/providers/+/status
gogauge/v1/providers/+/catalog
gogauge/v1/providers/+/state
```

Retained status and catalogue messages allow a new consumer to discover existing providers. Discovery adds data to GoGauge's provider and catalogue registries; it does not create or rearrange gauges automatically.

## 4. Status lifecycle

On successful connection, a provider publishes retained online status containing matching `provider_id` and `instance_id`.

The provider configures a retained MQTT Last Will with:

- the same `provider_id`
- the same `instance_id`
- `online: false`
- health `unknown`
- a useful disconnect message

On clean shutdown, the provider explicitly publishes its offline status before disconnecting.

## 5. Catalogue publication

The catalogue is a complete retained document. It carries the same `provider_id` and `instance_id` as status and state messages.

A new `revision` is published whenever the effective catalogue changes. Removed signals are absent from the new catalogue; consumers preserve user bindings and mark them unavailable.

## 6. State publication

Every state message is a complete provider snapshot, not a patch.

A provider may:

- publish promptly after changes
- coalesce changes up to its configured maximum publication rate
- publish a heartbeat snapshot at least once per second

The provider controls acquisition and MQTT publication rates. GoGauge controls render rate.

The state envelope includes:

- `contract_version`
- `provider_id`
- `instance_id`
- session `sequence`
- `published_at`
- complete `signals` map

`published_at` is snapshot transmission time. Each signal's `observed_at` is the timestamp of its retained value. These timestamps are not interchangeable.

## 7. Duplicate provider identity

If GoGauge observes overlapping live messages for one `provider_id` carrying different `instance_id` values, that provider identity is conflicted.

GoGauge must not silently merge conflicting state or choose one producer as authoritative. Exact timeout and recovery presentation belong to GoGauge Core, but no leader-election or coordination protocol is introduced in version 1.

## 8. Validity and staleness

Providers publish value, value timestamp, validity, enabled state and health. GoGauge calculates staleness using the signal definition and provider status.

When an observation fails and a last-good value is retained, its original `observed_at` remains unchanged. This prevents an old value from becoming falsely fresh.

## 9. Prohibited version 1 behaviours

Version 1 does not define:

- MQTT commands
- vehicle control
- raw CAN transport
- request/response RPC
- consumer-requested acquisition rates
- plugin negotiation
- dashboard or widget metadata
