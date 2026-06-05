---
document_type: module-criticality
level: ops
version: "1.1"
status: draft
producer: product-owner
timestamp: 2026-06-05T12:50:04
phase: 1b
inputs: [prd.md]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: prd.md
---

# Module Criticality Classification: otposture

## Tier Definitions

| Tier | Mutation Kill Rate Target | Description |
|------|--------------------------|-------------|
| **CRITICAL** | ≥ 95% | Wrong numbers in exec-facing reports, or lost/corrupted posture history |
| **HIGH** | ≥ 90% | Wrong content/credibility in user-facing surfaces |
| **MEDIUM** | ≥ 80% | Supporting functionality |
| **LOW** | ≥ 70% | Glue and boilerplate |

## Module Inventory

- **scoring-core** — pure crate: percent, sub-scores, bands, floor rule, deltas/drivers, comparability, ranking (SS-04)
- **posture-store** — persistence: atomic writes, append-only snapshots, integrity, locks, migration (SS-02)
- **catalog** — parse/validate/digest catalogs; bundled CPG 2.0 content (SS-01)
- **assessment** — answer recording, guided session, overrides (SS-03)
- **reporting** — status/diff rendering, exec HTML generation (SS-05)
- **cli** — command parsing, exit codes, error rendering, JSON projection (SS-06)

## Module Classification

| Module | Subsystem | Tier | Rationale | Kill Rate Target | VP Count |
|--------|-----------|------|-----------|-----------------|----------|
| scoring-core | SS-04 | CRITICAL | A scoring defect ships wrong numbers to a COO — the worst failure for a credibility product (R-005); formal-verification target | ≥ 95% | 20 (VP-040..059) |
| posture-store | SS-02 | CRITICAL | History loss/corruption destroys months of un-recreatable posture data (R-006); honesty guarantees live here (DI-002) | ≥ 95% | 15 (VP-010..024) |
| catalog | SS-01 | HIGH | Wrong framework content misleads real decisions; gateway validation protects everything downstream | ≥ 90% | 8 (VP-001..008) |
| assessment | SS-03 | HIGH | Data-entry integrity and override auditability (DI-006, DI-011) | ≥ 90% | 7 (VP-030..036) |
| reporting | SS-05 | HIGH | Exec-facing artifact; disclosure rules and injection safety (NFR-007) | ≥ 90% | 7 (VP-070..076) |
| cli | SS-06 | MEDIUM | Thin shell; contracts mostly delegated — but exit codes and offline guarantee are CI-enforced | ≥ 80% | 7 (VP-080..086) |

## Per-Module Risk Assessment

| Module | Tier | Blast Radius | Security Sensitivity | Implementation Complexity | Test Priority |
|--------|------|-------------|---------------------|--------------------------|--------------|
| scoring-core | CRITICAL | high (every surface) | low | medium (pure math, subtle rounding) | P0 |
| posture-store | CRITICAL | high (all data) | medium (integrity) | high (atomicity, crash consistency) | P0 |
| catalog | HIGH | high (inputs to everything) | medium (licensing, digests) | low | P0 |
| assessment | HIGH | medium | medium (DI-013 entry point) | medium (TTY interaction) | P1 |
| reporting | HIGH | medium (exec artifact) | high (escaping, disclosure) | medium | P1 |
| cli | MEDIUM | low | low | low | P1 |

## Classification Summary

| Tier | Module Count | Share |
|------|-------------|-------|
| CRITICAL | 2 | 33% |
| HIGH | 3 | 50% |
| MEDIUM | 1 | 17% |
| LOW | 0 | 0% |

> Holdout-scenario density and review depth follow tier: scoring-core and posture-store receive Kani proofs, fuzzing, and the densest holdout coverage (per NFR-008).
