---
document_type: domain-spec-section
level: L2
section: "entities"
version: "1.0"
status: draft
producer: business-analyst
timestamp: 2026-06-05T12:10:44
phase: 1a
inputs: [product-brief.md, planning/research-report.md]
input-hash: "d0490fef4b5ce75c2b7fc3c0e135aeb5"
traces_to: L2-INDEX.md
---

# Domain Entities

> **Sharded L2 section (DF-021).** Navigate via `L2-INDEX.md`.

## Entity Model

| Entity | Description | Key Attributes | Invariants |
|--------|-------------|---------------|------------|
| **FrameworkCatalog** | Immutable, versioned framework encoding (CPG 2.0 in v1). | `catalog_id`, `version` (semver), `framework_name`, `licensing_provenance`, `band_thresholds[4]`, Goals[], Functions[] | Immutable once published; provenance mandatory (DI-009); thresholds and weights live here, not in code (DI-008). |
| **Goal** | One assessable framework item. | `goal_id` (catalog-scoped, e.g. CPG ref), `title`, `function_id`, `weight` (>0), `default_tier` (Critical/High/Standard), `impact_meta`, `effort_meta`, `text_provenance` | `goal_id` unique within catalog (DEC-007); tier default carries citation/rationale in catalog. |
| **Function** | Goal grouping for sub-scores (CSF six functions for CPG 2.0). | `function_id`, `name`, `order` | Every Goal references exactly one Function. |
| **Site** | The single facility this store describes. | `name`, `sector`, `created_at`, free-form metadata | Exactly one Site per Posture Store (v1). |
| **Assessment** | Current working answer set, pinned to a catalog version. | `catalog_id`, `catalog_version`, Answers{goal_id → Answer} | Mutable; goals without an Answer are `Unknown` (DI-011); references only Goals present in the pinned catalog. |
| **Answer** | One Goal's current Implementation Level. | `level` (NI/PI/LI/FI/NA/Unknown), `note?`, `answered_at` | `answered_at` required for explicit answers; reserved fields for evidence refs (CAP-012). |
| **TierOverride** | Site-local tier reclassification. | `goal_id`, `tier`, `rationale` (non-empty), `created_at` | Rationale mandatory (DI-006); at most one active override per Goal; surfaced in reports. |
| **Snapshot** | Immutable point-in-time capture. | `seq` (monotonic int), `taken_at`, `catalog_id`+`version`, frozen Answers, computed Score, `label?` | Append-only, never mutated (DI-002); `seq` is the authoritative order, not `taken_at` (DEC-006/FM-005). |
| **Score** | Derived scoring result stored within a Snapshot. | `band`, `percent`, per-Function sub-scores, `floor_rule_triggered`, `critical_gaps[]` | Fully derivable from frozen Answers + catalog (DI-010); stored copy is a cache, recomputation must match. |
| **Delta** | Comparison of two Snapshots (computed, not stored). | `from_seq`, `to_seq`, `percent_delta`, `band_change?`, Drivers[] | Both Snapshots must share catalog version or be back-cast (DI-001); Driver contributions sum to `percent_delta` (DI-005). |
| **Driver** | One Goal's contribution to a Delta. | `goal_id`, `from_level`, `to_level`, `points` (signed) | Negative drivers reported honestly (DEC-009). |
| **SeriesBreak** | History marker for non-comparable catalog transition. | `at_seq`, `from_version`, `to_version`, `reason` | Diffs spanning a break are refused (DEC-003). |

## Aggregates & Boundaries

- **Posture Store aggregate** (consistency boundary): Site + Assessment + Snapshots + TierOverrides + SeriesBreaks. All writes are local, atomic per operation (FM-003), single-writer (FM-006).
- **FrameworkCatalog aggregate**: read-only input, distributed with the tool or supplied as a file. Never modified by assessment operations — site-local customization happens only via TierOverride in the Posture Store.
- **Projections** (no stored state): Score (cached in Snapshot but recomputable), Delta, Status view, Exec Report.

## Relationships

```
FrameworkCatalog 1──* Goal *──1 Function
Site 1──1 Assessment 1──* Answer (per Goal)
Site 1──* Snapshot (ordered by seq) ── each pins FrameworkCatalog version
Site 1──* TierOverride (≤1 active per Goal)
Snapshot ↔ Snapshot ⇒ Delta 1──* Driver
History 1──* SeriesBreak (between Snapshots)
```
