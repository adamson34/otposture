---
document_type: adversarial-review-index
level: ops
version: "1.0"
status: in-review
producer: adversary
timestamp: 2026-06-05T14:40:00
phase: 1d
pass: 4
inputs: [product-brief.md, "domain-spec/", prd.md, "prd-supplements/", "behavioral-contracts/"]
traces_to: prd.md
total_findings: 3
severity_distribution: { CRIT: 0, HIGH: 0, MED: 1, LOW: 2 }
---

# Adversarial Review -- Pass 4

> **Scope:** full. Cycle p1-greenfield. Commit 13ca667. 33 BCs (BC-INDEX + ss-01..06)
> + 10 domain-spec sections + PRD + 4 supplements. Fresh-context pass.

## Finding Catalog

| ID | Severity | Category | Title | Status | Depends On | Blocks |
|----|----------|----------|-------|--------|-----------|--------|
| ADV-P1GREEN-P04-MED-001 | MEDIUM | spec-fidelity | JSON example arithmetically self-contradictory: band_uncapped=managed with percent 0.6176 (< managed threshold 0.65) | open | -- | story-decomposition (JSON schema consumers) |
| ADV-P1GREEN-P04-LOW-001 | LOW | spec-fidelity | NBA JSON field `gain` drifts from BC-4.04.008 canonical `potential_gain` | open | -- | -- |
| ADV-P1GREEN-P04-LOW-002 | LOW | ambiguous-language | L2-INDEX Key Decision #2 pins four band names/count without the catalog-data hedge (pending intent verification) | open | -- | -- |

## Dependency Graph

```text
[All findings are independent — no inter-finding dependencies.]
ADV-P1GREEN-P04-MED-001 and ADV-P1GREEN-P04-LOW-001 both live in
prd-supplements/interface-definitions.md (JSON Output Schema) — fix together in one edit.
ADV-P1GREEN-P04-LOW-002 is independent (domain-spec/L2-INDEX.md).
Only MED-001 is worth gating Phase-2 (it would seed a contradictory JSON golden fixture).
```

## Category Groups

| Category | Finding IDs | Can Triage in Parallel? |
|----------|------------|------------------------|
| spec-fidelity | P04-MED-001, P04-LOW-001 | Yes — both in interface-definitions.md, fix in one pass |
| ambiguous-language | P04-LOW-002 | Yes — independent (L2-INDEX) |

## Fix-Verification Rollup (pass-3 @ commit 13ca667)

All 3 pass-3 findings verified RESOLVED:
- P03-HIGH-001 (band-threshold count contradiction): unified N≥2 / N−1 model propagated across
  BC-1.01.001/002, BC-4.04.003/004, DI-003, entities.md; "4 bands/3 thresholds" only as bundled-catalog scoping.
- P03-MED-001 (min-band-count / floor-cap undefined): BC-1.01.002 rule (e) + EC-004 (N=1 is a defect);
  floor-cap defined positionally (second-lowest) everywhere.
- P03-LOW-001 (migrate flag exclusivity unbacked): BC-2.02.006 EC-004 (exit 2, mutually exclusive),
  mirrored in interface-definitions Flag Interactions + migrate synopsis; E-VAL-010 refusal consistent.

S-7.01 partial-fix axis: frontmatter, sibling SS-01/SS-04 BCs, and prose band-count references all moved together. No propagation gaps.

## Convergence Note

Trajectory 10 → 6 → 3 → 3 (no regression; count held, median severity dropped — pass-3 had a HIGH,
pass-4's ceiling is MEDIUM). Novelty 1.00 — all 3 findings genuinely new, clustered in the JSON-schema /
band-naming seam of the authoritative interface supplement, a region prior passes verified at the BC/DI
layer but did not cross-check against the literal JSON example's arithmetic.

This pass is NOT clean. Clean-pass counter remains 0. Minimum 3 consecutive CLEAN passes required to
converge. After remediation of MED-001/LOW-001/LOW-002, at least 3 more passes.
