# GoGauges Core architecture — Chat 4 (`v4.0.4`)

Chat 4 defines the internal Core architecture of **GoGauges** on branch `v4.0_split`.

GoGauges is a generic MQTT-driven dashboard and gauge application. It consumes the authoritative shared contract already defined under [`docs/v4.0/v4.0.2/contracts/`](../v4.0.2/contracts/), discovers providers, tracks provider and catalogue state, stores current signal state, calculates staleness, resolves durable bindings and exposes immutable state to later dashboard and widget layers.

> **Core boundary:** GoDriveLog understands vehicle data. GoGauges understands displays. MQTT carries generic named signals between them.

## Authority and precedence

This documentation follows this priority order:

1. the Chat 4 prompt and accepted discussion outcomes
2. the authoritative shared contract and repository boundaries in `v4.0.2`
3. relevant accepted later-chat decisions that do not conflict with the above
4. later-chat drafts as implementation context only

The GoDriveLog Chat 3 mandatory-recording design is not carried forward. GoGauges does not depend on provider recording being enabled or available.

## Scope

GoGauges Core owns:

- application lifecycle
- MQTT consumer ownership and reconnect behaviour
- provider discovery and provider conflict handling
- provider registry state
- catalogue registry and catalogue cache
- signal-state storage
- consumer-calculated staleness
- durable signal bindings
- user configuration ownership
- immutable handoff to later UI and widget layers
- lightweight diagnostics and consumer-side testing

GoGauges Core does not own:

- CAN, OBD, ECU or vehicle decoding
- provider source lifecycle
- raw vehicle recording or replay
- vehicle-specific signal generation
- vehicle control
- gauge rendering, animation, physics or styling

## Documents

- [Accepted decisions](DECISIONS.md)
- [Open questions and proposed contract clarification](OPEN-QUESTIONS.md)
- [Core architecture](core-architecture.md)
- [MQTT and provider state](mqtt-and-provider-state.md)
- [Bindings and UI boundary](bindings-and-ui-boundary.md)
- [Testing](testing.md)

## Inherited-documentation corrections

This set records, without modifying earlier files, that:

- `GoGauges` is the required product name throughout this document set.
- references to the current contract use `docs/v4.0/v4.0.2/contracts/`.
- GoGauges accepts all four external health values: `ok`, `degraded`, `error`, `unknown`.
- GoGauges has no dependency on GoDriveLog recording.
- all new Chat 4 files live only beneath `docs/v4.0/v4.0.4/`.

## Status

The internal Core architecture is accepted for documentation. One shared-contract clarification is accepted in principle but remains explicitly non-normative until the final contract consolidation pass; see [OPEN-QUESTIONS.md](OPEN-QUESTIONS.md).
