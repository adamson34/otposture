---
document_type: domain-spec-section
level: L2
section: "events"
version: "1.0"
status: draft
producer: business-analyst
timestamp: 2026-06-05T12:10:44
phase: 1a
inputs: [product-brief.md, planning/research-report.md]
input-hash: "d0490fef4b5ce75c2b7fc3c0e135aeb5"
traces_to: L2-INDEX.md
---

# Domain Events / Processing Stages

> **Sharded L2 section (DF-021).** Navigate via `L2-INDEX.md`.
> otposture is a local CLI — "events" are domain state transitions, not a message bus.

| Event/Stage | Trigger | Preconditions | Outcomes |
|-------------|---------|---------------|----------|
| **CatalogLoaded** | Any command needing scoring/assessment | Catalog file exists, passes structural validation (CAP-001) | Catalog available in memory; validation failures abort with named defects (DEC-007, FM-002) |
| **StoreInitialized** | `init` command | Target directory empty of an existing store; catalog resolvable | Posture Store created: Site record, empty Assessment pinned to catalog version (CAP-002) |
| **AnswerRecorded** | User answers a Goal in guided assessment or direct set | Store initialized; Goal exists in pinned catalog; level is a valid Implementation Level | Assessment updated; `answered_at` stamped; store written atomically (FM-003) |
| **SnapshotTaken** | `snapshot` command (or implied by report/diff on dirty state — PRD decision) | Assessment exists; scoring succeeds (DI-010) | New immutable Snapshot appended with `seq = max+1`, frozen Answers, cached Score (CAP-005); zero-answer snapshot warns (DEC-004) |
| **DeltaComputed** | `diff`/`status`/`report` comparing two Snapshots | Both Snapshots share catalog version or back-cast values exist (DI-001) | Delta with reconciled Drivers (DI-005); refused with Series Break explanation otherwise (DEC-003) |
| **TierOverridden** | `override` command | Goal exists; non-empty rationale supplied (DI-006) | Active override recorded; effective tier changes; floor-rule re-evaluation on next scoring; disclosure in all subsequent reports (DEC-005) |
| **CatalogMigrated** | `migrate` command after catalog upgrade | New catalog valid; goal-mapping table available (or absent) | Either: back-cast values stored alongside originals for mappable history, or SeriesBreak recorded (CAP-011, DI-002) |
| **ReportGenerated** | `report --exec` | At least one Snapshot exists | Static HTML written to user-chosen path; pure projection — no store mutation (CAP-008, FM-004) |

## Ordering Constraints

1. `StoreInitialized` precedes all other store events.
2. `SnapshotTaken` events are strictly ordered by `seq`; `taken_at` timestamps are informational only (DEC-006).
3. `DeltaComputed` requires ≥2 Snapshots; the first Snapshot is the baseline (DEC-001).
4. `TierOverridden` affects only Scores computed after it; existing Snapshots' cached Scores are never retroactively changed (DI-002) — a band that was honestly uncapped last month stays as history.
5. `CatalogMigrated` is the only event that may add derived (back-cast) data to existing history, and it never overwrites originals.

## Pipeline View (machine consumers)

```
catalog.yaml ─→ [CAP-001 validate] ─→ Catalog
user input  ─→ [CAP-003 assess]  ─→ Assessment ─→ [CAP-004 score (pure)] ─→ Score
                                      │
                                      └─→ [CAP-005 snapshot] ─→ History ─→ [CAP-006 diff] ─→ Delta
                                                                   │
                                                                   └─→ [CAP-008 report] ─→ exec.html
```
