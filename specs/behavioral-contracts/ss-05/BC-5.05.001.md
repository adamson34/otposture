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
capability: "CAP-007"
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

# Behavioral Contract BC-5.05.001: Status View

## Description

`status` is the at-a-glance terminal summary: band (with floor state), percent, unknown count, trend vs. baseline/last snapshot, biggest movers, open critical gaps, active overrides, and top next-best-actions. Honest by construction — every disclosure rule lands here first.

## Preconditions

1. Store loads (BC-2.02.004). Works with zero snapshots (scores the live Assessment).

## Postconditions

1. Always shown: band (and "capped from X — floor rule" when triggered, with gap list), percent, assessed/total counts, catalog version, active override count.
2. With ≥1 snapshot: delta vs. latest snapshot ("uncommitted changes" concept); with ≥2: trend line/sparkline by seq with any Series Breaks rendered as visible discontinuities.
3. First-snapshot-only case: "baseline established <date>" with no fabricated trend (DEC-001).
4. Unknown count rendered prominently when > 0 — completeness before confidence (BC-3.03.003).

## Invariants

1. Status is read-only — never takes the lock, never writes (BC-2.02.005 inv. 1).
2. All numbers come from one scoring call (no mixed epochs, BC-4.04.002 inv. 2).
3. Output contains no personal data (DI-013).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Empty store (0 answers, 0 snapshots) | "0 of 34 assessed — run `otposture assess` to begin"; score shown as 0% honestly |
| EC-002 | Floor cleared since last snapshot by an override | Trend annotation: "band change driven by override (see rationale)" (DEC-005) |
| EC-003 | Narrow terminal (80 col) / no-color terminal / `NO_COLOR` env | Degrades to plain ASCII gracefully; same information content |
| EC-004 | Band fell while percent rose (DEC-010) | Both shown with the explanation line; no contradiction-looking output |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Store with 3 snapshots, improving | Band, %, ▲ delta, sparkline, movers, next actions | happy-path |
| Store with floor triggered | Cap disclosure + gaps above the fold | happy-path |
| Fresh store | Onboarding-toned zero state | edge-case |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-070 | Status emits floor/override disclosures whenever the underlying Score has them (no suppression path) | snapshot tests |
| VP-071 | Status performs zero write syscalls | manual/CI strace check |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-007 |
| L2 Domain Invariants | DI-013 |
| L2 Edge Cases | DEC-001, DEC-005, DEC-010 |
| Architecture Module | op-render (module-decomposition.md) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-4.04.002/004/006/008 — depends on (all computed inputs)
- BC-6.06.004 — related to (JSON twin of this view)
