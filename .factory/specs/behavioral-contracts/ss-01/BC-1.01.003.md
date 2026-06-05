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
subsystem: "SS-01"
capability: "CAP-001"
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

# Behavioral Contract BC-1.01.003: Catalog Identity, Version Exposure, and Immutability

## Description

Every loaded catalog exposes a stable `(catalog_id, version)` identity used for snapshot pinning (DI-001). The identity is content-bound: a content digest detects tampered or repackaged catalogs claiming the same version.

## Preconditions

1. A catalog has passed validation (BC-1.01.002).

## Postconditions

1. `(catalog_id, version)` and a `content_digest` (SHA-256 over canonicalized catalog content) are exposed to consumers.
2. Snapshot creation records all three (BC-2.02.003); mismatch between a stored digest and a loaded catalog of the same `(id, version)` is reported as E-CAT-004 (tampered/divergent catalog) on any operation that needs comparability.

## Invariants

1. The in-memory catalog is never mutated after load; site-local customization occurs only via TierOverride in the Posture Store (entities.md aggregates).
2. Two catalogs with equal digests score identically (with DI-010, gives cross-machine reproducibility).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Same `(id, version)` but different digest (edited copy) | E-CAT-004 warning on load; comparability operations against history refuse until resolved |
| EC-002 | Same content, differing key order / whitespace in file | Equal digests — canonicalization before hashing |
| EC-003 | Catalog renamed on disk (filename ≠ catalog_id) | Identity comes from content, not filename; loads normally |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Load bundled catalog twice | Identical `(id, version, digest)` | happy-path |
| Re-serialize catalog with shuffled key order, reload | Same digest | edge-case |
| One goal title edited, same declared version | Different digest; E-CAT-004 on comparability use | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-005 | Digest is invariant under canonicalization-equivalent re-encodings | proptest |
| VP-006 | Digest differs for any semantic content change | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-001 |
| L2 Domain Invariants | DI-001, DI-010 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-2.02.003 — composes with (snapshots pin this identity)
- BC-4.04.007 — composes with (comparability guard uses digest)
