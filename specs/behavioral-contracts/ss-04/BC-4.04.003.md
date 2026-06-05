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
capability: "CAP-004"
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

# Behavioral Contract BC-4.04.003: Maturity Band Derivation

## Description

The headline band (Foundational / Developing / Managed / Resilient) derives from the percent via catalog-defined thresholds, then the floor rule (BC-4.04.004) may cap it. Band names and thresholds are catalog data (DI-008) — renameable without engine changes (ASM-006 hedge).

## Preconditions

1. Percent computed (BC-4.04.001); catalog provides 4 bands with strictly increasing thresholds `t1 < t2 < t3` partitioning [0,1].

## Postconditions

1. Pre-floor band: Foundational if percent < t1; Developing if t1 ≤ percent < t2; Managed if t2 ≤ percent < t3; Resilient if percent ≥ t3 (boundary belongs to the higher band).
2. Result carries both `band_uncapped` and final `band` so surfaces can show "Managed → capped to Developing (floor rule)".
3. Undefined percent (all N/A) yields no band — surfaces show "not scoreable".

## Invariants

1. Band derivation is total over defined percents and uses no state beyond (percent, thresholds, floor result).
2. Bands are ordered; capping only ever lowers, never raises.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | percent exactly equal to a threshold | Higher band (documented closed-lower-bound rule) |
| EC-002 | percent 0.0 / 1.0 | Foundational / Resilient respectively |
| EC-003 | Custom catalog with different band names | Engine renders catalog's names verbatim everywhere |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| thresholds (.40,.65,.85); percent .65 | Managed (boundary up) | happy-path |
| percent .649999 | Developing | edge-case |
| All N/A assessment | No band; "not scoreable" | edge-case |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-046 | Band function is monotone in percent | kani |
| VP-047 | Exactly one band for every defined percent (total, no gaps/overlaps) | kani |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-004 |
| L2 Domain Invariants | DI-008, DI-010 |
| L2 Assumptions | ASM-006 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-4.04.004 — composes with (cap applied after derivation)
