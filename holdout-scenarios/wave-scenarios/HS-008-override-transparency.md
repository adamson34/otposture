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
id: "HS-008"
category: "behavioral-subtleties"
must_pass: "true"
priority: "must-pass"
epic_id: "E-4"
behavioral_contracts: [BC-3.03.004, BC-5.05.001, BC-5.05.003]
lifecycle_status: active
introduced: v0.1.0-greenfield
last_evaluated: null
staleness_check: null
stale_reason: null
retired: null
assumption_source: null
risk_source: "R-002"
---

# Holdout Scenario: Gaming the Floor Rule Is Possible But Never Invisible

> NEVER show to implementer/test-writer agents.

## Scenario

1. A store where the floor rule is active due to one Critical goal at Not Implemented. The user overrides that goal's tier to Standard with rationale "vendor-managed, contract §4", takes a snapshot, and produces status, diff, and the exec report.
2. The band rises (cap lifted).
3. Every surface that shows the new band also shows the override (goal, new tier, rationale) and the diff explains the band change as override-driven with zero level drivers; the prior snapshot retains its capped band untouched; clearing the override (with rationale) restores the cap on the next score and records the clear event.

## Behavioral Contract Linkage

| BC ID | Clause Tested | Scenario Aspect |
|-------|--------------|-----------------|
| BC-3.03.004 | postconditions 1–3, EC-001 | lifecycle + supersede history |
| BC-5.05.001/002/003 | disclosure invariants | no surface omits the override |
| BC-4.04.006 | EC-004 | override-driven band change attribution |

## Verification Approach

Run the override sequence; grep/parse all three surfaces (+ JSON) for the override disclosure co-located with the band; verify prior snapshot's stored band via verify; attempt override without rationale (must E-VAL-008).

## Evaluation Rubric

Functional correctness 0.4 (disclosure chain complete); edge handling 0.3 (history retained, clear event); data integrity 0.3 (prior snapshots untouched).

## Edge Conditions

Two successive overrides on the same goal (only latest active, both in history); override on a goal that later becomes the only floor gap again.

## Failure Guidance

"HOLDOUT LOW: HS-008 (satisfaction: 0.XX) -- an override changed the headline somewhere without being disclosed alongside it, or history was retroactively altered."
