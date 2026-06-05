---
document_type: domain-spec-section
level: L2
section: "risks"
version: "1.0"
status: draft
producer: business-analyst
timestamp: 2026-06-05T12:10:44
phase: 1a
inputs: [product-brief.md, planning/research-report.md]
input-hash: "d0490fef4b5ce75c2b7fc3c0e135aeb5"
traces_to: L2-INDEX.md
---

# Risk Register

> **Sharded L2 section (DF-021).** Navigate via `L2-INDEX.md`.

| ID | Risk | Likelihood | Impact | Mitigation | Status | Category | Traced To |
|----|------|-----------|--------|-----------|--------|----------|-----------|
| R-001 | CSET's CPG 2.0 module (Q1 2026) + existing Trend & Compare makes "free CPG assessment with trending" a CISA-official answer, eroding differentiation. | High | Medium | Position as complementary "ticker between assessments" (cadence, git-native history, driver attribution, exec one-pager); consider CSET import/export rather than competing on assessment depth. | open | business | CAP-005, CAP-006, CAP-008 |
| R-002 | Credibility attack: a skeptical CISO dismisses the score as arbitrary ("your % is made up"), poisoning the core value prop. | Medium | High | DI-003 floor rule, DI-005 driver attribution, DI-008 inspectable weights, band-first presentation (DEC-010); document the scoring rationale with framework precedents (C2M2/CSF) in user docs. | open | business | DI-003, DI-005, DI-008, CAP-004 |
| R-003 | CPG maintenance cadence uncertainty — CISA's 2025 staffing/budget cuts could orphan or delay CPG revisions, leaving the default catalog stale. | Medium | Medium | Catalog-as-data with version pinning (DI-001, DI-008) makes framework swaps cheap; community OSCAL catalog publication spreads maintenance; C2M2/CSF pluggability as fallback. | open | business | CAP-001, CAP-011 |
| R-004 | Licensing misstep: embedding framework text that turns out to be restricted (contractor-copyrighted CPG content, ISA/IEC or SANS text) triggers takedown against the flagship open-source artifact. | Low | High | ASM-001 validation gate before catalog authoring; DI-009 provenance required at load; restricted frameworks referenced by ID only; no AI-derived 62443 content ever. | open | business | DI-009, ASM-001 |
| R-005 | A scoring defect ships wrong numbers into exec-facing reports — the worst possible failure for a credibility-builder project. | Medium | High | Pure scoring core (CAP-004) with property tests (determinism DI-010, driver-sum DI-005, floor-rule DI-003), golden-fixture catalogs, mutation testing; Kani proofs on aggregation arithmetic per factory toolchain. | open | reliability | DI-010, CAP-004, FM-001 |
| R-006 | Posture Store corruption on plant workstations (power loss, full disk, sync tools) destroys months of history the persona cannot recreate. | Medium | High | Append-only snapshots (DI-002), atomic temp+rename writes (FM-003), schema validation + integrity check on load (FM-001), human-readable format enables manual recovery; docs recommend git or backup of the store directory. | open | reliability | DI-002, FM-001, FM-003 |
| R-007 | Default Criticality Tiers (editorial) draw community criticism ("who says goal X is Critical?"), undermining the floor rule's authority. | Medium | Medium | Every default tier carries a citation/rationale in the catalog (DI-008); overrides are first-class with recorded rationale (DI-006); invite community review of the tier table as a published, diffable artifact. | open | business | DI-003, DI-006, ASM-005 |
| R-008 | Scope creep toward GRC-platform features (multi-user, workflow, compliance claims) drowns the solo nights-and-weekends developer. | Medium | Medium | Brief's Out-of-Scope list is binding; P2 capabilities (CAP-012/013) are schema-reserved but not built; "ship when ready" with MVP-only releases. | open | business | (brief scope) |
