# Gauge verification harness

The gauge verification harness belongs to GoGauge and tests widgets and bindings without requiring a vehicle or provider implementation.

## Synthetic cases

- minimum-to-maximum and maximum-to-minimum ramps
- step changes
- jitter and noisy values
- slow and rapid updates
- invalid state with and without a last-good value
- stale state
- disabled signal
- provider offline
- provider conflict
- missing signal
- engineering limits and out-of-range values

## Separation

Basic widget verification does not require:

- GoDriveLog
- CAN or OBD hardware
- a live MQTT broker

Separate integration tests may feed validated contract messages through the MQTT boundary and signal store. The harness must not teach widgets to parse MQTT directly.
