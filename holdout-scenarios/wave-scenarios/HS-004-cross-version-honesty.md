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
id: "HS-004"
category: "integration-boundaries"
must_pass: "true"
priority: "must-pass"
epic_id: "E-3"
behavioral_contracts: [BC-4.04.007, BC-2.02.006, BC-2.02.007]
lifecycle_status: active
introduced: v0.1.0-greenfield
last_evaluated: null
staleness_check: null
stale_reason: null
retired: null
assumption_source: null
risk_source: "R-003"
---

# Holdout Scenario: A Framework Upgrade Never Produces a Quietly Wrong Trend

> NEVER show to implementer/test-writer agents.

## Scenario

1. A store has 6 snapshots under catalog v2.0.0. The user migrates to a v2.1.0 catalog **without** a mapping, confirming a series break; takes 2 more snapshots.
2. The user requests `diff 3 8` (spanning the break), then `status`.
3. The diff is refused with the break named, both versions, and the migrate remedy — no number is produced; status renders the trend as two segments with a visible discontinuity. A later re-migration **with** a mapping bridges the break: the same diff now succeeds using values labeled as derived, and the original snapshots remain byte-identical.

## Behavioral Contract Linkage

| BC ID | Clause Tested | Scenario Aspect |
|-------|--------------|-----------------|
| BC-4.04.007 | postcondition 2 | typed refusal, no bypass |
| BC-2.02.007 | postconditions 1–3, EC-002 | break recording, rendering, bridging |
| BC-2.02.006 | postconditions 1–2 | back-cast alongside originals |

## Verification Approach

Script the two-phase migration; hash snapshot records before/after; attempt the cross-break diff in both phases (human + --json); inspect status trend rendering and the derived-values label.

## Evaluation Rubric

Functional correctness 0.4; error quality 0.3 (refusal names break, versions, remedy); data integrity 0.3 (originals untouched; derived clearly labeled).

## Edge Conditions

Scripted (non-TTY) migration without `--series-break` must refuse (E-VAL-010); diff within one segment must work throughout.

## Failure Guidance

"HOLDOUT LOW: HS-004 (satisfaction: 0.XX) -- a cross-version comparison produced an unlabeled or unrefused number, or migration altered original history."
