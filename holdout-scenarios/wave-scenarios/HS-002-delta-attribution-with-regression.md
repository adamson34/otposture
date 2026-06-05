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
id: "HS-002"
category: "behavioral-subtleties"
must_pass: "true"
priority: "must-pass"
epic_id: "E-1"
behavioral_contracts: [BC-4.04.006, BC-5.05.002]
lifecycle_status: active
introduced: v0.1.0-greenfield
last_evaluated: null
staleness_check: null
stale_reason: null
retired: null
assumption_source: null
risk_source: "R-005"
---

# Holdout Scenario: Every Percent of Movement Has a Name — Including the Embarrassing Ones

> NEVER show to implementer/test-writer agents.

## Scenario

1. Snapshot A exists; between A and B the user improves three goals, regresses one (full → not — staff departure), and marks one previously-answered goal N/A.
2. The user runs `diff A B` (human and `--json`).
3. The displayed drivers (signed) plus a labeled composition adjustment sum exactly to the displayed total; the regression appears at its magnitude-ordered position with no visual burial; the N/A transition is explained as composition, not as a level driver.

## Behavioral Contract Linkage

| BC ID | Clause Tested | Scenario Aspect |
|-------|--------------|-----------------|
| BC-4.04.006 | postconditions 2–3, invariant 1 | exact + display reconciliation; regression prominence |
| BC-5.05.002 | postconditions 2–3 | rendering of drivers + composition row |

## Verification Approach

Script the store mutations; capture `diff` output; parse the driver table and re-add the numbers (display) and the JSON exact rationals (full precision). Check ordering by |points| and the composition row label.

## Evaluation Rubric

Functional correctness 0.4 (reconciliation both layers); edge handling 0.3 (regression + N/A composition); data integrity 0.3 (JSON exact fields sum to delta exact).

## Edge Conditions

Drivers that each display-round to 0% while the total shows 1% (largest-remainder allocation must reconcile).

## Failure Guidance

"HOLDOUT LOW: HS-002 (satisfaction: 0.XX) -- delta drivers did not reconcile to the total, or a regression/composition change was hidden or mislabeled."
