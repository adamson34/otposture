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
subsystem: "SS-06"
capability: "CAP-007"
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

# Behavioral Contract BC-6.06.001: Command Surface and Exit Codes

## Description

The v1 command surface is fixed and small: `init`, `assess`, `answer`, `override`, `snapshot`, `status`, `diff`, `report`, `verify`, `migrate`, plus `--help`/`--version`. Exit codes are stable, documented contract (scripts and CI depend on them).

## Preconditions

1. Any invocation of the binary.

## Postconditions

1. Exit codes: `0` success (including "no change" outcomes); `1` internal/unexpected error; `2` CLI usage error (unknown command/flag, parse failure); `3` input validation error (E-VAL-*); `4` store integrity/state error (E-STORE-*); `5` lock contention (E-LOCK-*); `6` catalog error (E-CAT-*); `7` comparability refusal (E-SCORE-002); `8` filesystem/environment error (E-FS-*, E-RPT-*).
2. `--help` for every command shows synopsis, flags with types/defaults, and one example; `--version` prints semver + catalog id/version bundled + engine version.
3. Unknown command suggests the nearest valid one.

## Invariants

1. Exit code classes map 1:1 to error-taxonomy categories (prd-supplements/error-taxonomy.md) — adding an error never reuses a foreign class.
2. Warnings never change a success exit code (severity `degraded` ⇒ exit 0 + stderr warning).
3. Diagnostics go to stderr; data/results to stdout (pipeable).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | `otposture statu` | Exit 2; "did you mean `status`?" |
| EC-002 | Flag valid for another command (`init --force`) | Exit 2; explains the flag isn't valid here (and that init never forces) |
| EC-003 | Multiple errors in one invocation (usage + would-be lock) | First-failing-phase wins: usage errors detected before any store access |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| `status` on healthy store | Exit 0, report on stdout | happy-path |
| `diff` across unbridged break | Exit 7, E-SCORE-002 on stderr | error |
| `frobnicate` | Exit 2 with suggestion | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-080 | Every E-code in the taxonomy maps to exactly one exit class; integration tests cover each class | CI matrix |
| VP-081 | stdout/stderr separation holds for all commands | snapshot tests |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-007 (and all command-backed CAPs) |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-6.06.002 — composes with (error rendering)
- All command BCs — depends on (this is their entry surface)
