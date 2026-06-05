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

# Behavioral Contract BC-2.02.009: Store Schema Upgrade

## Description

`upgrade` migrates the on-disk Posture Store schema to the running binary's version — the only operation permitted to rewrite store files wholesale. It is explicit, backed up, atomic as a unit, and content-preserving: every snapshot, answer, override, and break survives byte-equivalent in meaning.

## Preconditions

1. Store loads under the in-memory compatibility path (BC-2.02.004 EC-003); store schema version < binary schema version; lock held (BC-2.02.005).
2. Sufficient disk space for a full backup alongside the store.

## Postconditions

1. Prior store files are copied to a timestamped backup directory inside the store (e.g., `backup-schema-v1-<date>/`) **before** any rewrite; the backup is announced in output.
2. All store files are rewritten at the new schema version via the atomic discipline (BC-2.02.002); the upgraded store loads cleanly with zero warnings.
3. Semantic content is preserved exactly: snapshot count, seq values, answers, scores (recomputation still matches, DI-010), overrides, series breaks, and migration records are all unchanged in meaning.
4. Re-running `upgrade` on a current-version store is a no-op with notice (idempotent).

## Invariants

1. `upgrade` never runs implicitly — no other command rewrites the store schema on disk (BC-2.02.004 EC-003 keeps compatibility in memory only).
2. Append-only history semantics survive the rewrite: the upgraded snapshot records are semantically identical, and the backup preserves the original bytes (DI-002 honored across representations).
3. A failure at any point leaves either the original store intact (failure before switch-over) or the upgraded store complete (failure after) — never a mix; the backup exists in both outcomes.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Disk fills during backup copy | E-FS-002; original store untouched; partial backup directory removed best-effort |
| EC-002 | Kill -9 mid-rewrite | On next load: schema marker still old (switch-over is the last atomic step) ⇒ original intact; leftover temp artifacts cleaned with notice |
| EC-003 | Upgrade across multiple schema versions (v1 → v3) | Single command chains migrations internally; one backup of the original; intermediate forms never hit disk as the live store |
| EC-004 | Store contains quarantined orphan answers (E-STORE-005) | Carried through unchanged — upgrade neither resolves nor drops them |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| v1-schema store with 10 snapshots, binary at v2 | Backup created; store rewritten; `verify` clean; seqs/scores identical | happy-path |
| `upgrade` on current-version store | No-op notice; no backup created; exit 0 | edge-case |
| Fault injected between backup and switch-over | Original store loads; backup present; re-run succeeds | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-025 | Content preservation: parse(upgraded) is semantically equal to parse(original) for all valid stores | proptest |
| VP-026 | Crash safety: any single fault point yields original-intact or upgrade-complete, backup always present once switch-over begins | proptest (fault injection) |
| VP-027 | Idempotence: upgrade(upgrade(s)) = upgrade(s) | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-002 |
| L2 Domain Invariants | DI-002, DI-010 |
| L2 Failure Modes | FM-003 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-2.02.002 — depends on (atomic write discipline)
- BC-2.02.004 — composes with (EC-003 in-memory compatibility path; this is its on-disk counterpart)
- BC-6.06.001 — related to (command surface)
