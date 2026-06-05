---
document_type: domain-spec-section
level: L2
section: "invariants"
version: "1.0"
status: draft
producer: business-analyst
timestamp: 2026-06-05T12:10:44
phase: 1a
inputs: [product-brief.md, planning/research-report.md]
input-hash: "d0490fef4b5ce75c2b7fc3c0e135aeb5"
traces_to: L2-INDEX.md
---

# Domain Invariants

> **Sharded L2 section (DF-021).** Navigate via `L2-INDEX.md`.

| ID | Rule | Scope | Violation Behavior |
|----|------|-------|--------------------|
| DI-001 | Every Snapshot pins the exact catalog `(id, version)` it was scored under. Scores are numerically comparable only between Snapshots sharing a catalog version, or via explicit back-cast. | Snapshot, Delta | Cross-version diff attempts are refused with a Series Break explanation — never a silently wrong number. |
| DI-002 | Snapshot history is append-only and immutable. History is never rewritten, deleted, or silently rescored; back-casting stores new values **alongside** originals. | Posture Store | Any operation that would mutate an existing Snapshot is rejected; corruption of past snapshots is detected on load (FM-001). |
| DI-003 | **Floor rule:** the headline Maturity Band cannot exceed **Developing** while any Critical-tier Goal (effective tier, after overrides) is `Not Implemented` or `Unknown`. | Score | Scoring marks `floor_rule_triggered=true`, lists the gating `critical_gaps[]`, and every display surface shows why the band is capped. |
| DI-004 | `Not Applicable` Goals are excluded from numerator and denominator. `Unknown` Goals score 0 and are flagged distinctly from `Not Implemented`. | Score | A Function where all Goals are N/A has no sub-score and is excluded from the roll-up (DEC-002). |
| DI-005 | Every displayed Delta is fully attributed: the signed contributions of its Drivers sum exactly (within the defined rounding rule, DEC-008) to the total delta. | Delta | A delta that cannot be reconciled to drivers is a computation error — fail loudly, never display an unattributed delta. |
| DI-006 | A Tier Override requires a non-empty rationale and timestamp. The effective tier = active override if present, else catalog default. All reports disclose active overrides. | TierOverride | Override without rationale is rejected; reports that omit override disclosure are non-conforming (anti-gaming, DEC-005). |
| DI-007 | All domain operations are local. No network access, telemetry, or cloud dependency in any code path. | All capabilities | Architectural constraint: any networking dependency fails review. Offline-first is a product boundary, not a default. |
| DI-008 | Scoring parameters — weights, band thresholds, tier defaults, impact/effort metadata — live in catalog data, never in code. A user can reproduce any score by inspection. | FrameworkCatalog, Score | Hard-coded scoring constants are a defect; the same engine must score any well-formed catalog. |
| DI-009 | Every catalog carries licensing provenance. Only public-domain or permissively licensed framework content is embedded; restricted frameworks (IEC 62443, SANS text) may be referenced by ID only. | FrameworkCatalog | Catalog without provenance is refused at load (CAP-001). |
| DI-010 | Scoring is a pure, deterministic function: identical frozen Answers + identical catalog ⇒ identical Score (band, %, sub-scores, floor status). Band is always derived from the same computation — never set independently. | Score | Recomputing a stored Snapshot's Score must reproduce it bit-for-bit; mismatch indicates corruption or an engine regression (R-005). |
| DI-011 | An Assessment is scoreable at any completeness. Unanswered Goals default to `Unknown` (scored 0, flagged) rather than blocking. First scores are honest-and-low, not gated on completeness. | Assessment, Score | The tool never refuses to score for incompleteness; it discloses the Unknown count prominently instead. |
| DI-012 | Percentage formula: `percent = Σ(weight_g × level_value_g) / Σ(weight_g)` over applicable (non-N/A) Goals, computed per Function and overall. Level values: NI=0, PI=0.33, LI=0.67, FI=1.0, Unknown=0. | Score | Single formula for all roll-ups; any alternative aggregation in any surface is a defect. |

| DI-013 | **No PII by design.** The domain model contains no fields for personal data: no user accounts, no per-person attribution on Answers/Snapshots/Overrides, no names/emails/phone numbers anywhere in the schema. Free-text fields (Answer `note`, Override `rationale`, Site metadata) are documented as non-PII fields, and generated reports reference roles/teams, never individuals. | All entities, Reporting | Any schema addition carrying personal data fails review; report templates must not render content that the schema marks free-text without the non-PII guidance note in user docs. |

> DI-003 + DI-005 + DI-008 together are the product's credibility thesis: a number that cannot hide a fatal gap, cannot move without naming why, and can be re-derived by hand. DI-007 + DI-013 together are its trust thesis: nothing leaves the machine, and there is nothing personal in it to leak.
