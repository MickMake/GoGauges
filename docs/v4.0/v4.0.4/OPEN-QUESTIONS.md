# GoGauges Core open questions and consolidation items

No unresolved internal Core design question remains after the Chat 4 discussion. The following item is accepted as a **proposed clarification** but is not yet normative until the final shared-contract consolidation pass.

## OQ-C001 — Exact signal-key set in complete state snapshots

**Affected authoritative sections:**

- `docs/v4.0/v4.0.2/contracts/mqtt-contract.md`, §6 — State publication
- `docs/v4.0/v4.0.2/contracts/signal-contract.md`, §5 — Signal state dimensions

**Proposed clarification:**

For every provider state snapshot:

- the set of keys in `signals` must exactly match the current catalogue signal IDs
- signals removed from a new catalogue must disappear from the next state snapshot
- unobserved signals must still be present
- disabled signals must still be present
- invalid signals must still be present
- consumers replace the previous provider snapshot rather than patching it

A state snapshot with a missing, additional or unknown signal key is rejected in full as malformed. It is not partially applied.

**Status:** Accepted in principle during Chat 4. Pending final consolidation into the authoritative contract. This document does not silently make it normative.

## Inherited-documentation corrections for final consolidation

The final consolidation pass must preserve these corrections without modifying the meaning of the shared contract:

1. Ignore GoDriveLog Chat 3 mandatory-recording decisions. GoGauges does not depend on recording being enabled or available.
2. Use `GoGauges` consistently. Earlier singular product wording is treated as an error.
3. Keep Chat 4 files only beneath `docs/v4.0/v4.0.4/`.
4. Refer to the current authoritative contract at `docs/v4.0/v4.0.2/contracts/`.
5. Accept all four external health values: `ok`, `degraded`, `error`, `unknown`.
