---
document_type: domain-spec-section
level: L2
section: "edge-cases"
version: "1.0"
status: draft
producer: business-analyst
timestamp: 2026-06-05T12:10:44
phase: 1a
inputs: [product-brief.md, planning/research-report.md]
input-hash: "d0490fef4b5ce75c2b7fc3c0e135aeb5"
traces_to: L2-INDEX.md
---

# Domain-Level Edge Cases

> **Sharded L2 section (DF-021).** Navigate via `L2-INDEX.md`.

| ID | Capability | Edge Case | Expected Behavior |
|----|-----------|-----------|-------------------|
| DEC-001 | CAP-006/007 | Only one Snapshot exists (no comparison point) | Status shows "baseline established `<date>`" with current Band/%; no delta, no trend arrow. Diff command explains a second snapshot is needed. |
| DEC-002 | CAP-004 | Every Goal in a Function is `Not Applicable` | Function sub-score is undefined — displayed as "N/A", excluded from the overall roll-up denominator; never rendered as 0% or 100%. |
| DEC-003 | CAP-006 | Diff requested across a catalog version change with no back-cast | Refused with the Series Break shown: "v1.0→v2.0 at snapshot #12; run `migrate` to back-cast or compare within a version." Never emits a cross-rubric number. |
| DEC-004 | CAP-005 | Snapshot taken with zero explicit Answers (all Unknown) | Permitted with a prominent warning; Score = 0%, Band = Foundational, Unknown count = 100%. Honest starting point, not an error. |
| DEC-005 | CAP-010 | Tier Override demotes a Critical Goal that is currently triggering the floor rule | Permitted (site knows its context), but the override + rationale appear in status and exec report wherever the Band is shown — the cap's removal is never invisible. |
| DEC-006 | CAP-005 | Two Snapshots share a calendar timestamp, or clocks moved backwards between snapshots | `seq` is authoritative for ordering and trend; timestamps are display-only. No behavior depends on monotonic wall-clock time. |
| DEC-007 | CAP-001 | Catalog file has duplicate Goal IDs, a Goal referencing a missing Function, weight ≤ 0, or missing band thresholds | Load refused listing every defect found (not just the first); no partial catalog ever enters scoring. |
| DEC-008 | CAP-004/006 | Rounding: displayed % is rounded but Drivers must sum to the displayed delta (DI-005) | Internal arithmetic at full precision; display rounding to whole percents applied once, at the presentation edge; driver table reconciles to the displayed delta via largest-remainder allocation. |
| DEC-009 | CAP-006 | A Goal's Answer regressed (e.g., Fully → Not Implemented after an outage or staff change) | Negative Driver displayed with the same prominence as positive ones; trend can go down. The tool never suppresses regressions. |
| DEC-010 | CAP-004 | Floor rule newly triggers while raw % rose (critical Goal regressed but many small wins landed) | Headline Band drops/caps and the report leads with the critical gap; the rising % is shown as the sub-metric with explicit explanation. Band and % may legitimately move in opposite directions. |
| DEC-011 | CAP-003 | User answers a Goal ID not present in the pinned catalog version (typo, or goal from a newer catalog) | Rejected with nearest-match suggestion; Assessment never contains orphan Answers. |
| DEC-012 | CAP-011 | Catalog upgrade removes a Goal that has an Answer and a Tier Override | Back-cast drops the Goal from recomputed values with a migration note; the original Answer/Override remain in originals. Orphaned overrides are reported, not silently discarded. |
