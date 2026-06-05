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

# Behavioral Contract BC-3.03.001: Record Answer

## Description

Recording an answer sets one Goal's Implementation Level in the Assessment, with optional note, stamped with `answered_at`. The atomic unit of posture data entry — valid levels only, valid goals only, durable immediately.

## Preconditions

1. Store initialized and lock held; goal_id exists in the pinned catalog version.
2. Level ∈ {not, partial, large, full, na, unknown} (canonical CLI tokens for NI/PI/LI/FI/NA/Unknown).

## Postconditions

1. Assessment contains the Answer with current `answered_at`; previous answer for the goal (if any) is replaced — the Assessment holds current state only (history lives in snapshots).
2. The write is durable before the command returns (BC-2.02.002, FM-007).
3. Explicitly setting `unknown` is recorded as an explicit answer (with timestamp) — distinct from never-answered, which has no Answer record (both score identically per DI-011).

## Invariants

1. No answer field carries personal identification; notes are documented non-PII (DI-013).
2. An Answer never references a goal absent from the pinned catalog (DEC-011). Enforced at recording time; answers found orphaned at load (hand edits, out-of-band catalog swaps) are quarantined per BC-2.02.004 EC-004 and never enter the scoring answer set — so VP-030 holds at the scoring boundary regardless of on-disk state.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | goal_id with wrong case or hyphen variant ("1a", "1-A") | Normalized to canonical ID when unambiguous; otherwise E-VAL-004 with nearest-match suggestion (DEC-011) |
| EC-002 | Unknown goal_id ("7.Z") | E-VAL-004 with suggestion list; nothing written |
| EC-003 | Re-answering with the identical level | Accepted; `answered_at` refreshed (it's a re-confirmation — relevant to future staleness, CAP-012) |
| EC-004 | Note exceeding the documented size cap (16 KiB) | E-VAL-005; caps keep the store diffable |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| `answer 1.A full --note "CISO charter signed"` | Answer recorded; durable; visible in next status | happy-path |
| `answer 1.A largeish` | E-VAL-006 invalid level, lists valid tokens | error |
| `answer 9.Q full` | E-VAL-004 with suggestions | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-030 | Assessment never contains an orphan goal_id | proptest |
| VP-031 | Level parsing accepts exactly the documented token set | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-003 |
| L2 Domain Invariants | DI-011, DI-013 |
| L2 Edge Cases | DEC-011 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-3.03.002 — composes with (guided session calls this per answer)
- BC-2.02.002 — depends on (durability)
