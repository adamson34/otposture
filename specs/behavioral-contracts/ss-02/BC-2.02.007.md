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
subsystem: "SS-02"
capability: "CAP-011"
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

# Behavioral Contract BC-2.02.007: Series Break Recording and Enforcement

## Description

When history cannot (or the user chooses not to) be back-cast across a catalog change, a Series Break is recorded at the transition point. Breaks make non-comparability explicit and permanent — the honest alternative to a silently shifted rubric (DEC-003).

## Preconditions

1. A catalog version change is being applied without (full) back-cast (BC-2.02.006 EC-003, or user opts out).

## Postconditions

1. A SeriesBreak record persisted: `at_seq` (last snapshot of the old series), `from_version`, `to_version`, `reason`, timestamp.
2. All subsequent comparability checks (BC-4.04.007) treat snapshot pairs spanning the break as non-comparable.
3. Status/report trend rendering shows the break as a visible discontinuity marker with the version pair.

## Invariants

1. Series breaks are append-only and never removed — a later back-cast may *bridge* a break (adding comparability) but the break record remains as history.
2. Creating a break requires explicit confirmation in interactive use, or an explicit flag in scripted use — never implicit.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Multiple breaks in one store's history | Each segment is internally comparable; trend renders per-segment |
| EC-002 | Later migration supplies a mapping bridging an old break | Back-cast records added; break remains recorded but marked bridged; cross-break diffs now permitted via back-cast values |
| EC-003 | Diff explicitly requested across an unbridged break | E-SCORE-002 refusal naming the break, versions, and the `migrate` remedy (DEC-003) |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Migrate without mapping; confirm break | Break recorded; status shows discontinuity | happy-path |
| `diff 3 8` where break at seq 5 | E-SCORE-002 with remedy text | error |
| Bridge break via later mapping; `diff 3 8` | Succeeds using back-cast values; output notes derivation | edge-case |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-021 | No comparability path exists across an unbridged break (no API returns a cross-break Delta) | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-011 |
| L2 Domain Invariants | DI-001, DI-002 |
| L2 Edge Cases | DEC-003 |
| Architecture Module | op-store (module-decomposition.md) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-2.02.006 — composes with (back-cast vs. break decision)
- BC-4.04.007 — blocks (comparability guard consumes break records)
