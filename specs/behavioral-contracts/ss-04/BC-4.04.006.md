---
document_type: behavioral-contract
level: L3
version: "1.1"
status: draft
producer: product-owner
timestamp: 2026-06-05T12:50:04
phase: 1a
inputs: [domain-spec/L2-INDEX.md, product-brief.md]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: domain-spec/L2-INDEX.md
origin: greenfield
subsystem: "SS-04"
capability: "CAP-006"
lifecycle_status: active
introduced: v0.1.0
modified: []
deprecated: null
deprecated_by: null
replacement: null
retired: null
removed: null
removal_reason: null
---

# Behavioral Contract BC-4.04.006: Delta Computation with Driver Attribution

## Description

A Delta between two comparable snapshots decomposes the total percent change into per-goal Drivers that sum exactly to the total (DI-005). "▲ +8%: +5% IDS deployment, +3% inventory refresh" — every movement names its cause, including regressions.

## Preconditions

1. Two snapshots passing the comparability guard (BC-4.04.007).

## Postconditions

1. Delta contains: `percent_delta` (full precision), `band_change` (from→to incl. floor states), Drivers[] = every goal whose level changed, each with from/to levels and signed `points = weight_g × (value_to − value_from) / Σ(weight_applicable_to)`.
2. Exact reconciliation at full precision: Σ driver points + composition_adjustment = percent_delta, where `composition_adjustment` is a separately displayed term arising when the applicable set changed (N/A transitions changing the denominator). No unexplained remainder, ever.
3. Display rounding (whole percents) allocates rounding residue by largest-remainder so the displayed driver list reconciles to the displayed total (DEC-008).

## Invariants

1. Negative drivers carry equal prominence in ordering rules (sorted by |points| descending) — regressions cannot be buried (DEC-009).
2. Goals with unchanged levels never appear as drivers; an empty driver set occurs iff percent_delta = 0 and no composition change.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Goal transitions to/from N/A between snapshots | Appears under composition_adjustment with explanation, not as a level driver |
| EC-002 | Band changed solely by floor rule (level change on one Critical goal), percent nearly flat | band_change shows "Managed → Developing (floor)"; the critical driver listed even if its points are small (DEC-010) |
| EC-003 | Displayed drivers each round to 0% but total rounds to 1% | Largest-remainder allocation gives one driver the 1%; reconciliation holds |
| EC-004 | Tier override changed between snapshots, no level changes | percent_delta 0; band may differ via floor; Delta reports "override-driven band change" with the override cited |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| 2.E: NI→FI (w=1), 34 equal weights | One driver +1/34; percent_delta = +0.0294…; displays as "+3%" | happy-path |
| 3 ups, 1 down | 4 drivers, signed; exact sum | happy-path |
| Goal FI→N/A only | Zero level-drivers; composition_adjustment explains the percent shift | edge-case |
| Identical snapshots | percent_delta 0, empty drivers, "no change" | edge-case |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-053 | Reconciliation: Σ points + composition_adjustment = percent_delta (full precision, all input pairs) | kani |
| VP-054 | Display reconciliation under largest-remainder rounding | proptest |
| VP-055 | Driver set = exactly the changed-level goals | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-006 |
| L2 Domain Invariants | DI-005 |
| L2 Edge Cases | DEC-008, DEC-009, DEC-010 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-4.04.007 — depends on (comparability)
- BC-5.05.002 — blocks (diff display)
