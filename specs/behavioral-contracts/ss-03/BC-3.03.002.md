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
subsystem: "SS-03"
capability: "CAP-003"
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

# Behavioral Contract BC-3.03.002: Guided Assessment Session

## Description

`assess` walks the user through Goals interactively — showing goal title, outcome text, and OT note, accepting an ordinal answer or skip. Resumable, order-flexible, and crash-safe: every answer persists as given (FM-007). This is the minutes-scale loop the product thesis depends on (ASM-004).

## Preconditions

1. Store initialized; lock acquired; interactive TTY (non-TTY invocation gets E-VAL-007 directing to `answer` for scripting).

## Postconditions

1. Session presents Goals filtered/ordered by: `--function` (one function), `--only unknown|stale` (default `unknown` for resume behavior), or `--all`; default order is catalog order.
2. Each accepted answer is durably persisted before the next prompt (BC-3.03.001).
3. Session end (finish, quit, or kill) loses at most the single in-flight prompt; a summary shows answered/skipped/remaining counts and current score preview.

## Invariants

1. Skipping never writes anything; quitting is always safe.
2. The session displays the goal's official CPG text (catalog data) — never paraphrases requirements.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | All goals already answered, `--only unknown` | "Nothing to assess" with hint to use `--all` to revisit |
| EC-002 | Terminal killed mid-session (SSH drop) | All previously confirmed answers durable; re-running resumes at remaining unknowns (FM-007) |
| EC-003 | User answers `back` | Returns to previous goal showing its just-recorded answer for correction (correction = new answer, normal replace semantics) |
| EC-004 | Window resize / narrow terminal (80 col) | Goal text wraps readably; no truncated prompts |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Fresh store, `assess`, answer 5 goals, quit | 5 durable answers; summary "5 answered, 29 unknown"; score preview | happy-path |
| Kill -9 after 3rd confirmation | Store has exactly 3 answers | edge-case |
| `assess` with no TTY (piped stdin) | E-VAL-007 pointing to `answer` | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-032 | Confirmed-answer count on disk ≥ confirmed-answer count at any prior instant (monotone durability) | proptest (kill-injection) |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-003 |
| L2 Assumptions | ASM-003, ASM-004 |
| L2 Failure Modes | FM-007 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-3.03.001 — depends on (per-answer recording)
- BC-5.05.001 — related to (score preview reuses status computation)
