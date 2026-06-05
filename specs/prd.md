---
document_type: prd
level: L3
version: "1.0"
status: draft
producer: product-owner
timestamp: 2026-06-05T12:50:04
phase: 1a
inputs: [product-brief.md, domain-spec/L2-INDEX.md]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: domain-spec/L2-INDEX.md
supplements: [interface-definitions.md, error-taxonomy.md, nfr-catalog.md, module-criticality.md]
---

# Product Requirements Document: otposture

> **BC Index Model:** this PRD is an index. Full contracts live one-per-file under
> `behavioral-contracts/ss-NN/`; supplements under `prd-supplements/`. Tables here
> are summaries with pointers — do not inline contract detail.

## 1. Product Overview

### 1.1 Problem Statement

Dual-hatted plant OT engineers cannot measure, track, or communicate security posture at the cadence it changes. Paid point-in-time assessments are stale within weeks; CISA's own CPG worksheet prescribes review "after 12 months"; upward reporting to the C-suite is unpaid manual overhead. Only 9% of ICS/OT security professionals are fully dedicated to security (SANS/OPSWAT 2025).

### 1.2 Solution Vision

A single-binary offline Rust CLI: CISA CPG 2.0 encoded as a validated data catalog; ordinal answers persisted in a human-readable, git-friendly local store; a pure deterministic scoring core producing a floor-ruled maturity band with fully attributed deltas; append-only snapshot history; self-generating one-page executive HTML report.

### 1.3 Key Differentiators

| ID | Differentiator | Description |
|----|---------------|-------------|
| KD-001 | Ticker between assessments | Minutes-scale re-assessment, snapshot/diff cadence vs. annual assessment events (CSET, consultants) |
| KD-002 | Defensible number | Band-first headline; floor rule prevents averaged-away critical gaps; every delta names its drivers |
| KD-003 | Transparent scoring | All weights/thresholds/tiers inspectable catalog data; reproducible by hand vs. proprietary platform scores |
| KD-004 | Self-writing exec one-pager | The C-suite update generates itself from committed snapshots |
| KD-005 | Offline & data-sovereign | Zero network syscalls, ever; no PII by design (DI-007, DI-013) |
| KD-006 | Honest history | Append-only snapshots, explicit series breaks, regressions at full prominence — never silently rescored |

### 1.4 Target Users

| Persona | Description | Volume | Pain Level |
|---------|-------------|--------|-----------|
| Dual-hatted plant OT engineer/security lead | Runs the plant AND owns security reporting; small/mid industrial sites | The norm (91% of ICS/OT pros are part-time on security) | High — overwhelm, no metric, reporting burden |
| OT security consultant | Re-assesses the same clients repeatedly | Secondary | Medium — baseline continuity |

## 2. Requirements by Subsystem

| ID | Subsystem | Description | Criticality | BCs |
|----|-----------|-------------|-------------|-----|
| SS-01 | Framework Catalog | Catalog parse/validate; bundled CPG 2.0 content; identity & digests | HIGH | 4 |
| SS-02 | Posture Store | Local persistence: init, atomic writes, append-only snapshots, integrity, locking, migration/series breaks, deterministic serialization, schema upgrade | CRITICAL | 9 |
| SS-03 | Assessment | Guided + direct answering, Unknown semantics, tier overrides | HIGH | 4 |
| SS-04 | Scoring Engine | Pure core: percent, sub-scores, bands, floor rule, deltas/drivers, comparability, next-best-actions | CRITICAL | 8 |
| SS-05 | Reporting | Status, diff display, exec HTML report, output safety | HIGH | 4 |
| SS-06 | CLI Surface | Commands, exit codes, error rendering, offline guarantee, JSON mode | MEDIUM | 4 |

Full per-BC listing: `behavioral-contracts/BC-INDEX.md` (33 contracts). Section/subsystem numbering is aligned (Section N ⇔ SS-0N).

## 3. Interface Definition

See `prd-supplements/interface-definitions.md` — CLI surface (11 commands), exit-code semantics, JSON schema outline, store/catalog file formats.

## 4. Non-Functional Requirements

See `prd-supplements/nfr-catalog.md` — 10 NFRs. Headlines: zero network syscalls (NFR-001); crash consistency on plant workstations (NFR-002); <15-minute re-assess session (NFR-004); cross-platform bit-identical scoring (NFR-003); no PII anywhere (NFR-006).

## 5. Error Taxonomy

See `prd-supplements/error-taxonomy.md` — categories CAT/STORE/VAL/LOCK/SCORE/FS/RPT mapped 1:1 to exit-code classes 2–8.

## 6. Competitive Differentiator Traceability

| Differentiator | Backing BCs |
|----------------|-------------|
| KD-001 Ticker | BC-3.03.002, BC-2.02.003, BC-4.04.006, BC-5.05.001 |
| KD-002 Defensible number | BC-4.04.004 (floor), BC-4.04.006 (attribution), BC-4.04.003 (band-first) |
| KD-003 Transparent scoring | BC-1.01.002 (rule h), BC-4.04.005, BC-4.04.008, BC-1.01.004 (tier rationales) |
| KD-004 Exec one-pager | BC-5.05.003, BC-5.05.004, BC-4.04.008 |
| KD-005 Offline & sovereign | BC-6.06.003, BC-5.05.003 (no external resources), DI-013 across BC-2.02.001/3.03.001/5.05.003/6.06.004 |
| KD-006 Honest history | BC-2.02.003/006/007, BC-4.04.007, BC-5.05.002 (DEC-009) |

## 7. Module Criticality

See `prd-supplements/module-criticality.md`. CRITICAL: scoring core, posture store. The scoring core is the formal-verification target (Kani on VP-040/041/044/046/047/048/049/050/053/056).

## Cross-Cutting Concerns

1. **Offline guarantee** (BC-6.06.003) and **no-PII** (DI-013) bind every subsystem.
2. **Single scoring source:** every surface (status/diff/report/JSON) projects the same Score/Delta structures (BC-4.04.005); divergence is an R-005-class defect.
3. **Disclosure chain:** floor caps (BC-4.04.004) and tier overrides (BC-3.03.004) must surface in status (BC-5.05.001), diff (BC-5.05.002), and exec report (BC-5.05.003) — no surface may omit them.
4. **Atomicity discipline** (BC-2.02.002) applies to store mutations and report output alike.

## Assumptions

From `domain-spec/assumptions.md`: ASM-001 ✅ validated (CPG 2.0 TLP:CLEAR, 2026-06-05); ASM-002 ✅ validated (34 goals, 6 functions, ratings present); ASM-003/004/006/007/008 open; ASM-005 strengthened (Cost/Impact/Ease ratings confirmed with published logic — tier-default sufficiency still to confirm during catalog authoring).

## Open Questions

1. **Store format choice (YAML vs TOML vs JSON-with-comments)** — deferred to architecture; BC-2.02.008's determinism/diffability constraints are format-agnostic and binding.
2. **Default band thresholds (t1,t2,t3)** — catalog data; proposal (.40/.65/.85) needs a written rationale in the catalog before v1 (R-002).
3. **Default weights** — v1 proposal: uniform weights with criticality expressed via tiers + floor rule rather than weight skew (simpler to defend); confirm during catalog authoring.
4. **Goal-mapping table format for migrations** (BC-2.02.006) — needed only when CPG 2.1+ ships; design with the catalog schema.
5. **`snapshot` auto-take on `report`** when the Assessment is dirty (events.md notes this as a PRD decision) — current decision: **no implicit snapshots**; `report` warns if Assessment differs from the latest snapshot. Revisit after dogfood.
