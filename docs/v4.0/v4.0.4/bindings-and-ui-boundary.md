# Bindings and UI boundary

## Configuration ownership

GoGauges maintains two separate stores.

### Catalogue registry and cache

Application-managed metadata describing what providers say is available.

- updated through discovery
- rebuildable from providers
- not hand-edited
- not proof of provider online state

### User configuration

User-owned description of what should be displayed and how.

- explicitly editable
- persistent
- never silently rewritten by discovery
- may refer to absent providers or signals

Starter configuration generation, when implemented, produces a user-owned result. It does not remain an automatically regenerated view of discovery.

## Durable binding identity

A durable signal binding uses:

```go
type SignalKey struct {
    ProviderID string
    SignalID   string
}
```

`instance_id` never appears in durable binding configuration because it identifies one provider process, not the provider installation.

## Binding validation

At configuration load:

- syntax and required fields must validate
- duplicate binding IDs must be rejected
- known catalogue definitions are used for type compatibility checks
- an unknown provider or signal does not invalidate the whole dashboard; the binding remains unresolved

At runtime:

- provider online: binding may resolve to current state
- provider offline: binding remains durable and exposes offline/stale state
- signal removed from catalogue: binding remains but becomes unavailable
- compatible signal returns: binding resolves automatically
- incompatible signal type returns: binding remains unresolved and reports incompatibility

Catalogue `min`, `max`, unit and enum values may assist validation or initial configuration, but they do not silently rewrite user presentation choices.

## Separation from presentation

A signal binding selects a semantic signal. It does not own:

- gauge family
- visual scale
- dashboard position
- animation
- needle physics
- smoothing
- colour or theme
- render cadence

Those belong to later dashboard and widget design.

## Resolved binding state

Core exposes a resolved binding view containing at least:

- stable binding ID
- `provider_id`
- `signal_id`
- resolution state
- compatible catalogue definition, when available
- current typed signal state, when available
- validity
- enabled state
- external health and message
- freshness
- provider offline state
- provider conflict state
- diagnostic reason when unresolved

Suggested resolution states are ordinary Core values rather than a plugin interface:

```text
resolved
provider_missing
provider_offline
provider_conflicted
signal_missing
signal_incompatible
unobserved
```

These states do not replace the separate signal dimensions. For example, a resolved signal can still be invalid or stale.

## Core-to-UI boundary

Later UI code receives immutable snapshots containing:

- provider snapshots
- catalogue information
- signal state
- binding resolution
- validity
- enabled state
- health and messages
- freshness
- diagnostics summary

UI code does not receive:

- MQTT client objects
- MQTT topics or payload bytes
- mutable provider or signal maps
- catalogue-cache file handles
- GoDriveLog internals
- CAN IDs, OBD PIDs or ECU addresses

## Notification model

Core publishes a monotonically increasing snapshot revision and sends a coalesced notification that a newer snapshot exists.

The UI fetches the latest complete immutable snapshot. It is not required to render every MQTT state publication or every intermediate Core mutation.

Slow UI consumers cannot block MQTT parsing or Core mutation indefinitely. At most they miss intermediate revisions and then read the newest coherent state.

## Provider and signal disappearance

When a provider goes offline:

- preserve its bindings
- preserve cached definitions
- retain any previous value only as stale/offline state
- do not treat cached metadata as live proof

When a new provider instance is accepted:

- clear previous live values to unobserved
- preserve bindings
- wait for the new instance's complete state snapshot

When a signal disappears from a catalogue:

- mark it unavailable
- preserve bindings and presentation configuration
- do not delete or rearrange gauges

When it returns compatibly, resolution resumes without rewriting user configuration.
