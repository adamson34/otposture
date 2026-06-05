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
id: "HS-005"
category: "edge-case-combinations"
must_pass: "true"
priority: "must-pass"
epic_id: "E-3"
behavioral_contracts: [BC-2.02.004, BC-6.06.001]
lifecycle_status: active
introduced: v0.1.0-greenfield
last_evaluated: null
staleness_check: null
stale_reason: null
retired: null
assumption_source: null
risk_source: "R-006"
---

# Holdout Scenario: The Store Was Hand-Edited — Legitimately, Clumsily, and Maliciously

> NEVER show to implementer/test-writer agents.

## Scenario

1. Three copies of a healthy 10-snapshot store are tampered differently: (a) a legitimate hand-edit changing an answer level in assessment.toml; (b) a clumsy edit corrupting snapshot record 7's TOML syntax; (c) a quiet edit changing a frozen answer inside snapshot 4 without touching its cached score.
2. The user runs `status` and `verify` (and `verify --json`) on each.
3. (a) loads normally and scores the edited value; (b) loads snapshots 1–6 read-only, names record 7, refuses mutation until repaired; (c) loads, but `verify` flags snapshot 4's score mismatch (E-STORE-003) with engine-version context and exits 4.

## Behavioral Contract Linkage

| BC ID | Clause Tested | Scenario Aspect |
|-------|--------------|-----------------|
| BC-2.02.004 | postconditions 2–3 | salvage, named damage, recompute check |
| BC-6.06.001 | invariant 2 exception | verify exit 4 on findings |

## Verification Approach

Prepare the three tampered stores from one fixture; run status/verify; check exit codes (0 for (a), 4 for store-damage paths), salvage read-only behavior, JSON verify-results document shape (`ok: false`, mismatch entries).

## Evaluation Rubric

Functional correctness 0.4; error quality 0.3 (localization: file + record named, remedy present); edge handling 0.2 (legit edit NOT punished); data integrity 0.1.

## Edge Conditions

Tail-truncated final record (distinguished as incomplete-tail); orphan answer for a goal not in the catalog (quarantined E-STORE-005, listed, excluded from scoring).

## Failure Guidance

"HOLDOUT LOW: HS-005 (satisfaction: 0.XX) -- a tampered or damaged store was loaded as healthy, salvage lost recoverable history, or verify missed a score mismatch."
