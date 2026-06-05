---
document_type: domain-spec-index
level: L2
version: "1.1"
status: draft
producer: business-analyst
timestamp: 2026-06-05T12:10:44
phase: 1a
inputs: [product-brief.md, planning/brainstorming-report.md, planning/research-report.md]
input-hash: "d0490fef4b5ce75c2b7fc3c0e135aeb5"
traces_to: product-brief.md
sections:
  - capabilities.md
  - entities.md
  - invariants.md
  - events.md
  - edge-cases.md
  - assumptions.md
  - risks.md
  - failure-modes.md
  - differentiators.md
  - ubiquitous-language.md
---

# L2 Domain Specification: otposture

> **Sharded artifact (DF-021).** This index provides navigation and summary.
> Detail lives in per-section files listed below. Each section targets
> 800-1,200 tokens for optimal LLM consumption.

## Domain Summary

otposture models **continuous OT security posture tracking** for a single industrial site: a versioned framework catalog (CPG 2.0) is answered on an ordinal scale, scored into a floor-ruled maturity band with fully attributed deltas, persisted as an append-only snapshot history, and projected into a self-generating executive report — all offline, deterministic, and reproducible by inspection.

## Document Map

| Section | File | Primary Consumer | Purpose |
|---------|------|-----------------|---------|
| Domain Capabilities | capabilities.md | product-owner, architect, story-writer | CAP-001..013 capability catalog with pipeline view |
| Domain Entities | entities.md | architect, product-owner | Entity model, aggregates (Posture Store / Catalog), relationships |
| Domain Invariants | invariants.md | product-owner, architect | DI-001..013 business rules incl. floor rule, determinism, attribution, no-PII |
| Domain Events | events.md | architect | State transitions, ordering constraints, processing pipeline |
| Edge Cases | edge-cases.md | story-writer, test-writer | DEC-001..012 domain-level edge cases |
| Assumptions | assumptions.md | product-owner, test-writer | ASM-001..008 with validation methods; ASM-001/002 are pre-implementation gates |
| Risks | risks.md | product-owner, architect | R-001..008 risk register |
| Failure Modes | failure-modes.md | architect, test-writer | FM-001..008 plant-workstation runtime failures |
| Differentiators | differentiators.md | product-owner | Differentiator → CAP/DI mapping + threat watch list |
| Ubiquitous Language | ubiquitous-language.md | all agents | Glossary incl. deliberately avoided terms |

> Omitted sections: `bounded-contexts.md` (single-process CLI; aggregate boundaries covered in entities.md), `event-flow.md` (pipeline diagram included in events.md).

## Cross-References

| If you need... | Read these together |
|----------------|-------------------|
| BC creation input | capabilities.md + invariants.md + edge-cases.md + assumptions.md + risks.md + differentiators.md |
| Architecture design input | capabilities.md + entities.md + invariants.md + events.md + risks.md + failure-modes.md |
| Story decomposition input | capabilities.md + edge-cases.md |
| Holdout scenario generation | assumptions.md + risks.md + failure-modes.md |
| NFR derivation | risks.md + failure-modes.md |
| Full domain review (adversary/spec-reviewer) | ALL sections |

## ID Registry Summary

| ID Format | Count | Section |
|-----------|-------|---------|
| CAP-NNN | 13 | capabilities.md |
| DI-NNN | 13 | invariants.md |
| DEC-NNN | 12 | edge-cases.md |
| ASM-NNN | 8 | assumptions.md |
| R-NNN | 8 | risks.md |
| FM-NNN | 8 | failure-modes.md |

## Priority Distribution

| Priority | Count | Items |
|----------|-------|-------|
| P0 (must-have) | 8 | CAP-001, CAP-002, CAP-003, CAP-004, CAP-005, CAP-006, CAP-007, CAP-008 |
| P1 (should-have) | 3 | CAP-009, CAP-010, CAP-011 |
| P2 (nice-to-have, schema-reserved) | 2 | CAP-012, CAP-013 |

## Key Domain Decisions (resolved with the human, 2026-06-05)

1. **History model:** self-contained append-only snapshots; git is a layer on top, not a dependency.
2. **Bands:** four named — Foundational / Developing / Managed / Resilient (thresholds in catalog data).
3. **Criticality tiers:** opinionated cited defaults in the catalog, site-overridable with mandatory rationale.
4. **Next best actions:** static catalog impact/effort metadata in v1; no user-supplied estimates.
