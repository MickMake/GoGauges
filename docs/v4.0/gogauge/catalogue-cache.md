# Catalogue cache

## Purpose

The catalogue cache remembers provider and signal definitions across GoGauge restarts. It allows configured bindings to resolve known definitions while providers are offline.

It is not user configuration and is not proof that a provider is currently online.

## Runtime model

1. Load the JSON cache at startup.
2. Mark every cached provider offline until live status proves otherwise.
3. Maintain the active catalogue registry in memory.
4. Mark the cache dirty when provider definitions change.
5. Persist only when dirty, except for an explicit forced flush.

## Flush triggers

- periodic dirty flush
- immediate forced flush on `SIGHUP`
- final synchronous flush on `SIGTERM` or `SIGINT`

`SIGHUP` means cache flush only in this design. Configuration reload requires a separate accepted decision.

## Atomic write

```text
write catalogue-cache.json.tmp
flush and close
rename over catalogue-cache.json
```

Writes are serialised. Dirty state is cleared only after a successful replacement. On failure, GoGauge logs the error and keeps the cache dirty.

## Cached content

Cache:

- `cache_version`
- provider identity and descriptive metadata
- last known catalogue
- catalogue revision and optional definition hash
- last discovery time

Do not cache as authority:

- current online state
- rapidly changing live values
- state sequence numbers
- consumer freshness result

## Example top level

```json
{
  "cache_version": 1,
  "providers": {}
}
```

The exact file location, periodic interval and future cache migrations remain GoGauge Core decisions.
