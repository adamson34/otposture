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

# Behavioral Contract BC-4.04.002: Per-Function Sub-Scores

## Description

Each catalog Function gets a sub-score using the identical formula (BC-4.04.001) restricted to its goals. Sub-scores prevent the roll-up from hiding where the weakness lives — the C2M2 lesson encoded.

## Preconditions

1. Same as BC-4.04.001; functions partition the goal set.

## Postconditions

1. One sub-score per function with ≥1 applicable goal; functions with zero applicable goals (all N/A) report "N/A" and are excluded from the overall denominator entirely (DEC-002).
2. The overall percent equals the weighted combination of function sub-scores (weights = Σ goal weights per function) — consistency between views is exact, not approximate.

## Invariants

1. Same purity, bounds, and monotonicity as BC-4.04.001, per function.
2. Function sub-scores and overall score always derive from the same answer set in the same call — no surface can mix epochs.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | RECOVER function (1 goal) answered N/A | RECOVER shows "N/A"; overall excludes its weight; no division by zero |
| EC-002 | Every function N/A except one | Overall equals that function's sub-score |
| EC-003 | Function sub-scores all ≥ band threshold but overall below (weight distribution) | Impossible by construction (postcondition 2) — consistency property |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| CPG catalog: PROTECT all FI, others all NI | PROTECT 1.0; others 0.0; overall = PROTECT weight share | happy-path |
| GOVERN goals all N/A, rest mixed | GOVERN "N/A"; overall over remaining 29 goals | edge-case |
| Empty function in a custom catalog | Caught at catalog validation (BC-1.01.002); engine asserts otherwise | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-044 | Decomposition identity: overall = Σ(funcWeight × funcScore)/Σ(funcWeight) over scoreable functions | kani/proptest |
| VP-045 | Sub-score independence: changing answers in function A never changes function B's sub-score | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-004 |
| L2 Domain Invariants | DI-004, DI-012 |
| L2 Edge Cases | DEC-002 |
| Architecture Module | op-score (module-decomposition.md) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-4.04.001 — depends on (shared formula)
- BC-5.05.001 — blocks (status renders sub-scores)
