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
capability: "CAP-002"
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

# Behavioral Contract BC-2.02.001: Posture Store Initialization

## Description

`init` creates a Posture Store for exactly one Site, pinned to a validated catalog version. Initialization is refused over an existing store — it can never destroy data.

## Preconditions

1. Target directory exists and is writable; catalog resolves and validates (BC-1.01.002).
2. No Posture Store already exists at the target (detected by the store marker file).

## Postconditions

1. Store created containing: Site record (name, sector, created_at), empty Assessment pinned to `(catalog_id, version, digest)`, empty snapshot history, store schema version marker.
2. The store is immediately loadable (BC-2.02.004) and scoreable (DI-011 — all goals `Unknown`).
3. Site record contains no personal-data fields; user docs note site metadata is non-PII (DI-013).

## Invariants

1. `init` over an existing store exits with E-STORE-001 and changes nothing — no force-overwrite flag exists for `init`.
2. All files created in one atomic sequence; a failed init leaves no partial store (FM-003 discipline).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Target directory contains unrelated files but no store marker | Init proceeds; only store files are created; unrelated files untouched |
| EC-002 | Init interrupted (kill) mid-creation | On next run: leftover temp files detected and cleaned; init re-runnable; never half-initialized |
| EC-003 | Site name containing path separators or control characters | Rejected E-VAL-002 (it becomes file content, not paths — but control chars break human-readable diffs) |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| `init --site "Mill Creek WTP" --sector water` in empty dir | Store created; load succeeds; score = 0%, 34 Unknown | happy-path |
| `init` in directory with existing store | E-STORE-001, exit ≠ 0, store unmodified (byte-identical) | error |
| `init` with unwritable target | E-FS-001 with path and permission detail | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-010 | Init is atomic: post-state is exactly {no store, complete store} | proptest (fault injection) |
| VP-011 | Init never deletes or modifies pre-existing non-store files | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-002 |
| L2 Domain Invariants | DI-001, DI-007, DI-013 |
| Architecture Module | op-store (module-decomposition.md) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-2.02.002 — depends on (uses atomic write discipline)
- BC-3.03.001 — blocks (answers require an initialized store)
