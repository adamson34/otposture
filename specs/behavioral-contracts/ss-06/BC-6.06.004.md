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

# Behavioral Contract BC-6.06.004: JSON Output Mode

## Description

`status`, `diff`, and `verify` accept `--json`, emitting a stable, versioned JSON document on stdout for scripting (CI gates, dashboards the user builds themselves). The machine twin of the human output — same single scoring source, never a separate computation.

## Preconditions

1. `--json` passed to a supporting command.

## Postconditions

1. stdout is exactly one JSON document (no banners/spinners); diagnostics still on stderr; exit codes unchanged (BC-6.06.001).
2. Document carries `schema_version` (semver; additive changes minor, breaking major) plus the full data of the human view. For `status`/`diff`: score (band, uncapped band, percent full-precision, floor state, gaps), sub-scores, unknown count, deltas with full-precision driver points, overrides, series breaks. For `verify`: a distinct check-results document — snapshots checked, score mismatches (seq, expected, actual), quarantined orphans, integrity errors, and an overall `ok` boolean (shape outlined in interface-definitions.md).
3. Numbers are emitted at full precision; display rounding is the consumer's choice.
4. No personal data fields exist in the schema (DI-013).

## Invariants

1. JSON and human outputs are projections of the same computed structures — a divergence is a defect (R-005 class).
2. Field names are stable; removal/rename requires a major schema_version bump.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | `--json` on an error (e.g., comparability refusal) | JSON error object on stdout (`{"error": {"code": "E-SCORE-002", ...}}`), same exit code |
| EC-002 | Free-text fields (notes excerpts, rationales) in JSON | Included verbatim where the human view shows them; consumers warned in schema docs they're free text |
| EC-003 | NaN-adjacent cases (undefined percent, all N/A) | `"percent": null` with `"scoreable": false` — never NaN in JSON |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| `status --json` | Valid JSON matching published schema | happy-path |
| `diff 1 3 --json` | Drivers with full-precision points summing to delta | happy-path |
| `diff` across break `--json` | JSON error object, exit 7 | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-085 | All JSON outputs validate against the published JSON Schema | CI schema validation |
| VP-086 | JSON values equal the human view's underlying values (shared-source check) | golden tests |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-007, CAP-006 |
| L2 Domain Invariants | DI-010, DI-013 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-5.05.001/002 — related to (human twins)
- BC-6.06.001 — composes with (exit semantics)
