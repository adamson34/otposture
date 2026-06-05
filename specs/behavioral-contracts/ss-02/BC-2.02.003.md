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
subsystem: "SS-02"
capability: "CAP-005"
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

# Behavioral Contract BC-2.02.003: Append-Only Snapshot Persistence

## Description

Snapshots are the immutable history (DI-002). Each `snapshot` appends a record with monotonic `seq`, frozen answers, catalog pin `(id, version, digest)`, computed Score, timestamp, and optional label. Nothing ever mutates or deletes an existing snapshot.

## Preconditions

1. Store initialized; scoring of the current Assessment succeeds (BC-4.04.005).

## Postconditions

1. New snapshot persisted with `seq = max(existing seq) + 1` (1 for the first).
2. Frozen answers are a deep copy — later Assessment edits never alter the snapshot.
3. The cached Score in the snapshot equals a fresh recomputation at write time (DI-010).
4. Zero-explicit-answer snapshot permitted with a prominent warning (DEC-004).

## Invariants

1. The snapshot file region for existing records is byte-stable across all subsequent operations (verifiable by hashing past records).
2. `seq` values are unique, contiguous, and strictly increasing; `taken_at` is informational only (DEC-006).
3. No mutation API for snapshots exists at any layer — including no "delete last snapshot" (mistakes are corrected by taking a new snapshot; history records what was believed when).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Snapshot taken when Assessment unchanged since last snapshot | Allowed; delta vs. previous = 0 with empty driver set (legitimate "re-confirmed" event) |
| EC-002 | `taken_at` earlier than previous snapshot's (clock moved back) | Append proceeds; warning emitted; ordering remains by `seq` (FM-005) |
| EC-003 | Label containing newline/control characters | Rejected E-VAL-003 (labels appear in reports and diffs) |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Two snapshots after answering 3 goals between them | seq 1,2; snapshot 1's answers unchanged by the edits | happy-path |
| Snapshot with zero explicit answers | Created; warning "34 Unknown"; Score 0%, Foundational | edge-case |
| Attempt to edit snapshot 1 via any CLI surface | No such operation exists; direct file tamper detected by BC-2.02.004 | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-014 | After any operation sequence, previously written snapshot bytes are unchanged | proptest |
| VP-015 | seq is a gapless strictly-increasing sequence | kani/proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-005 |
| L2 Domain Invariants | DI-001, DI-002, DI-010 |
| L2 Edge Cases | DEC-004, DEC-006 |
| Architecture Module | op-store (module-decomposition.md) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-1.01.003 — depends on (catalog identity pinning)
- BC-4.04.006 — blocks (deltas read snapshots)
