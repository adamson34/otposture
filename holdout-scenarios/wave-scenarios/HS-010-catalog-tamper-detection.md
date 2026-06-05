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
id: "HS-010"
category: "security-probes"
must_pass: "true"
priority: "must-pass"
epic_id: "E-2"
behavioral_contracts: [BC-1.01.003, BC-4.04.007]
lifecycle_status: active
introduced: v0.1.0-greenfield
last_evaluated: null
staleness_check: null
stale_reason: null
retired: null
assumption_source: null
risk_source: "R-004"
---

# Holdout Scenario: A Doctored Catalog Claiming to Be CPG 2.0 Is Caught

> NEVER show to implementer/test-writer agents.

## Scenario

1. A store has history pinned to the bundled cisa-cpg 2.0.0. Someone supplies a modified catalog file via `--catalog` — same `catalog_id` and `version`, but one goal's tier silently changed from Critical to Standard.
2. The user runs `status` and `diff` against history with the doctored catalog.
3. The digest mismatch is detected (E-CAT-004); comparability operations against pinned history refuse rather than silently rescoring under altered rules; a cosmetically re-formatted (but semantically identical) catalog produces the same digest and is accepted.

## Behavioral Contract Linkage

| BC ID | Clause Tested | Scenario Aspect |
|-------|--------------|-----------------|
| BC-1.01.003 | postcondition 2, EC-001/EC-002 | content-bound identity both directions |
| BC-4.04.007 | postcondition 2 | digest path in the guard |

## Verification Approach

Create both variants (semantic edit; whitespace/key-order-only edit) from the bundled catalog; run status/diff; assert E-CAT-004 + exit 6 for the doctored one and clean behavior for the reformatted one.

## Evaluation Rubric

Functional correctness 0.5 (detect true tamper, accept benign re-encoding); error quality 0.3 (names the mismatch and remedy); data integrity 0.2 (history never rescored under doctored rules).

## Edge Conditions

Doctored catalog also failing validation (validation error must not mask the clearer story); tampered catalog used at `init` (allowed — new store pins what it's given; digest recorded honestly).

## Failure Guidance

"HOLDOUT LOW: HS-010 (satisfaction: 0.XX) -- a content-altered catalog was accepted against pinned history, or a benign re-encoding was rejected."
