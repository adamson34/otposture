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
id: "HS-009"
category: "behavioral-subtleties"
must_pass: "true"
priority: "must-pass"
epic_id: "E-5"
behavioral_contracts: [BC-3.03.003, BC-5.05.001, BC-5.05.003]
lifecycle_status: active
introduced: v0.1.0-greenfield
last_evaluated: null
staleness_check: null
stale_reason: null
retired: null
assumption_source: "ASM-004"
risk_source: null
---

# Holdout Scenario: Day One Is Honest, Not Discouraging — and Day Thirty Shows the Climb

> NEVER show to implementer/test-writer agents.

## Scenario

1. A brand-new user: `init`, immediate `status` (nothing answered), `snapshot` (baseline), then a first short `assess` session answering 9 goals, another snapshot, and a baseline-vs-now exec report.
2. The zero state shows 0%/lowest band/"0 of 34 assessed" with an onboarding pointer — no error, no fabricated trend; the first snapshot warns about all-Unknown (DEC-004) but succeeds.
3. After the session: status shows the delta with the 9 answers as drivers; the report leads with the data-completeness caveat (25 unknown) **before** any percentage; next-best-actions tells the user that assessing remaining Unknowns is the next action when they top the ranking.

## Behavioral Contract Linkage

| BC ID | Clause Tested | Scenario Aspect |
|-------|--------------|-----------------|
| BC-3.03.003 | postconditions + EC-001 | honest Unknown accounting |
| BC-5.05.001 | postcondition 3–4, EC-001 | zero/baseline states |
| BC-5.05.003 | EC-001/EC-002 | baseline framing + caveat-before-percent |
| BC-4.04.008 | EC-002 | unknown-assessment as next action |

## Verification Approach

Scripted first-day flow; assert zero-state copy and exit 0; check snapshot warning; parse the report section order (caveat box precedes the percent rendering); verify driver count = 9.

## Evaluation Rubric

Functional correctness 0.4; edge handling 0.2; error quality 0.2 (onboarding tone, no false errors); data integrity 0.2 (counts consistent across surfaces).

## Edge Conditions

`diff` with only one snapshot (clear explanation, no crash); session quit after 0 answers (nothing written).

## Failure Guidance

"HOLDOUT LOW: HS-009 (satisfaction: 0.XX) -- the new-user flow errored, fabricated confidence, or buried the completeness caveat."
