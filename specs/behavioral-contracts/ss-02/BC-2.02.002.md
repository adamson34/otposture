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

# Behavioral Contract BC-2.02.002: Atomic Store Mutations

## Description

Every mutation of any store file follows write-temp → fsync → rename. A crash, kill, or disk-full at any instant leaves the store in its previous valid state (FM-003). This is the foundational durability contract for plant workstations that lose power.

## Preconditions

1. A mutating operation (answer, snapshot, override, migration) has produced new file content.

## Postconditions

1. On success: the target file contains exactly the new content.
2. On any failure (ENOSPC, kill, power loss simulated): the target file contains exactly the prior content; temp files are ignorable/cleanable artifacts.

## Invariants

1. No code path writes a store file in place or truncates before rewrite.
2. Rename happens only after fsync of the temp file (and directory fsync where the platform honors it).
3. One logical operation = one atomic commit point; multi-file operations order writes so any prefix is a valid store (append-only files make this tractable).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Disk full during temp write | E-FS-002; original intact; temp removed best-effort |
| EC-002 | Store directory on a folder-sync tool (Dropbox/OneDrive) producing transient locks on rename | Bounded retry with backoff, then E-FS-003 naming the locked file; original intact |
| EC-003 | Orphan temp files from a previous crash | Detected on next load; reported and cleaned; never interpreted as store content |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Record answer; verify file replaced atomically | New content present; no intermediate observable state | happy-path |
| Inject failure between temp-write and rename | Store loads as pre-operation state | error |
| Fill disk; attempt snapshot | E-FS-002; prior history fully intact | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-012 | Crash-consistency: for any single injected fault point, store loads to a valid state | proptest (fault injection harness) |
| VP-013 | No syscall sequence truncates an existing store file before its replacement is durable | manual code audit + lint |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-002, CAP-005 |
| L2 Failure Modes | FM-003 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-2.02.003 — composes with (snapshot appends use this discipline)
- BC-3.03.002 — composes with (per-answer persistence, FM-007)
