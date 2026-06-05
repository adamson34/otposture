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

# Behavioral Contract BC-2.02.008: Deterministic Human-Readable Serialization

## Description

Store files are human-readable text with fully deterministic serialization: stable key order, stable formatting, one semantic change ⇒ one minimal textual diff. This is what makes the store git-diffable and hand-repairable — a first-class differentiator, not a nicety.

## Preconditions

1. Any store write is occurring (BC-2.02.002 provides the durability; this contract governs the bytes).

## Postconditions

1. Serializing the same logical state always yields identical bytes (key order, whitespace, number formatting, line endings all canonical).
2. Changing one Answer produces a textual diff touching only that answer's lines (plus any single summary line).
3. Files parse with standard tooling for the chosen format and contain explanatory header comments (schema version, "do not edit while a session is active", non-PII note per DI-013).

## Invariants

1. No timestamps, ordering, or representation depend on map-iteration order, locale, or platform (LF line endings everywhere).
2. Numbers serialize with fixed precision rules (no float jitter between writes).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Note text containing the format's special characters (quotes, colons, leading dashes) | Escaped correctly; round-trips byte-stable |
| EC-002 | Same store written on Windows and Linux | Byte-identical output (LF, no platform paths inside content) |
| EC-003 | Very long note (multi-KB) | Serialized in the format's block style keeping per-line diffs readable |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Write state S twice | Byte-identical files | happy-path |
| Change one answer level; diff files | Diff confined to that answer's block | happy-path |
| Round-trip 1000 random valid states | parse(serialize(s)) == s, and serialize is stable | edge-case |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-022 | serialize is deterministic: s == s' ⇒ bytes == bytes' | proptest |
| VP-023 | Round-trip identity: parse(serialize(s)) == s | proptest |
| VP-024 | Single-field change yields a diff bounded to that field's block | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-002, CAP-005 |
| L2 Domain Invariants | DI-013 |
| Differentiator | "Git-friendly versioned posture history" (differentiators.md) |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-2.02.004 — composes with (parser side of round-trip)
- BC-1.01.003 — related to (same canonicalization idea for catalog digests)
