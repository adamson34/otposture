---
document_type: holdout-scenario
level: ops
version: "1.0"
status: draft
producer: product-owner
timestamp: 2026-06-05T15:44:03
inputs: [stories/, behavioral-contracts/, prd.md]
input-hash: "56ca7a142db7b60de70a3281283396cb"
traces_to: ""
id: "HS-001"
category: "behavioral-subtleties"
must_pass: "true"
priority: "must-pass"
epic_id: "E-1"
behavioral_contracts: [BC-4.04.004, BC-4.04.003]
lifecycle_status: active
introduced: v0.1.0-greenfield
last_evaluated: null
staleness_check: null
stale_reason: null
retired: null
assumption_source: null
risk_source: "R-002"
---

# Holdout Scenario: A Near-Perfect Plant With One Missing Critical Control Cannot Look Good

> NEVER show to implementer/test-writer agents.

## Scenario

1. A store where 33 of 34 goals are answered Fully Implemented; the remaining goal is Critical-tier and answered Not Implemented.
2. The user runs `status` and generates the exec report.
3. The headline band shown everywhere is capped at the second-lowest band, with the gap named, while the percentage still shows the (high) true value with explicit explanation; nothing anywhere displays the uncapped band without its cap.

## Behavioral Contract Linkage

| BC ID | Clause Tested | Scenario Aspect |
|-------|--------------|-----------------|
| BC-4.04.004 | postcondition 1, invariant 2 | cap + exhaustive gap naming |
| BC-4.04.003 | postcondition 2 | band_uncapped retained but never solo |

## Verification Approach

Build a store via `init`/`answer` (33×full + 1 critical not), then run `status`, `status --json`, `report --exec`. Inspect: band field, cap explanation, gap list presence in all three surfaces; JSON `band_uncapped` vs `band`; HTML cap banner.

## Evaluation Rubric

Functional correctness 0.5 (cap applied in all surfaces); edge handling 0.2 (gap list exhaustive + tier source); error quality 0.1; data integrity 0.2 (JSON/human/HTML agree).

## Edge Conditions

Also probe: the same store with the critical goal Unknown instead of Not Implemented (must gate identically); and with the goal overridden to Standard (cap lifts, override disclosed everywhere).

## Failure Guidance

"HOLDOUT LOW: HS-001 (satisfaction: 0.XX) -- a missing critical control failed to cap the headline, or the cap was displayed inconsistently across surfaces."
