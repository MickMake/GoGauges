# GoGauge signal contract version 1

## 1. Identity

### Provider identity

- `provider_id` is stable, configured, human-readable and unique on the broker.
- `instance_id` identifies one running provider process and is regenerated at startup.
- User configuration never binds to `instance_id`.

### Signal identity

A signal is identified by:

```text
provider_id + signal_id
```

`signal_id` is stable and unique within the provider. Readable dot-separated identifiers are recommended, for example:

```text
engine.rpm
battery.second.voltage
solar.panel.power
```

## 2. Supported value types

Version 1 supports:

- `number`
- `boolean`
- `string`
- `enum`

Arrays, nested user schemas and arbitrary objects are outside version 1.

## 3. Signal definition

| Field | Required | Meaning |
|---|---:|---|
| `id` | yes | Stable `signal_id`. |
| `type` | yes | One of the supported value types. |
| `label` | no | Human-readable label. |
| `description` | no | Longer explanation. |
| `unit` | no | Canonical provider unit. |
| `min` | no | Recommended lower engineering limit for number signals. |
| `max` | no | Recommended upper engineering limit for number signals. |
| `values` | for enum | Allowed enum strings. |
| `expected_update_ms` | no | Provider's expected observation interval. Metadata only; not consumer control. |
| `stale_after_ms` | no | Threshold GoGauge may use to calculate staleness. |

`min` and `max` are engineering hints, not mandatory display ranges.

If `stale_after_ms` is absent, the contract does not provide a time-based stale threshold. GoGauge may still represent invalid or offline state but must not invent an authoritative provider threshold.

## 4. Provider status dimensions

Provider status separates:

- `online`: broker-visible provider connectivity
- `enabled`: provider intent to operate
- `health`: `ok`, `degraded`, `error` or `unknown`
- `message`: optional human-readable diagnostic detail

`offline` and `disabled` are not health enum values.

## 5. Signal state dimensions

Each signal snapshot entry separates:

| Field | Meaning |
|---|---|
| `value` | Latest valid value retained by the provider, or null if none exists. |
| `observed_at` | UTC RFC 3339 timestamp associated with that retained value, or null. |
| `valid` | Whether the provider currently considers the signal assessment valid. |
| `enabled` | Whether the provider intends this signal to operate. |
| `health` | `ok`, `degraded`, `error` or `unknown`. |
| `message` | Optional diagnostic detail. |

### State mapping

#### Unobserved

```text
value: null
observed_at: null
valid: false
enabled: true
health: unknown
```

#### Valid

```text
value: current valid value
observed_at: timestamp of that value
valid: true
health: ok, unless the provider explicitly reports usable-but-impaired degraded state
```

#### Invalid observation with a previous valid value

```text
value: last valid value
observed_at: timestamp of that last valid value
valid: false
health: error
message: explanation of the failed latest assessment
```

A failed observation must not refresh `observed_at` while retaining an older value.

#### Disabled

```text
value: null
observed_at: null
valid: false
enabled: false
health: unknown
```

## 6. Staleness

Staleness is calculated by GoGauge from:

- `observed_at`
- signal `stale_after_ms`
- provider online/offline status

The provider does not publish an authoritative `stale` field or stale health value. `degraded` must be an explicit provider assessment that data is still usable but impaired; it is not inferred automatically from elapsed time.

## 7. Catalogue revision

A catalogue contains:

- `contract_version`
- `provider_id`
- `instance_id`
- integer `revision`
- optional `definition_hash`
- signal definitions

`revision` increments whenever the effective published catalogue changes for that provider installation. `definition_hash` is optional diagnostic metadata and does not replace the integer revision.

## 8. Removed signals

When a signal disappears from a later catalogue revision:

- it becomes unavailable in GoGauge
- existing user bindings are preserved
- the signal may be rediscovered if the same stable ID returns
- discovery must not silently delete or rewrite user configuration
