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
capability: "CAP-011"
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

# Behavioral Contract BC-2.02.006: Back-Cast Migration Storage

## Description

`migrate` to a new catalog version recomputes historical snapshots under the new rubric where a goal-mapping exists, storing back-cast values **alongside** originals (DI-002). Trend continuity survives framework upgrades without ever rewriting what was true.

## Preconditions

1. New catalog validates (BC-1.01.002); a goal-mapping table (old goal_id → new goal_id | dropped | split) is available for the version pair (shipped with the catalog or supplied).
2. Store lock held (BC-2.02.005).

## Postconditions

1. For each historical snapshot: a back-cast Score (and remapped answer view) under the new catalog is stored as a supplementary record keyed to the original `seq`; the original record is byte-unchanged.
2. The Assessment is remapped to the new catalog version; unmappable answers are preserved in an `orphaned` section with a migration note (DEC-012).
3. A migration record documents: version pair, mapping table digest, timestamp, per-goal disposition counts (mapped/dropped/split/orphaned overrides).

## Invariants

1. Originals are never modified or deleted; back-cast values are clearly marked as derived (DI-002).
2. Re-running the same migration is idempotent (same mapping ⇒ same back-cast values; duplicates refused).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Goal split (one old → two new) | Old answer back-casts to both new goals with a `derived-from-split` marker; flagged for re-assessment |
| EC-002 | Tier override on a dropped goal | Override preserved in originals, listed as orphaned in migration report (DEC-012); never silently discarded |
| EC-003 | No mapping table available for the version pair | Back-cast impossible: migration records a Series Break instead (BC-2.02.007) only after explicit interactive confirmation or the `--series-break` flag; non-interactive without the flag is refused with E-VAL-010 — breaks are never implicit |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Migrate 5-snapshot store cpg 2.0.0→2.1.0 with full mapping | 5 back-cast records; originals hash-identical; trend continuous | happy-path |
| Mapping drops 2 goals that had answers | Back-cast excludes them; migration report lists both | edge-case |
| Re-run identical migration | No-op with "already migrated" notice | edge-case |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-019 | Migration preserves original records byte-exactly | proptest |
| VP-020 | Back-cast scoring uses only mapped answers and the new catalog (no leakage from old weights) | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-011 |
| L2 Domain Invariants | DI-001, DI-002 |
| L2 Edge Cases | DEC-012 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-2.02.007 — composes with (fallback when unmappable)
- BC-4.04.007 — composes with (back-cast enables cross-version comparison)
