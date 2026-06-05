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
subsystem: "SS-04"
capability: "CAP-006"
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

# Behavioral Contract BC-4.04.007: Comparability Guard

## Description

Before any Delta is computed, both snapshots must be provably comparable: same catalog `(id, version, digest)`, or bridged by back-cast values, with no unbridged Series Break between them (DI-001, DEC-003). The guard is the only gateway to Delta computation — there is no bypass.

## Preconditions

1. Two snapshot references (by seq, label, or date-resolution) from the same store.

## Postconditions

1. Comparable: returns the effective answer/score pair to diff (originals, or back-cast values when bridging versions — the output states which).
2. Not comparable: E-SCORE-002 naming the precise reason — version mismatch (with both versions), digest mismatch (E-CAT-004 path), or unbridged Series Break (with break details) — plus the remedy (`migrate`).

## Invariants

1. No public API yields a Delta without passing this guard (single chokepoint, verifiable by architecture).
2. Comparisons using back-cast values always label the result as derived in every downstream surface.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Same version, same digest, break recorded between (user forced a break for a reason) | Not comparable — breaks dominate; reason displayed |
| EC-002 | Date-based reference resolving ambiguously (two snapshots same day, DEC-006) | E-VAL-009 listing candidates by seq; user picks |
| EC-003 | from_seq > to_seq (reversed order) | Valid — computes the reverse delta with correct signs; display marks direction |
| EC-004 | Snapshot references identical (seq N vs seq N) | Valid trivially; zero delta |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| diff 1 5, same catalog throughout | Comparable; originals used | happy-path |
| diff across migrated boundary with back-cast | Comparable; derived values; labeled | happy-path |
| diff across unbridged break | E-SCORE-002 with remedy | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-056 | Guard soundness: any pair it admits shares an effective catalog digest | kani/proptest |
| VP-057 | Guard is the unique entry to delta computation (no alternate code path) | manual/architecture review |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-006, CAP-011 |
| L2 Domain Invariants | DI-001 |
| L2 Edge Cases | DEC-003, DEC-006 |
| Architecture Module | op-score (module-decomposition.md) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-2.02.006/007 — depends on (back-cast and break records)
- BC-4.04.006 — blocks (delta computation)
