# GoGauge vocabulary

| Term | Meaning |
|---|---|
| Provider | A running program that publishes conforming telemetry. |
| `provider_id` | Stable configured identity used in topics and user bindings. |
| `instance_id` | Runtime identity for one provider process; regenerated at startup. |
| Signal | A stable named typed value exposed by a provider. |
| `signal_id` | Stable identifier unique within one provider. |
| Signal identity | The pair `provider_id + signal_id`. |
| Signal definition | Provider-owned catalogue metadata for one signal. |
| Catalogue | Provider-published set of signal definitions. |
| Catalogue registry | GoGauge's in-memory view of provider catalogues. |
| Catalogue cache | Application-managed JSON persistence of discovered definitions; not proof of online state. |
| Provider registry | GoGauge's current view of providers, runtime instances, status and conflicts. |
| Value | Latest valid value retained for a signal, or null if none exists. |
| `observed_at` | Timestamp associated with the retained value. |
| Validity | Whether the provider currently considers the signal assessment valid. |
| Enabled | Whether the provider intends the provider or signal to operate. |
| Health | Explicit `ok`, `degraded`, `error` or `unknown` assessment. |
| Online | Provider connectivity state derived from status and Last Will. |
| Stale | Consumer-calculated freshness result using sample time, threshold and provider status. |
| Complete snapshot | One state message containing every signal currently represented by that provider instance. |
| Signal store | Typed current signal state used by bindings. |
| Binding | User-configured connection from a provider signal to a gauge input. |
| Widget | Reusable visual implementation. |
| Gauge | Configured widget instance. |
| Dashboard | User-owned arrangement of gauges. |
