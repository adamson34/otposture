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
subsystem: "SS-01"
capability: "CAP-001"
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

# Behavioral Contract BC-1.01.002: Catalog Validation Rejects Defective Catalogs with Itemized Defects

## Description

Structural validation runs after parsing and before any catalog use. A defective catalog never enters scoring; the user receives every defect found in one pass, not just the first (DEC-007).

## Preconditions

1. A catalog has been parsed (BC-1.01.001) into candidate form.

## Postconditions

1. A valid catalog is returned for use; an invalid catalog yields E-CAT-003 with a defect list and a non-zero exit.
2. Each defect names: the rule violated, the offending element (`goal_id`/`function_id`/field), and the expected condition.
3. Validation checks at minimum: (a) unique `goal_id`s; (b) every `goal.function_id` resolves; (c) every `weight > 0`; (d) `default_tier` ∈ {Critical, High, Standard}; (e) exactly 4 band thresholds, strictly increasing, within [0,1]; (f) non-empty `licensing_provenance` (DI-009); (g) valid semver `version`; (h) impact/effort metadata present for every goal (CAP-009 dependency).

## Invariants

1. No partial catalog is ever observable by downstream subsystems — validation is all-or-nothing.
2. Validation is pure: no I/O beyond the already-read file content.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Catalog with 3 independent defects | All 3 reported in one invocation, machine-parseable order (stable sort by goal_id then rule) |
| EC-002 | Duplicate goal_id differing only in case ("1.A" vs "1.a") | Treated as duplicate — IDs are case-insensitive-unique, stored canonical-case |
| EC-003 | Band thresholds equal (non-strict ordering) | Defect (e); thresholds must be strictly increasing |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Bundled CPG 2.0 catalog | Zero defects | happy-path |
| Catalog with weight=0 on one goal and a dangling function_id | E-CAT-003; exactly 2 defects, each naming its element | error |
| Catalog missing `licensing_provenance` | E-CAT-003 citing DI-009 rule | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-003 | Validation verdict is deterministic and order-independent w.r.t. goal ordering in the file | proptest |
| VP-004 | Any catalog accepted by validation satisfies all eight rules (no false accept) | kani/proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-001 |
| L2 Domain Invariants | DI-008, DI-009 |
| L2 Edge Cases | DEC-007 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-1.01.001 — depends on (validates parser output)
- BC-4.04.001 — blocks (scoring assumes a validated catalog)
