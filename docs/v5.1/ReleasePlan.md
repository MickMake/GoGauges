# v4.1 Release Plan

Status: planned

## Theme

Odometer and image-based numeric display realism.

All behaviour, configuration, constraints and non-goals are defined only in `docs/Designs/`.

## Release scope

| Slice | Name | Design |
|---|---|---|
| v4.1.0 | Release planning docs | Documentation-only |
| v4.1.1 | Odometer backlash | `../Designs/Gauge/rolling_drum_or_counter/quirks/custom_backlash.md` |
| v4.1.2 | Per-digit response lag | `../Designs/Gauge/segmented_display/quirks/custom_per_digit_response_lag.md` |
| v4.1.3 | Leading-zero behaviour | `../Designs/Gauge/segmented_display/quirks/custom_leading_zero_behaviour.md` |
| v4.1.4 | Digit bleed | `../Designs/Gauge/segmented_display/quirks/custom_digit_bleed.md` |
| v4.1.5 | Ghosting | `../Designs/Gauge/segmented_display/quirks/custom_ghosting.md` |
| v4.1.6 | Uneven brightness | `../Designs/Gauge/segmented_display/quirks/custom_uneven_brightness.md` |
| v4.1.7 | Load sag | `../Designs/Gauge/segmented_display/quirks/custom_load_sag.md` |
| v4.1.8 | Tests, previews and docs checkpoint | Relevant v4.1 Design documents |

## Release boundaries

- One slice per branch and pull request.
- Do not implement later slices early.
- Do not add unrelated realism features.
- Do not redesign existing renderers.
- Any design change must be made in `docs/Designs/` before implementation.

## Completion

v4.1 is complete when every slice is implemented, tested, documented and marked complete in `ImplementationState.md` and `docs/Status.md`.
