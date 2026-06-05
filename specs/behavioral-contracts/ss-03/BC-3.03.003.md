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
subsystem: "SS-03"
capability: "CAP-003"
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

# Behavioral Contract BC-3.03.003: Unknown-by-Default Semantics

## Description

Goals without an explicit Answer behave as `Unknown` everywhere: scored 0, flagged distinctly from `Not Implemented`, counted prominently (DI-011). The tool never blocks on incompleteness — first scores are honest-and-low, which is the designed onboarding experience ("don't know where to start" → start anywhere).

## Preconditions

1. Any scoring, status, diff, or report operation runs over an Assessment with unanswered goals.

## Postconditions

1. Every unanswered goal contributes 0 to its function score with full weight in the denominator (not excluded like N/A — DI-004).
2. All surfaces show the Unknown count and distinguish "never answered" from "explicitly marked unknown" in detail views.
3. A Critical-tier goal that is Unknown triggers the floor rule identically to Not Implemented (DI-003).

## Invariants

1. There is no state in which an unanswered goal silently inflates a score (no "skip = N/A" behavior anywhere).
2. Unknown count of 0 is achievable only by explicitly answering all 34 goals.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Status on a store with 34/34 Unknown | Score 0%, Foundational, "0 of 34 assessed" leading the output (DEC-004) |
| EC-002 | User tries `answer 1.A skip` expecting N/A semantics | `skip` is not a level token; E-VAL-006 explains unknown vs. na distinction |
| EC-003 | Exec report on a heavily-Unknown store | Report renders with an explicit data-completeness caveat box — never a confident-looking 12% without context |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| 17 of 34 answered `full`, rest untouched, equal weights | Score = 50%; Unknown count 17 displayed | happy-path |
| Critical goal left unanswered, everything else `full` | Floor rule active: band capped at Developing, gap named | edge-case |
| Compare "explicit unknown" vs "never answered" in scoring | Identical scores; different detail-view annotation | edge-case |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-033 | Adding an answer can only increase or hold the score, never decrease it, when the answer level > 0 and no other state changes | proptest |
| VP-034 | Unknown and Not Implemented are score-identical but flag-distinct | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-003, CAP-004 |
| L2 Domain Invariants | DI-003, DI-004, DI-011 |
| L2 Edge Cases | DEC-004 |
| Architecture Module | op-assess (module-decomposition.md) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-4.04.001 — composes with (valuation rules)
- BC-4.04.004 — composes with (floor rule on Unknown criticals)
