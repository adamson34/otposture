---
document_type: domain-spec-section
level: L2
section: "assumptions"
version: "1.0"
status: draft
producer: business-analyst
timestamp: 2026-06-05T12:10:44
phase: 1a
inputs: [product-brief.md, planning/research-report.md]
input-hash: "d0490fef4b5ce75c2b7fc3c0e135aeb5"
traces_to: L2-INDEX.md
---

# Assumptions Requiring Validation

> **Sharded L2 section (DF-021).** Navigate via `L2-INDEX.md`.

| ID | Assumption | Confidence | Validation Method | Impact if Wrong | Status | Traced To |
|----|------------|------------|-------------------|-----------------|--------|-----------|
| ASM-001 | CPG 2.0 content is US-government public domain with no contractor copyright notice, so goal text can be embedded in the catalog. | High | Read the copyright page of the CPG 2.0 PDF and Matrix spreadsheet before catalog authoring. | HIGH — catalog must be rewritten to paraphrase/ID-only; release blocked until resolved. | **validated** (2026-06-05: CPG 2.0 PDF is TLP:CLEAR, "may be distributed without restriction," CISA-authored US gov work, no contractor notice) | CAP-001, DI-009, R-004 |
| ASM-002 | CPG 2.0's goal structure (six CSF functions, ~tens of goals, per-goal cost/impact/complexity ratings) maps cleanly onto ordinal answers + weighted Functions. | High | Author the catalog from the official PDF + Matrix; confirm every goal is assessable as a single ordinal question (split compound goals if not). | MEDIUM — catalog modeling rework; possible per-goal sub-questions. | **validated** (2026-06-05: 34 goals — GOVERN 5, IDENTIFY 5, PROTECT 19, DETECT 2, RESPOND 2, RECOVER 1; per-goal Outcome/Action/Risk/Scope/OT-notes; Cost/Impact/Ease ratings with published definitions) | CAP-001, CAP-003 |
| ASM-003 | The primary persona (dual-hatted plant OT engineer) is CLI-comfortable; the HTML report covers non-technical stakeholders. | Medium | Author is the persona (initial validation); confirm with ≥5 external early users post-release. | HIGH — would force the local-web-UI form factor; significant scope change. | unvalidated | CAP-003, CAP-008 |
| ASM-004 | A returning user can update answers and regenerate the exec report in <15 minutes per session. | Medium | Dogfood timing during author's real monthly updates; instrument nothing (offline) — manual timing. | MEDIUM — undermines the "ticker" thesis; would push toward smarter question filtering (only-changed prompts). | unvalidated | CAP-003, CAP-008 |
| ASM-005 | CPG 2.0's published cost/impact/complexity metadata is sufficient to seed default Criticality Tiers and Next-Best-Action impact/effort ratings. | Medium | Inspect the Complete CPGs Matrix; where absent, author cited editorial defaults (research notes CPG guidance is IT-leaning — OT adjustment may be needed). | MEDIUM — tier defaults need independent rationale; ranking quality degrades. | unvalidated | CAP-009, CAP-010, DI-003 |
| ASM-006 | Four named bands (Foundational/Developing/Managed/Resilient) read credibly to executives and are not confused with C2M2 MILs. | Medium | Feedback from first exec-report consumers; check no collision with terms already loaded in the user's org. | LOW — rename bands; thresholds unchanged (names are catalog data, DI-008). | unvalidated | CAP-004, CAP-008 |
| ASM-007 | Plant environments can run a single static Rust binary on Windows/Linux workstations without installer/admin friction. | High | Test on a stock Windows 10/11 engineering-workstation profile without admin rights. | MEDIUM — would need portable-app packaging attention, still no architecture change. | unvalidated | (constraint, brief) |
| ASM-008 | Site-level single-writer usage (one person updates posture) is the dominant pattern; concurrent multi-user editing is not needed in v1. | High | Confirm via early-user feedback; matches the dual-hatted persona reality. | LOW — lockfile + clear error covers the rare collision (FM-006). | unvalidated | FM-006 |

> ASM-001 and ASM-002 are **pre-implementation gates**: both are resolved by reading two documents and should be closed during PRD creation, before any catalog authoring.
