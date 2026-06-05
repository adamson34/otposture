---
document_type: adversarial-review
level: ops
version: "1.0"
status: complete
producer: adversary
timestamp: 2026-06-05T13:45:00
phase: 1d
inputs: [product-brief.md, "domain-spec/", prd.md, "prd-supplements/", "behavioral-contracts/"]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: prd.md
pass: 3
previous_review: pass-2.md
---

# Adversarial Review: otposture (Pass 3)

> **Cycle:** p1-greenfield · **Scope:** full (product-brief.md, domain-spec/, prd.md,
> prd-supplements/, behavioral-contracts/ — BC-INDEX + 33 BCs). Verification-properties
> and architecture artifacts do not yet exist on disk and were not in scope.

## Finding ID Convention

`ADV-P1GREEN-P03-<SEV>-<SEQ>` (cycle prefix P1GREEN, pass 03).

## Part A — Fix Verification (pass-2 findings)

| ID | Previous Severity | Status | Notes |
|----|-------------------|--------|-------|
| ADV-P1GREEN-P02-HIGH-001 | HIGH | RESOLVED | Floor rule is now positional. DI-003, ubiquitous-language.md, BC-4.04.004 (Desc + PC#1 + VP-048), and BC-4.04.003 all define the cap as "the catalog's second-lowest band (positional, not by name)" — survives band renames. (New gap surfaced by this fix: see P03-HIGH-001.) |
| ADV-P1GREEN-P02-MED-001 | MEDIUM | RESOLVED | `upgrade` now has a full behavioral contract — BC-2.02.009 (Store Schema Upgrade), VP-025/026/027, listed in BC-INDEX (SS-02 now 9 BCs, 33 total). BC-6.06.001 command surface includes `upgrade`; BC-2.02.004 EC-003 cross-references it as the on-disk counterpart. |
| ADV-P1GREEN-P02-MED-002 | MEDIUM | RESOLVED | VP reserved-range bookkeeping reconciles. module-criticality.md (v1.1) now reads posture-store "18 used, VP-010..027". Actual BC usage: SS-01 VP-001..008 (8), SS-02 VP-010..027 (18, incl. new VP-025..027), SS-03 VP-030..036 (7), SS-04 VP-040..059 (20), SS-05 VP-070..076 (7), SS-06 VP-080..086 (7) = 67 VPs, every one inside its BC-INDEX reserved range. |
| ADV-P1GREEN-P02-MED-003 | MEDIUM | RESOLVED | `verify --json` shape is now defined. interface-definitions.md publishes a distinct `kind: "verify-results"` document (snapshots_checked, score_mismatches, orphan_answers, integrity_errors, ok). BC-6.06.004 PC#2 describes the same distinct check-results document. |
| ADV-P1GREEN-P02-LOW-001 | LOW | RESOLVED | VP-058 reworded to be rank-position-independent ("raising that goal to FI ... rank-position-independent, so floor-gating reordering cannot falsify it"); BC-4.04.008 PC#3 makes floor-gating reorder explicit. No longer ambiguous under reorder. |
| ADV-P1GREEN-P02-LOW-002 | LOW | RESOLVED | Implementation-Level values pinned as exact rationals everywhere: DI-012 (NI=0, PI=1/3, LI=2/3, FI=1, Unknown=0; displayed 0.33/0.67; rational/fixed-point never binary float), BC-4.04.001 Desc, ubiquitous-language.md, BC-4.04.005 Inv 1. |

**Pass-1 spot re-verification (carried forward):** the pass-1 CRITICAL orphan-answer
contradiction stays resolved — BC-3.03.001 Inv 2 (quarantine at load, never enters
scoring set) and BC-2.02.004 EC-004 agree; VP-030 holds "at the scoring boundary."
`assess --only <unknown|all>` token and reserved-`stale` note are present (BC-3.03.002
PC#1). JSON-error-on-stdout is now the single documented exception (BC-6.06.001 Inv 3).

All six pass-2 findings RESOLVED. No regressions in previously-fixed material.

## Part B — New Findings

### HIGH

#### ADV-P1GREEN-P03-HIGH-001: Band-threshold count contradiction (4 vs 3) across catalog validator, scorer, entity model, and catalog format
- **Severity:** HIGH
- **Category:** contradictions
- **Location:** BC-1.01.002 rule (e); BC-4.04.003 precondition 1; domain-spec/entities.md FrameworkCatalog; prd-supplements/interface-definitions.md catalog YAML (`bands:`)
- **Description:** Four authoritative artifacts disagree on how many band thresholds a catalog has, which would yield a validator and a scorer that are mutually inconsistent.
  - **BC-1.01.002 rule (e):** "exactly **4** band thresholds, strictly increasing, within [0,1]."
  - **BC-4.04.003 precond 1:** "catalog provides 4 bands with strictly increasing thresholds **`t1 < t2 < t3`**" — i.e. **3** interior thresholds, consumed as `Foundational if percent < t1; Developing if t1 ≤ percent < t2; Managed if t2 ≤ percent < t3; Resilient if percent ≥ t3`.
  - **entities.md:** `band_thresholds[4]` — an array of length **4**.
  - **interface-definitions.md:** `bands: {foundational: 0.0, developing: 0.40, managed: 0.65, resilient: 0.85}` — **4** lower-bound entries, where `foundational: 0.0` is a trivial anchor (every percent ≥ 0), not a partition boundary.

  N bands partitioning [0,1] require exactly **N−1** interior thresholds. BC-4.04.003 is mathematically correct (3 thresholds for 4 bands). BC-1.01.002's "exactly 4 thresholds, strictly increasing" is either wrong (off by one) or is silently counting band *lower-bounds* including the 0.0 anchor — but then "strictly increasing within [0,1]" is unenforceable as written (the 0.0 anchor is fixed, and a validator literally requiring 4 *strictly increasing* thresholds will accept a fourth boundary that BC-4.04.003 never reads). An implementer building the validator from BC-1.01.002 and the scorer from BC-4.04.003 will produce divergent representations of the same catalog data — a DI-008/DI-010 reproducibility hazard at the SS-01/SS-04 seam.
- **Evidence:** Quoted lines above. Cross-check: `grep -rn "4 band\|t1 < t2\|band_thresholds" specs/` returns exactly these four conflicting statements.
- **Proposed Fix:** Pick one representation and propagate. Recommended: catalog stores **band definitions as 4 named lower-bounds** (the YAML form), with `foundational` pinned at 0.0; validation rule (e) becomes "the catalog defines N≥2 named bands as lower-bounds; the lowest is 0.0; the remaining N−1 are strictly increasing within (0,1]"; BC-4.04.003 precondition is restated in terms of "N−1 interior thresholds derived from the band lower-bounds." Update entities.md (`band_thresholds` is N lower-bounds, not a fixed `[4]`) and BC-1.01.002 rule (e) text to match. Read BC-4.04.003 and BC-1.01.002 together before editing — the band-count must be parameterized, not hard-coded to 4, to honor DI-008's "score any well-formed catalog."

### MEDIUM

#### ADV-P1GREEN-P03-MED-001: Positional floor-cap is undefined for catalogs with fewer than two bands; no validator rule enforces a minimum band count
- **Severity:** MEDIUM
- **Category:** missing-edge-cases
- **Location:** DI-003 / BC-4.04.004 (PC#1, VP-048) vs BC-1.01.002 (rule e) and BC-1.01.001 (EC-002)
- **Description:** The pass-2 fix redefined the floor cap positionally as "**the second-lowest band** in the catalog's ordered band list." This silently introduces a precondition: the catalog must have **≥2 bands**. Nothing enforces it. BC-1.01.002 rule (e) constrains only threshold ordering/range, not a minimum band count. BC-1.01.001 EC-002 explicitly admits a "minimal valid catalog: 1 function, 1 goal," and DI-008 promises "the same engine must score any well-formed catalog." For a 1-band catalog there is no second-lowest band, so `floor_cap_band` is undefined and `min(band_uncapped, floor_cap_band)` (BC-4.04.004 PC#1) and the Kani proof VP-048 ("final band ≤ floor_cap_band") reference a non-existent value — an E-SCORE-001 engine-invariant panic at best, undefined behavior at worst.
- **Evidence:** BC-4.04.004 PC#1 and VP-048 both name "the catalog's second-lowest band (positional)"; BC-1.01.002 rule (e) lists no minimum-band-count check; BC-1.01.001 EC-002 admits a minimal catalog.
- **Proposed Fix:** Add a catalog-validation rule (BC-1.01.002): "≥2 bands defined." Add a BC-4.04.004 edge case for the degenerate ≥2-but-floor-cap-is-top-band case (already implicitly fine since cap only lowers). Tie this to the HIGH-001 band-count parameterization fix so the two land together. Note in DI-008/DI-003 that the positional rule presumes the validator-enforced ≥2-band minimum.

### LOW

#### ADV-P1GREEN-P03-LOW-001: `migrate --mapping` + `--series-break` declared mutually exclusive in interface table, but no BC states or tests the exclusivity
- **Severity:** LOW
- **Category:** spec-fidelity
- **Location:** prd-supplements/interface-definitions.md Flag Interactions table vs BC-2.02.006 / BC-2.02.007
- **Description:** The Flag Interactions table asserts `migrate --mapping` + `--series-break` ⇒ "Usage error (exit 2) if both" as "mutually exclusive intents." Neither BC-2.02.006 (Back-Cast Migration) nor BC-2.02.007 (Series Break) states this exclusivity, and no canonical test vector covers passing both flags. BC-2.02.006 EC-001/EC-003 describe partial-mapping situations (goal splits, some goals unmappable) where a mapping is supplied yet a break could still be the chosen disposition, and BC-2.02.007 EC-002 shows mapping and breaks coexisting across history. The interface rule may be the intended design (one disposition per invocation), but it is an interface-only assertion with no behavioral-contract backing — an implementer reading only the BCs would not implement the exit-2 guard.
- **Evidence:** interface-definitions.md line 156 ("Usage error (exit 2) if both"); BC-2.02.006 EC-003 ("`--series-break` flag" as the unmappable fallback, no mention of conflict with `--mapping`); no `migrate --mapping --series-break` test vector in either SS-02 migration BC.
- **Proposed Fix:** Add the exclusivity to BC-2.02.006 (Invariants or an edge case) with an error vector (exit 2, or a CLI usage error), or relax the interface table if partial-mapping-plus-explicit-break is a legitimate combined intent. Make the BC and the interface table agree on one behavior.

## Summary

| Severity | Count |
|----------|-------|
| CRITICAL | 0 |
| HIGH | 1 |
| MEDIUM | 1 |
| LOW | 1 |

**Overall Assessment:** pass-with-findings
**Convergence:** findings remain — iterate (clean-pass count resets; this pass is NOT clean)
**Readiness:** requires revision (HIGH-001 is a hard BC-to-BC contradiction at the SS-01/SS-04 seam and should resolve before story decomposition)

## Novelty Assessment

| Field | Value |
|-------|-------|
| **Pass** | 3 |
| **New findings** | 3 |
| **Duplicate/variant findings** | 0 |
| **Novelty score** | 1.00 (3 / 3) |
| **Median severity** | 3.0 (MEDIUM) |
| **Trajectory** | 10 → 6 → 3 |
| **Verdict** | FINDINGS_REMAIN |

<!--
  Trajectory is monotonically decreasing (10→6→3) — no regression.
  Novelty score is 1.00 because all 3 findings are genuinely new (none is a
  restatement of a pass-1/pass-2 finding). HIGH-001 and MED-001 were *created
  by* the pass-2 positional-floor and 4-band fixes and could only surface once
  those landed. Per the adversarial-review skill: minimum 3 CLEAN passes to
  converge; this pass is not clean, so the clean-pass counter is at 0. At least
  3 more passes required after these findings are remediated.
-->
