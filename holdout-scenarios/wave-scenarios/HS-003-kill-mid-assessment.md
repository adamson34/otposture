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
id: "HS-003"
category: "edge-case-combinations"
must_pass: "true"
priority: "must-pass"
epic_id: "E-4"
behavioral_contracts: [BC-3.03.002, BC-2.02.002]
lifecycle_status: active
introduced: v0.1.0-greenfield
last_evaluated: null
staleness_check: null
stale_reason: null
retired: null
assumption_source: null
risk_source: "R-006"
---

# Holdout Scenario: SSH Drops Mid-Assessment on a Plant Workstation

> NEVER show to implementer/test-writer agents.

## Scenario

1. A user is 7 confirmed answers into a guided `assess` session over SSH.
2. The connection dies (SIGKILL to the process) while the 8th prompt is on screen.
3. Re-running `assess` later: exactly 7 answers are durable, the session resumes at the remaining unanswered goals, the store loads with zero warnings, and any leftover lock is broken automatically with a notice.

## Behavioral Contract Linkage

| BC ID | Clause Tested | Scenario Aspect |
|-------|--------------|-----------------|
| BC-3.03.002 | postconditions 2–3, EC-002 | per-answer durability, resume |
| BC-2.02.002 | invariants | no partial writes from the kill |
| BC-2.02.005 | postcondition 3 | stale-lock self-clear |

## Verification Approach

Drive `assess` via a PTY harness; SIGKILL after the 7th confirmation; verify store contents (7 answers), `verify` clean, re-run resumes correctly, lock notice on next mutating command.

## Evaluation Rubric

Functional correctness 0.4; edge handling 0.3 (kill timing variations: during write, between prompts); error quality 0.2 (lock/resume messaging); data integrity 0.1.

## Edge Conditions

Repeat with kill injected during the answer-write window itself (the in-flight answer may be lost but never half-written); disk-full at the same moment.

## Failure Guidance

"HOLDOUT LOW: HS-003 (satisfaction: 0.XX) -- a killed session lost confirmed answers, corrupted the store, or failed to resume/unlock cleanly."
