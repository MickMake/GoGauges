# Configuration ownership

## Two different stores

### Catalogue registry and cache

Application-managed description of what providers say is available.

- updated by discovery
- rebuildable from providers
- not hand-edited
- not proof of provider online state

### User configuration

User-owned description of what should be displayed and how.

- explicitly editable
- persistent
- never silently rewritten by discovery
- may refer to providers or signals that are currently absent

## Binding identity

User configuration binds with `provider_id + signal_id`.

It must not contain runtime `instance_id` values.

## Discovery changes

- New provider or signal: add to available catalogue data; do not add a gauge automatically in this contract design.
- Removed signal: mark unavailable; preserve the binding.
- Returning signal with the same stable identity: allow the existing binding to resume.
- Catalogue metadata change: update the application-managed catalogue without rewriting visual choices.

Starter configuration generation may be designed later. Once generated, the result is user-owned.
