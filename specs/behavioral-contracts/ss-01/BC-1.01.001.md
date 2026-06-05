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

# Behavioral Contract BC-1.01.001: Parse Well-Formed Framework Catalog

## Description

A syntactically and structurally valid catalog file loads into an in-memory catalog exposing all fields needed by assessment, scoring, and reporting. Parsing is lossless: every goal, function, weight, tier default, rating, band threshold, and provenance field in the file is available after load.

## Preconditions

1. A catalog file exists at the resolved path (bundled CPG 2.0 or user-supplied via `--catalog`).
2. The file is valid UTF-8 in the documented catalog schema version.

## Postconditions

1. Returns a Catalog exposing: `catalog_id`, `version` (semver), `framework_name`, `licensing_provenance`, ordered Functions, and Goals with `goal_id`, `title`, `function_id`, `weight`, `default_tier`, `impact_meta`, `effort_meta`, `outcome_text`.
2. Goal count and Function count equal the counts in the file; no goal is dropped, reordered across functions, or synthesized.
3. The ordered band list is exposed: N ≥ 2 named bands with exactly N−1 interior thresholds as ordered numeric boundaries (bundled catalog: 4 bands, 3 thresholds).

## Invariants

1. Parsing performs no network access (DI-007) and no writes.
2. The loaded catalog is immutable for the process lifetime (entities.md: FrameworkCatalog aggregate).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Catalog file with unknown extra fields (forward compatibility) | Unknown fields ignored with a debug-level note; load succeeds |
| EC-002 | Catalog schema version newer than the binary supports | Load refused with E-CAT-002 naming both versions |
| EC-003 | Goal with empty optional metadata (no OT note) | Loads; optional fields default to absent, never to empty-string sentinels |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Bundled CPG 2.0 catalog file | Catalog with 34 goals across 6 functions (5/5/19/2/2/1) | happy-path |
| Minimal valid catalog: 1 function, 1 goal | Catalog loads; all required fields present | edge-case |
| File is valid YAML but missing `goals` key | E-CAT-001 structural error; no partial catalog returned | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-001 | Round-trip: serialize(parse(file)) preserves all semantic content | proptest |
| VP-002 | Parser never panics on arbitrary byte input | fuzz |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-001 |
| L2 Domain Invariants | DI-007, DI-008 |
| Architecture Module | op-catalog (module-decomposition.md) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-1.01.002 — composes with (validation runs on the parsed structure)
- BC-1.01.004 — depends on (bundled content loads through this parser)
