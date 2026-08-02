# GoGauge v4.0 architecture documentation

GoGauge is a generic MQTT-driven dashboard and realistic gauge application. It discovers conforming providers, stores typed signal state, binds signals to configured gauges, and renders dashboards without knowing how providers acquire their data.

A provider may represent a vehicle, weather station, solar system, workshop machine, simulator or any other conforming data source.

> **Core boundary:** GoDriveLog understands vehicle data. GoGauge understands displays. MQTT carries generic named signals between them.

GoGauge owns the only authoritative shared signal and MQTT contract under [`contracts/`](contracts/README.md).

This documentation is a living design. Git records its history; do not create nested documentation versions beneath `docs/v4.0/`.

## Start here

- [Document index](INDEX.md)
- [Current status](STATUS.md)
- [Accepted decisions](DECISIONS.md)
- [Architecture overview](architecture/overview.md)
- [Authoritative contract](contracts/README.md)
- [Repository split migration](migration/repository-split.md)
