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
subsystem: "SS-05"
capability: "CAP-008"
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

# Behavioral Contract BC-5.05.004: Report Output Path Safety

## Description

Report generation never silently overwrites and never half-writes (FM-008). Pre-flight checks the output path; overwrite requires `--force`; the report file itself is written atomically.

## Preconditions

1. Report content rendered successfully (BC-5.05.003); output path supplied or defaulted (`./otposture-report-<date>-s<seq>.html`).

## Postconditions

1. Path free + parent writable: file written via temp+rename (BC-2.02.002 discipline applied to the output).
2. Path exists without `--force`: E-RPT-001 naming the path; nothing written.
3. Path exists with `--force`: replaced atomically.
4. Default filename is collision-resistant (date + snapshot seq) and contains no PII (DI-013).

## Invariants

1. Output writes never touch any store file (separate directory trees by default; refuses to write the report inside the store directory).
2. Failure at any point leaves either the old file intact or the new file complete — never a truncated report (an exec might mail a half-rendered file without noticing).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Output path is a directory | E-RPT-002 suggesting a filename inside it |
| EC-002 | Output on a full disk | E-FS-002; pre-existing file untouched |
| EC-003 | Output path inside the Posture Store directory | E-RPT-003 refusal (keeps store diffs clean of generated artifacts) |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| `report --exec` in empty dir | Default-named file created | happy-path |
| Re-run same command | E-RPT-001 (same default name, same day/seq) | error |
| Re-run with `--force` | Replaced atomically | happy-path |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-076 | Output file is always either absent, previous-complete, or new-complete (fault injection) | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-008 |
| L2 Failure Modes | FM-008 |
| Architecture Module | op-cli (module-decomposition.md) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-5.05.003 — depends on (content)
- BC-2.02.002 — related to (same atomic-write discipline)
