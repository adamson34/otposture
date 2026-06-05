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
subsystem: "SS-04"
capability: "CAP-004"
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

# Behavioral Contract BC-4.04.005: Scoring Determinism and Purity

## Description

The entire scoring computation (percent, sub-scores, band, floor, gaps) is one pure, deterministic function of (frozen answers, effective tiers, catalog) — DI-010. Same inputs anywhere, any time, any machine ⇒ identical Score. This is what makes cached snapshot scores verifiable and cross-machine results reproducible.

## Preconditions

1. Validated catalog; complete effective-tier resolution; answer set.

## Postconditions

1. The Score structure is fully populated in one call; repeated calls with equal inputs return structurally identical results.
2. The function signature admits no I/O handles, clocks, RNGs, environment, or global state (compiler-enforced in the pure core crate/module).

## Invariants

1. No floating-point nondeterminism: arithmetic uses fixed-point/rational representation so results are bit-stable across platforms and optimization levels.
2. The engine version is recorded in snapshots so future engine changes that alter results (bugfixes) are distinguishable from data corruption during `verify` (BC-2.02.004 E-STORE-003).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Same store scored on Windows x86_64 and Linux aarch64 | Identical Score values |
| EC-002 | Scoring bugfix changes results for some inputs | New engine version constant; `verify` distinguishes "engine drift" (informational, lists affected snapshots) from corruption |
| EC-003 | Concurrent scoring calls (parallel report + status) | Trivially safe — pure function, no shared state |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Golden store fixture scored 1000× | 1000 identical Scores | happy-path |
| Cross-compile matrix (CI): same fixture | Identical Score bytes per platform | happy-path |
| Score with answers map in different insertion orders | Identical Scores (VP-043 extension) | edge-case |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-051 | Purity: scoring core has no I/O imports (enforced by crate boundary + lint) | manual/CI lint |
| VP-052 | Determinism across platforms on golden fixtures | CI matrix golden tests |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-004 |
| L2 Domain Invariants | DI-010 |
| L2 Risks | R-005 |
| Architecture Module | (filled by architect — the pure core crate) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-4.04.001..004 — composes with (the functions this purity governs)
- BC-2.02.004 — composes with (verify recomputation)
