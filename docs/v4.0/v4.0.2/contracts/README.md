# GoGauge shared contract

This directory is the **only authoritative copy** of the GoGauge shared contract.

## Normative documents

- [Signal contract](signal-contract.md)
- [MQTT contract](mqtt-contract.md)

## Normative examples

- [Provider status](examples/provider-status.json)
- [Signal catalogue](examples/signal-catalogue.json)
- [Provider state](examples/provider-state.json)

## Authority rule

A Go package may mirror these types and validation rules, but the package is an implementation. Providers written in other languages must be able to conform by reading these documents and examples alone.

GoDriveLog and other providers may document their dependency on this contract. They must not maintain a second handwritten specification.

## Version 1 scope

Version 1 defines:

- provider and runtime identity
- signal definitions
- typed signal state
- provider status and health
- MQTT topics and delivery semantics
- provider discovery
- complete snapshots
- consumer freshness inputs

Version 1 does not define commands, RPC, raw CAN transport, vehicle control, plugins, scripting or dashboard appearance.
