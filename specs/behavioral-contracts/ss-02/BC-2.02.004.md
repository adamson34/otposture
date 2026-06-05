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

# Behavioral Contract BC-2.02.004: Load-Time Validation and Integrity Verification

## Description

Every store load validates schema and verifies integrity. Corruption is detected, localized, and reported with the maximum salvageable prefix made available read-only (FM-001) — months of posture history must survive a damaged tail record.

## Preconditions

1. A store marker exists at the target path.

## Postconditions

1. Valid store: fully loaded, all records schema-checked, each snapshot's cached Score spot-verified by recomputation (full verification via `verify` command).
2. Corrupted store: E-STORE-002 names the first invalid record (file, record seq/line); all records before it load in read-only mode; mutating commands are refused until repaired.
3. Hand-edited-but-valid store (it's human-readable — edits are legitimate): loads normally; `verify` flags any snapshot whose cached Score no longer matches recomputation as E-STORE-003 (tamper/engine-drift indicator, DI-010).

## Invariants

1. Load performs no writes (except optional orphan-temp cleanup, which is announced).
2. No partial/corrupt record is ever silently skipped — salvage is explicit, never quiet.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Truncated final snapshot record (power loss before BC-2.02.002 commit — shouldn't happen, but disks lie) | Tail truncation detected; prefix loads; message distinguishes "incomplete tail" from "corrupted middle" |
| EC-002 | Store schema version newer than binary | E-STORE-004 "created by a newer otposture"; no load attempt |
| EC-003 | Store schema version older than binary | Transparent in-memory upgrade; on-disk migration only with explicit `store upgrade` command |
| EC-004 | Unknown goal_id in an Answer (catalog/store drift) | Load succeeds with warning; scoring surfaces it per DEC-011/DEC-012 rules |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Load store with 10 valid snapshots | All loaded; spot-check passes | happy-path |
| Corrupt byte in snapshot 7 of 10 | Snapshots 1–6 readable read-only; E-STORE-002 names record 7 | error |
| Hand-edit an answer level in current Assessment | Loads; next score reflects edit (legitimate); `verify` clean | edge-case |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-016 | For any single-byte corruption, load either succeeds correctly or reports the damaged record — never loads wrong data as valid | fuzz |
| VP-017 | verify() recomputation matches cached Scores for stores written by the same engine version | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-002 |
| L2 Domain Invariants | DI-002, DI-010 |
| L2 Failure Modes | FM-001 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-2.02.003 — composes with (validates snapshot records)
- BC-2.02.008 — depends on (parses the deterministic format)
