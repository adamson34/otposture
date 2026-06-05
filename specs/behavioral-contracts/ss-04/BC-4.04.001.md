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

# Behavioral Contract BC-4.04.001: Weighted Percent Computation

## Description

The single scoring formula (DI-012): `percent = Σ(weight_g × value_g) / Σ(weight_g)` over applicable goals. Level values are **exact rationals** — NI=0, PI=1/3 (displays as 0.33), LI=2/3 (displays as 0.67), FI=1, Unknown=0; arithmetic is rational/fixed-point, never binary floating point. N/A excluded entirely (DI-004). This is the most formally-verified function in the product.

## Preconditions

1. Validated catalog (weights > 0); answer set (explicit + implicit Unknown).

## Postconditions

1. Returns overall percent in [0,1] at full internal precision (display rounding is presentation-layer only, DEC-008).
2. Applicable set = all catalog goals minus N/A-answered goals; if the applicable set is empty, percent is undefined (surfaces render "N/A", DEC-002 generalization).

## Invariants

1. Pure function: no I/O, no clock, no global state — input fully determines output.
2. Monotonicity: raising any single goal's level (with all else fixed) never lowers the percent.
3. Bounds: 0 ≤ percent ≤ 1 for every input; percent = 1 iff every applicable goal is FI.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | All applicable goals N/A | Undefined percent; explicit "not scoreable" result, never 0% or 100% |
| EC-002 | Single applicable goal | percent = that goal's level value |
| EC-003 | Extreme weight skew (one weight 1000× others) | Formula holds; no overflow/precision loss (fixed-point or rational arithmetic) |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| 2 goals, equal weight, FI + NI | 0.50 | happy-path |
| 3 goals w=1,1,2: FI, PI, N/A | (1 + 1/3)/2 = 2/3 exactly (displays "67%") | edge-case |
| 34-goal catalog, all Unknown | 0.0 | edge-case |
| Weight = 0 in input | Unreachable (BC-1.01.002); engine asserts E-SCORE-001 if violated | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-040 | Bounds: result ∈ [0,1] for all valid inputs | kani |
| VP-041 | Monotonicity in any single level | kani/proptest |
| VP-042 | N/A exclusion: result independent of N/A goals' weights | proptest |
| VP-043 | Permutation invariance: goal order never affects the result | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-004 |
| L2 Domain Invariants | DI-004, DI-010, DI-012 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-4.04.002 — composes with (same formula per function)
- BC-3.03.003 — depends on (Unknown semantics)
