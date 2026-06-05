---
document_type: domain-spec-section
level: L2
section: "ubiquitous-language"
version: "1.0"
status: draft
producer: business-analyst
timestamp: 2026-06-05T12:10:44
phase: 1a
inputs: [product-brief.md, planning/research-report.md]
input-hash: "d0490fef4b5ce75c2b7fc3c0e135aeb5"
traces_to: L2-INDEX.md
---

# Ubiquitous Language

> **Sharded L2 section (DF-021).** Navigate via `L2-INDEX.md`.

| Term | Definition |
|------|-----------|
| **Framework Catalog** | A versioned, machine-readable encoding of a security framework (v1: CISA CPG 2.0): Goals grouped by Function, with weights, tier defaults, impact/effort metadata, band thresholds, and licensing provenance. Data, never code. |
| **Goal** | One assessable item from a Framework Catalog (a CPG goal). The atomic unit of assessment and scoring. Synonym avoided: "control" (reserved for talking about other frameworks). |
| **Function** | A Goal grouping inherited from the catalog (CPG 2.0: the six CSF functions — Govern, Identify, Protect, Detect, Respond, Recover). Sub-scores roll up per Function. |
| **Implementation Level** | The ordinal answer for a Goal: `Not Implemented (0)` / `Partially (0.33)` / `Largely (0.67)` / `Fully (1.0)`, plus `Not Applicable` (excluded from scoring) and `Unknown` (scores 0, flagged). |
| **Answer** | An Implementation Level for one Goal, with timestamp and optional note. Unanswered Goals are implicitly `Unknown`. |
| **Assessment** | The current working set of Answers for a Site against a pinned catalog version. Mutable; always partially completable. |
| **Snapshot** | An immutable, sequence-numbered capture of the Assessment plus its computed Score, appended to history. The unit of "posture at a point in time." |
| **Score** | Derived result of scoring a Snapshot: Maturity Band (headline), percentage, per-Function sub-scores, floor-rule status. Deterministic: same Answers + same catalog ⇒ same Score. |
| **Maturity Band** | Headline rating: **Foundational → Developing → Managed → Resilient**. Thresholds defined in the catalog. The band, not the %, is the hero number. |
| **Floor Rule** | Constraint capping the headline Band at **Developing** while any Critical-tier Goal is `Not Implemented` or `Unknown` — a fatal gap cannot be averaged away. |
| **Criticality Tier** | `Critical` / `High` / `Standard` per Goal. Catalog ships cited defaults; a Site may apply a Tier Override. |
| **Tier Override** | A Site-local change to a Goal's tier, requiring a non-empty rationale; recorded and surfaced in reports (auditable, anti-gaming). |
| **Delta** | The difference between two Snapshots sharing a catalog version: signed score change plus its Drivers. |
| **Driver** | A Goal whose Answer changed between two Snapshots, with its signed point contribution. Driver contributions must sum to the Delta. |
| **Series Break** | An explicit marker in history when the catalog version changes without back-casting; Snapshots across a break are not numerically comparable. |
| **Back-cast** | Recomputing prior Snapshots under a new catalog version (where Goals map) so the trend stays comparable. Originals are retained; never silently rescored. |
| **Next Best Action** | A recommended Goal to improve next, ranked by potential score impact ÷ catalog effort rating. |
| **Exec Report** | The generated one-page static HTML (print-to-PDF) summary: Band, trend, biggest movers, top open risks, Next Best Actions. A pure projection — never a data source. |
| **Site** | The single plant/facility a Posture Store describes. One Site per store in v1. |
| **Posture Store** | The local, human-readable, git-friendly on-disk data root holding Site, Assessment, Snapshots, and Overrides. Never leaves the machine. |

## Deliberately Avoided Terms

| Term | Why avoided |
|------|------------|
| "Compliance" / "audit" | otposture measures posture, not regulatory compliance; claiming compliance invites legal weight the tool doesn't carry. |
| "MIL" / "maturity level" | C2M2's reserved vocabulary; using it implies C2M2 equivalence. We say Maturity Band. |
| "Risk score" | Commercial-platform connotation (proprietary, asset-level). otposture scores implementation posture, not quantified risk. |
| "Scan" | Implies network activity, which is a hard product boundary violation. |
