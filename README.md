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
GoGauges is a deliberately small Go/Ebiten in-vehicle telemetry dashboard for Raspberry Pi-style installs.

The project is being reshaped around the active v3 runtime:

```text
vehicle endpoint
-> sensor polling runtime
-> sensor events
-> logs and dashboards as subscribers
-> dashboard scene model
-> renderer adapter
```

Ebiten is the default and only active renderer implementation in the v3.3 branch. The renderer remains a boundary: runtime, sensors, logging, and dashboard scene generation should not become Ebiten-owned.

The goal is simple: read vehicle telemetry, keep the runtime boring, log useful data, and render a dashboard that can look convincingly like real retro hardware instead of a web page wearing driving gloves.

## Current status

GoGauges now uses Ebiten for the active v3 dashboard command path. Earlier Fyne dashboard code lives in the v3.2.x line only; v3.3.x and later are Ebiten-first.

The current v3.3 implementation state is documented under `docs/v3.3/`. Some older config/runtime documents may still describe legacy concepts while the repo is being migrated.

## v3 direction

The intended v3 config shape is:

```yaml
vehicles: {}
sensors: {}
assets: {}
logs: {}
dashboards: {}
```

The important design rules are:

- Sensors own polling cadence using `poll`.
- Logs and dashboards subscribe to sensor events.
- Logs do not poll sensors independently.
- Dashboards do not fetch OBD values directly.
- If a documented config item exists, it is active.
- Each dashboard owns its physical/logical display target.
- GoGauges connects to an OBD-like endpoint address.
- Bench testing should use an OBD-like endpoint, for example `tcp://127.0.0.1:35000`.
- Unknown config fields should fail validation during v3 implementation.

Boring boundaries are intentional. Cleverness is allowed only when it pays rent and does not bring a YAML demon as a lodger.

## Runtime model

The intended v3 runtime is:

```text
load config
resolve vehicle
connect to the vehicle OBD-like endpoint
start sensor runtime
poll sensors according to sensors.<id>.poll
emit sensor reading/status events
logs receive selected sensor events
dashboards receive sensor events implied by widgets
render dashboard updates from event state
```

Sensor readings should carry their original read timestamp. Log writers may add their own write timestamp, but the sensor timestamp is the source of truth.

Sensor status should distinguish real values from trouble states such as:

```text
ok
stale
error
missing/unsupported
```

Do not use `0` as an error value. Zero is a perfectly respectable number and should not be framed for crimes committed by the transport layer.

## Dashboard asset direction

The v3 dashboard direction is asset-driven and photoreal-friendly.

Common render pattern:

```text
asset background
+ value/state-driven dynamic layer
+ optional foreground/glass/bezel overlay
= rendered widget
```

The active example dashboard uses self-contained gauge packages under `examples/assets/gauges/**/gauge.yaml`. Gauge widgets place packages; gauge packages own their sensor binding, value formatting/mapping, visual layers, and package-local geometry.

For numeric gauge packages that use seven-segment artwork, digit positions are artwork-alignment coordinates. They may look larger than the declared logical package size because the source artwork and the dashboard fit box are not always the same coordinate system. The rendered result and package comments are the authority.

## Documentation

Useful docs live under:

```text
docs/v3.3/
docs/v3.2/
docs/archive/
```

The v3.2 docs describe the final supported Fyne dashboard line. The active v3.3 docs describe the Ebiten-first renderer path and the active renderer boundary.

## Build

From the repository root:

```bash
go mod tidy
go build ./cmd/GoGauges
```

The binary will be written to the current directory as `GoGauges` unless you pass `-o`.

## Baseline dashboard harness

From the repository root:

```bash
go run ./cmd/GoGauges dashboard harness vw_caddy \
  --config ./examples/baseline-dashboard.yaml \
  --pattern sweep \
  --interval 50ms \
  --duration 60s \
  --renderer ebiten
```

`--renderer ebiten` is explicit for readability. Ebiten is already the default renderer in the active v3.4 dashboard command path.

With `--pattern sweep`, the harness is now gauge-aware: numeric and odometer sources walk from `-20` to `+30`, radial and segmented sources keep the full-range sweep, indicator sources flash, and bar sources pulse at 90 bpm.

## Dashboard overview

To inspect the resolved dashboard config without dumping the whole YAML:

```bash
go run ./cmd/GoGauges dashboard --config ./examples/baseline-dashboard.yaml
```

The bare `dashboard` command prints a compact overview of vehicles, attached dashboards, widget/gauge sources, and OBD-backed PIDs. It is a map, not the territory, but it is at least the correct map.

## Gauge preview

To inspect one gauge manually without live OBD input or the full harness:

```bash
go run ./cmd/GoGauges dashboard preview \
  ./examples/gauge-realism/radial/00-baseline.yaml
```

Useful controls:

- Left/Right jump to min/max.
- Up/Down step the value.
- Shift + Up/Down uses a coarse step.
- Ctrl/Cmd + Up/Down uses a fine step.
- `R` resets to midpoint.
- `Space` replays the last transition.
- `Esc` or `Q` quits.

Baseline preview files live under `examples/gauge-realism/` for `radial`, `numeric`, `odometer`, `bar`, `indicator`, and `segmented`.

## Raspberry Pi notes

The active v3.3 dashboard renderer is Ebiten. Raspberry Pi builds should focus on Go, graphics/display dependencies needed by Ebiten, and the selected kiosk/display setup.

## OBD transport

The intended v3 model is that GoGauges connects to an OBD-like endpoint declared by the selected vehicle:

```yaml
vehicles:
  vw_caddy:
    name: "VW Caddy"
    obd:
      address: "serial:///dev/ttyUSB0"
      timeout: 1000
```

For bench or harness work, use `dashboard harness` and the reusable baseline config instead of requiring live OBD hardware.
