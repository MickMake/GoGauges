# GoGauges

> [!WARNING]
> **v4.0 is a split from another repository and is currently being worked on. Things may be broken until the split is complete.**

GoGauges is a generic MQTT-driven instrument and dashboard application focused on producing **highly realistic gauges**.

Its purpose is not merely to display a value inside a dial, bar, graph, or indicator. GoGauges aims to reproduce the visual character, movement, limitations, and quirks of real instruments.

GoGauges does **not** know about GoDriveLog at runtime. GoDriveLog is only one possible data provider.

A provider could instead be:

- a barometer
- a weather station
- a wind turbine
- a solar inverter
- a generator
- an industrial controller
- a simulator
- any other system that publishes conforming MQTT data

Anything that does not conform to the GoGauges contract will be ignored or rejected rather than guessed at.

## Core goals

GoGauges provides:

- highly realistic gauges and instruments
- realistic motion and response rather than mechanically perfect animation
- support for physical and historical gauge characteristics
- reusable gauges that are independent of the data source
- clear separation between incoming data, gauge behaviour, and rendering
- user-configurable dashboards
- provider and signal auto-discovery
- a standalone gauge verification harness
- a simple architecture with clearly owned components

A gauge should be capable of reproducing characteristics such as:

- inertia
- damping
- overshoot
- bounce
- lag
- hysteresis
- zero drift and offset
- vibration
- backlash
- intermittent movement
- nonlinear scales
- limited resolution
- mechanical stops
- warm-up behaviour
- tapping behaviour
- lighting, reflections, shadows, glass, wear, and material appearance

Not every gauge will use every characteristic. The goal is to make these behaviours available without turning the application into a cathedral built to store one screwdriver.

## v4.0 direction

Version 4.0 separates display responsibilities from vehicle-data collection.

GoGauges will focus on:

- owning and documenting the MQTT data contract
- discovering conforming providers
- discovering available signals
- maintaining current signal state
- handling stale, invalid, and offline data
- binding signals to gauges
- dashboard configuration
- starter configuration generation
- reusable gauge packages
- realistic gauge behaviour
- gauge assets, visual layers, and themes
- gauge verification and simulation tools

The normal runtime flow will be:

```text
MQTT Broker
     |
     v
Provider Discovery
     |
     v
 Signal Store
     |
     v
   Bindings
     |
     v
Gauge Behaviour
     |
     v
Rendering / Dashboards
```

The behaviour layer is important. Incoming data should not necessarily move a pointer or indicator directly. A real instrument may respond slowly, overshoot, vibrate, stick, drift, or settle according to its configured physical characteristics.

## Contract ownership

GoGauges owns the public contract used by data providers.

The contract will define:

- protocol version
- MQTT topic structure
- provider identity and status
- signal catalogues
- signal definitions
- signal samples
- value types
- units, ranges, validity, and staleness
- validation rules

A Go provider may import the Go contract package.

A provider written in another language should be able to conform by following the published MQTT and JSON specification without importing any Go code.

## Provider independence

GoGauges should use generic concepts such as:

```text
Provider
Signal
Sample
Unit
Status
Catalogue
Binding
Gauge
Dashboard
```

It must not require knowledge of:

- CAN
- OBD
- ECUs
- vehicle models
- diagnostic protocols
- GoDriveLog internals

Provider-defined signal names may still be domain-specific:

```text
engine.rpm
weather.pressure
inverter.output_power
turbine.rotor_speed
```

Those names are data, not hard-coded application architecture.

## Gauge packages

A gauge package should contain the information required to reproduce a particular instrument or instrument type.

Depending on the gauge, this may include:

- visual assets
- scale and range
- units and labels
- pointer or indicator geometry
- pivots and mechanical limits
- lighting and layer order
- movement behaviour
- realism quirks
- default configuration
- verification examples

Gauge packages should remain reusable. A realistic radial gauge should not care whether its input represents engine RPM, air pressure, voltage, wind speed, or something more peculiar involving steam and poor judgement.

## Auto-discovery

GoGauges should discover conforming providers and their published signal catalogues through MQTT.

It may generate a starter dashboard configuration from available signals. Once generated, that configuration belongs to the user and should not be silently rewritten whenever a provider changes.

Auto-discovery should assist configuration. It should not replace deliberate dashboard design.

## Gauge verification

The gauge verification harness belongs in this repository.

It should test both appearance and behaviour using synthetic inputs such as:

- ramps
- steps
- jitter
- vibration
- rapid reversals
- invalid values
- stale values
- provider disconnects
- minimum and maximum limits
- overshoot and settling
- hysteresis
- drift
- mechanical sticking
- tapping or disturbance events

Basic gauge testing should not require a vehicle, GoDriveLog, or a live MQTT provider.

The harness should make it possible to verify a gauge independently before placing it into a complete dashboard.

## Architectural boundaries

GoGauges owns:

- its MQTT contract
- provider discovery
- signal storage
- bindings
- gauge behaviour
- gauge rendering
- dashboards
- visual assets
- gauge verification

GoGauges does not own:

- provider-specific data acquisition
- CAN or OBD acquisition
- vehicle decoding
- raw CAN recording
- ECU discovery
- provider business logic
- direct vehicle control

Widgets and gauges should receive data through the GoGauges signal store. Individual gauges should not connect directly to MQTT.

## Design approach

GoGauges should remain modular, explicit, and small enough to understand.

Every major component should have one clear purpose.

The project should avoid:

- unnecessary abstraction
- speculative plugin systems
- hidden data paths
- direct coupling between gauges and providers
- a universal framework attempting to model every instrument ever made before breakfast

Realism should come from well-defined gauge packages and behaviour components, not from architectural sprawl.

## Current state

The v4.0 repository began as a renamed copy of the pre-split project. During migration, it may still contain vehicle acquisition, decoding, recording, and other provider-side code.

Those parts will be removed as the repository split progresses.

Expect temporary breakage, duplicated code, and a few cupboards that still contain tools belonging in the other shed.
