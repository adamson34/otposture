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
subsystem: "SS-05"
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

# Behavioral Contract BC-5.05.002: Diff Display

## Description

`diff <from> <to>` renders a Delta: headline change, full driver table (signed, sorted by |points|), composition adjustments, band transitions including floor states, and derivation labels for back-cast comparisons. The terminal twin of the exec report's "what changed" section.

## Preconditions

1. Two snapshot references resolving via the comparability guard (BC-4.04.007).

## Postconditions

1. Header: from/to (seq, date, label), percent from→to with signed delta, band from→to (floor states annotated).
2. Driver table: every driver with goal ID, title, level transition, signed display points; regressions visually marked but identically placed (DEC-009); composition adjustments as a distinct labeled row (BC-4.04.006).
3. Rounding reconciliation visible: displayed rows sum to the displayed total (DEC-008).
4. Back-cast comparisons carry a "values derived via migration vX→vY" banner.

## Invariants

1. Read-only; no lock.
2. No driver filtering by default — `--top N` truncation states "showing N of M drivers" (no silent caps).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Zero-change diff | "No changes between #3 and #4"; exit 0 |
| EC-002 | Pure-override band change (no level drivers) | Explanation row: "band change driven by tier override on 3.H (rationale shown)" |
| EC-003 | Reversed order args | Correct negative rendering with direction note (BC-4.04.007 EC-003) |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| diff 1 4 with 6 changed goals | 6 drivers, signed, sorted, reconciled | happy-path |
| diff with one regression among gains | Regression marked, placed by magnitude | happy-path |
| diff across unbridged break | E-SCORE-002 (guard) — display layer never reached | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-072 | Displayed driver sum equals displayed total (post-rounding) for all inputs | proptest (render-parse round trip) |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-006 |
| L2 Edge Cases | DEC-008, DEC-009 |
| Architecture Module | op-render (module-decomposition.md) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-4.04.006 — depends on (Delta data)
- BC-6.06.004 — related to (JSON twin)
