# GoGauge v4.0 open questions

These questions belong to later GoGauge Core, dashboard and widget design. They do not change the accepted shared contract unless explicitly brought back to this documentation owner.

## Provider registry

- What timeout and recovery rules resolve overlapping `instance_id` conflicts?
- How is a conflict reported to users and bindings?

## Catalogue cache

- What is the default periodic flush interval?
- Where is the platform-specific cache file located?
- What migration policy applies when `cache_version` changes?

## Signal store

- What display state is used when `stale_after_ms` is omitted?
- How are monotonic and wall-clock time combined safely for freshness calculation?
- How much recent history, if any, belongs in memory for graph widgets?

## Configuration and discovery

- How are newly discovered or removed signals presented to the user?
- What is the exact dashboard and binding configuration schema?
- How are starter configurations generated without silently rewriting user choices?

## Widgets and dashboards

- How are realistic gauge families, quirks, layout and themes represented?
- How are dashboard templates reused across providers?
- What visual behaviours represent invalid, stale, disabled, offline and conflicted state?

## Security and deployment

- Which broker authentication and TLS defaults apply?
- Which environments require broker access control beyond a local Raspberry Pi installation?
