---
document_type: domain-spec-section
level: L2
section: "capabilities"
version: "1.0"
status: draft
producer: business-analyst
timestamp: 2026-06-05T12:10:44
phase: 1a
inputs: [product-brief.md, planning/research-report.md]
input-hash: "d0490fef4b5ce75c2b7fc3c0e135aeb5"
traces_to: L2-INDEX.md
---

# Domain Capabilities

> **Sharded L2 section (DF-021).** Navigate via `L2-INDEX.md`.

| ID | Capability | Description | Business Rule | Priority |
|----|-----------|-------------|---------------|----------|
| CAP-001 | Load & validate framework catalog | Parse a versioned catalog file (CPG 2.0 in v1, OSCAL-shaped); reject structurally invalid catalogs before any scoring. | Duplicate Goal IDs, missing weights/tiers/thresholds, or absent licensing provenance ⇒ load refused (DI-009, DEC-007). | P0 |
| CAP-002 | Initialize posture store | Create the local data root for one Site, pinned to a catalog version. | One Site per store; store is human-readable and git-friendly (DI-007). | P0 |
| CAP-003 | Guided incremental assessment | Prompt-driven answering of Goals (ordinal Implementation Levels), resumable in minutes-scale sessions; any subset answerable in any order. | Unanswered Goals are `Unknown`; assessment is never "blocked on completeness" (DI-011). | P0 |
| CAP-004 | Compute score | Derive Band, %, per-Function sub-scores, and floor-rule status from Answers + catalog. Pure, deterministic computation. | Weighted formula with N/A exclusion (DI-004, DI-010); floor rule caps Band (DI-003). | P0 |
| CAP-005 | Take snapshot | Freeze current Assessment + Score as an immutable, sequence-numbered history entry. | Append-only; never rewritten (DI-002); pins catalog version (DI-001). | P0 |
| CAP-006 | Diff with driver attribution | Compare two Snapshots: signed score delta plus per-Goal Drivers that sum to the delta. | Cross-catalog-version diffs refused unless back-cast (DI-001, DEC-003); driver sum reconciles (DI-005). | P0 |
| CAP-007 | Status view | CLI summary of current Band/%, trend vs. baseline or chosen Snapshot, biggest movers, open critical gaps. | First snapshot shows "baseline established," no delta (DEC-001). | P0 |
| CAP-008 | Generate exec report | Render the one-page static HTML (print-to-PDF) executive summary from a Snapshot (+ optional comparison Snapshot). | Report is a pure projection; rendering failure never corrupts data (FM-004). Tier Overrides in effect are disclosed (DEC-005). | P0 |
| CAP-009 | Next best actions | Rank unimproved Goals by potential score impact ÷ catalog effort metadata; surface top N in status and report. | Static catalog metadata in v1; no user-supplied estimates. | P1 |
| CAP-010 | Tier override | Site-local reclassification of a Goal's Criticality Tier with mandatory rationale, recorded in the store. | Non-empty rationale required; overrides surfaced in all reports (DI-006). | P1 |
| CAP-011 | Catalog version migration | On catalog upgrade: back-cast history where Goals map, or record an explicit Series Break. | Never silently rescore history (DI-002); originals retained alongside back-cast values. | P1 |
| CAP-012 | Evidence & staleness (reserved) | Post-MVP: attach evidence artifacts to Answers; per-Goal freshness windows decay credit. | Data formats must reserve room (forward-compatible schema) without implementing behavior. | P2 |
| CAP-013 | otsniff import (reserved) | Post-MVP: ingest otsniff pcap findings as evidence pre-filling network-related Goals. | Depends on CAP-012. | P2 |

> Pipeline view: CAP-001/002 (setup) → CAP-003 (input) → CAP-004 (pure scoring core) → CAP-005 (persist) → CAP-006/007 (analyze) → CAP-008/009 (project). CAP-004 is the purity boundary — no I/O, fully property-testable.
