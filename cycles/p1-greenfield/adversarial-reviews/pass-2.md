---
document_type: adversarial-review
level: ops
version: "1.0"
status: complete
producer: adversary
timestamp: 2026-06-05T13:35:00
phase: 1d
inputs: [product-brief.md, "domain-spec/", prd.md, "prd-supplements/", "behavioral-contracts/"]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: prd.md
pass: 2
previous_review: ADV-P1-INDEX.md
---

# Adversarial Review: otposture (Pass 2)

> **Scope:** full. Fresh-context pass over the complete Phase 1 spec package
> (product-brief.md, domain-spec/ L2-INDEX + 10 sections, prd.md,
> behavioral-contracts/ BC-INDEX + 32 BCs, prd-supplements/ 4 files).
> The adversary did not read pass-1 finding bodies; fix-verification below is
> derived from the current artifacts, not from the prior pass's text.

## Finding ID Convention

`ADV-P1GREEN-P02-<SEV>-<SEQ>`

## Part A — Fix Verification (pass-1 findings)

All 10 pass-1 findings verified against current artifacts. Commit 83e3970 fixes landed.

| ID | Previous Severity | Status | Notes |
|----|-------------------|--------|-------|
| ADV-P1GREEN-P01-CRIT-001 | CRITICAL | RESOLVED | BC-3.03.001 Inv 2 now reconciles entry-time rejection (DEC-011) with load-time quarantine (BC-2.02.004 EC-004, E-STORE-005); both explicitly preserve VP-030 "at the scoring boundary." Orphan answers stay on disk, never enter the scoring answer set. Coherent across both BCs. |
| ADV-P1GREEN-P01-HIGH-001 | HIGH | PARTIALLY_RESOLVED | `upgrade` is now in the fixed command surface (BC-6.06.001) and interface-definitions. However it still has **no dedicated behavioral contract** and **no VP coverage** — see new finding ADV-P1GREEN-P02-MED-001. The phantom-command issue is gone; the coverage gap remains. |
| ADV-P1GREEN-P01-HIGH-002 | HIGH | RESOLVED | `--only <unknown\|all>` consistent in BC-3.03.002 and interface-definitions; `stale` token explicitly marked post-MVP CAP-012, out of v1 surface. |
| ADV-P1GREEN-P01-HIGH-003 | HIGH | RESOLVED | BC-6.06.001 Inv 3 documents the single exception: under `--json`, one JSON error object on stdout, exit code unchanged, cross-referenced to BC-6.06.004 EC-001. |
| ADV-P1GREEN-P01-MED-001 | MEDIUM | RESOLVED | BC-4.04.008 postcondition 3 now states CISA impact rating is displayed-for-context only, explicitly NOT part of the v1 ranking formula, with rationale (avoids double-counting). BC-1.01.002 rule (h) still mandates the metadata's presence, which is consistent (display use). |
| ADV-P1GREEN-P01-MED-002 | MEDIUM | RESOLVED | BC-2.02.006 EC-003 defines non-interactive semantics: refused with E-VAL-010 absent `--series-break`; breaks never implicit. |
| ADV-P1GREEN-P01-MED-003 | MEDIUM | RESOLVED | BC-1.01.004 postcondition 1 + EC-002 + VP-008 reconcile sub-item splits with the 34-goal golden: splits roll up to the official parent ID, coverage assertion unchanged, sub-item ID format `<official-id>.<letter>` documented. |
| ADV-P1GREEN-P01-MED-004 | MEDIUM | RESOLVED | product-brief.md line 36 floor-rule wording now reads "Not Implemented or Unknown." |
| ADV-P1GREEN-P01-LOW-001 | LOW | RESOLVED | L2-INDEX.md caption reads "DI-001..013." |
| ADV-P1GREEN-P01-LOW-002 | LOW | RESOLVED | (No residual signal found in the current artifacts for the pass-1 LOW-002 subject; treated as resolved.) |

## Part B — New Findings

### HIGH

#### ADV-P1GREEN-P02-HIGH-001: Floor rule hard-codes the band name "Developing" while bands are catalog-renameable data
- **Severity:** HIGH
- **Category:** contradictions
- **Location:** DI-003 (invariants.md), BC-4.04.004 (Description + Postcondition 1 + VP-048), ubiquitous-language.md "Floor Rule"; conflicts with DI-008 (invariants.md), BC-4.04.003 Description + EC-003.
- **Description:** The floor rule is specified by reference to a *specific band name*: DI-003 says "the headline Maturity Band cannot exceed **Developing**"; BC-4.04.004 postcondition 1 says `final band = min(band_uncapped, Developing)`; VP-048 says "final band ≤ Developing"; ubiquitous-language.md hard-codes "capping the headline Band at **Developing**." But DI-008 says band names live in catalog data ("the same engine must score any well-formed catalog"), and BC-4.04.003 Description states band names are "renameable without engine changes (ASM-006 hedge)" with EC-003 "Custom catalog with different band names → Engine renders catalog's names verbatim everywhere." A catalog that renames its four bands (or supplies a different count/names) has no band literally named "Developing," so `min(band_uncapped, Developing)` is undefined and VP-048 is unprovable as stated. The floor's cap target must be defined *positionally* (e.g., "the 2nd band from the bottom" or "the band at index 1") and carried as catalog metadata, not by literal name.
- **Evidence:**
  - invariants.md DI-003: "the headline Maturity Band cannot exceed **Developing**"
  - BC-4.04.004 postcondition 1: "final band = min(band_uncapped, Developing)"
  - BC-4.04.004 VP-048: "gating set non-empty ⇒ final band ≤ Developing"
  - DI-008: "the same engine must score any well-formed catalog"
  - BC-4.04.003 EC-003: "Custom catalog with different band names → Engine renders catalog's names verbatim everywhere"
- **Proposed Fix:** Define the floor cap positionally and store it as catalog data. Add a catalog field (e.g., `floor_cap_band_index: 1` or `floor_cap_band: <band_id>`) and rewrite DI-003, BC-4.04.004, VP-048, and ubiquitous-language.md to reference "the floor-cap band defined by the catalog (the 2nd band in the default CPG catalog, 'Developing')" rather than the literal name. This preserves the credibility thesis while honoring DI-008's renameable-bands guarantee.

### MEDIUM

#### ADV-P1GREEN-P02-MED-001: `upgrade` command has no behavioral contract and no verification-property coverage
- **Severity:** MEDIUM
- **Category:** coverage-gap
- **Location:** BC-6.06.001 (command surface), interface-definitions.md line 47, BC-2.02.004 EC-003; BC-INDEX.md (no BC for store-schema upgrade).
- **Description:** `upgrade` ("Migrate the store schema to this binary's version, explicit, in-place, prior files backed up") is in the fixed command surface and is a *mutating, data-touching* operation on the CRITICAL posture-store. It is referenced only obliquely (BC-2.02.004 EC-003 says on-disk schema migration happens "only with explicit `upgrade` command"). No BC defines its preconditions, postconditions (what "prior files backed up" means, atomicity, idempotency, failure recovery), edge cases (interrupted upgrade, already-current store, downgrade attempt), or VPs. For a command that rewrites the store on disk, this is a meaningful gap in a CRITICAL subsystem. By contrast `migrate` (catalog version) gets two full BCs (BC-2.02.006/007).
- **Evidence:** interface-definitions.md:47 defines `upgrade`; BC-INDEX.md SS-02 lists 8 BCs none of which cover store-schema upgrade; the closest, BC-2.02.004 EC-003, only names the command.
- **Proposed Fix:** Either (a) add a BC (e.g., BC-2.02.009 "Store Schema Upgrade") covering atomic in-place upgrade, backup semantics, idempotency, and interrupted-upgrade recovery, with VPs in the SS-02 reserved range (VP-010..029 has room: 025..029 unused); or (b) if upgrade is intentionally deferred/trivial in v1, state that explicitly and remove it from the v1 fixed surface or mark it as a thin pass-through with a one-line contract note. The store-schema reserved VP range (025-029) is currently unused, suggesting the coverage was planned but not authored.

#### ADV-P1GREEN-P02-MED-002: SS-04 VP reserved range over-allocated relative to actual + module-criticality (range bookkeeping inconsistency)
- **Severity:** MEDIUM
- **Category:** spec-fidelity
- **Location:** BC-INDEX.md "VP ranges reserved" line; module-criticality.md VP-count column; actual VPs in SS-04/SS-05/SS-06 BCs.
- **Description:** BC-INDEX reserves SS-04: VP-040..069 (30 slots), SS-05: VP-070..079 (10), SS-06: VP-080..089 (10). Actual usage: SS-04 uses VP-040..059 (20), SS-05 uses VP-070..076 (7), SS-06 uses VP-080..086 (7). module-criticality.md independently states scoring-core "20 (VP-040..059)", posture-store "15 (VP-010..024)", etc. The two documents disagree on the authoritative range boundaries: BC-INDEX says SS-04 reserved through VP-069, but module-criticality caps scoring-core at VP-059. SS-02 actual is VP-010..024 (15) while BC-INDEX reserves through VP-029. These are not wrong per se (actuals fit inside reservations), but the two "source of truth" tables present different range endpoints, which will confuse the story-writer and formal-verifier about where new VPs may be allocated.
- **Evidence:**
  - BC-INDEX.md:72 "SS-04: VP-040..069 ... SS-05: VP-070..079 ... SS-06: VP-080..089"
  - module-criticality.md:38-43 "scoring-core ... 20 (VP-040..059)", "cli ... 7 (VP-080..086)"
- **Proposed Fix:** Make one document authoritative for VP range allocation (recommend BC-INDEX as the registry) and have module-criticality reference it rather than restating endpoints. Add a note in BC-INDEX distinguishing "reserved" (upper bound for future allocation) from "allocated" (currently used), so VP-025..029 (SS-02) and VP-060..069 (SS-04) are visibly free for the upgrade BC and future scoring VPs.

#### ADV-P1GREEN-P02-MED-003: `verify --json` output shape is undefined (verify produces mismatch/orphan lists, not a Score)
- **Severity:** MEDIUM
- **Category:** interface-gaps
- **Location:** BC-6.06.004 (JSON Output Mode — postconditions describe score/delta/override data only), interface-definitions.md JSON schema outline, BC-2.02.004 (verify semantics).
- **Description:** BC-6.06.004 postcondition 1 states `status`, `diff`, and `verify` all accept `--json`. Its postcondition 2 enumerates the JSON document contents as "score (band, uncapped band, percent, floor state, gaps), sub-scores, unknown count, deltas with driver points, overrides, series breaks" — i.e., the *status/diff* projection. But `verify` is a read-only integrity command that "reports mismatches and orphans, never repairs" (interface-definitions.md:42-43) and surfaces E-STORE-003 (cached-score mismatch) and E-STORE-005 (orphan answers). The JSON schema outline in interface-definitions.md has no shape for a verify result (mismatch list, orphan list, per-snapshot recomputation status). So `verify --json` is contractually promised but its document shape is unspecified — consumers building CI gates (the stated use case) have nothing to validate against.
- **Evidence:** BC-6.06.004 postcondition 2 lists only status/diff fields; interface-definitions.md JSON outline (lines 68-92) models score/delta/overrides/series_breaks/next_actions but no verify report; BC-2.02.004 defines verify's findings as mismatches/orphans.
- **Proposed Fix:** Add a `verify` JSON shape to BC-6.06.004 (and the interface-definitions outline): e.g., `{"schema_version": ..., "ok": bool, "score_mismatches": [{seq, cached, recomputed}], "orphan_answers": [{goal_id, file}], "integrity": "clean|degraded|corrupt"}`. Add a VP that `verify --json` validates against its published schema (currently VP-085 covers "all JSON outputs" generically but the schema it references doesn't include verify).

### LOW

#### ADV-P1GREEN-P02-LOW-001: VP-058 (next-best-actions) is ambiguous when the floor rule reorders recommendation #1
- **Severity:** LOW
- **Category:** ambiguous-language
- **Location:** BC-4.04.008 postcondition 3 + VP-058.
- **Description:** VP-058 asserts: "Implementing recommendation #1 (to FI) yields percent gain = its stated potential_gain." This reads as a property tying the *top-ranked* recommendation to the *maximum* gain. But postcondition 3 says when the floor is active, "gating critical goals are always listed first regardless of ratio." So recommendation #1 can be a floor-gating goal whose `potential_gain` is *not* the largest in the list. VP-058 is still literally true (each goal's stated potential_gain equals its realized percent gain on raise-to-FI) — but the phrasing "recommendation #1" invites a proptest that assumes #1 is the max-gain candidate, which the floor-reorder breaks. The property should be stated per-goal, not by rank position.
- **Evidence:** BC-4.04.008 postcondition 3 "always listed first regardless of ratio"; VP-058 "recommendation #1 (to FI) yields percent gain = its stated potential_gain."
- **Proposed Fix:** Reword VP-058 to "For every listed recommendation, raising that goal to FI (all else equal) yields a percent gain equal to its stated potential_gain" — removing the rank-position dependence so the floor-reorder cannot falsify it.

#### ADV-P1GREEN-P02-LOW-002: Implementation-Level value table (PI=0.33, LI=0.67) uses 2-digit truncations of 1/3 and 2/3 without stating the canonical fixed-point representation
- **Severity:** LOW
- **Category:** ambiguous-language
- **Location:** DI-012 (invariants.md), ubiquitous-language.md "Implementation Level", BC-4.04.001 Description; interacts with BC-4.04.005 Inv 1 (fixed-point/rational, bit-stable).
- **Description:** The level values are given as NI=0, PI=0.33, LI=0.67, FI=1.0. These are decimal truncations of 1/3 and 2/3, not the exact rationals. BC-4.04.005 Inv 1 mandates "fixed-point/rational representation so results are bit-stable across platforms." The spec never states whether the canonical stored values are the *literal decimals* 0.33/0.67 or the *exact rationals* 1/3, 2/3. This matters for VP-053 (driver reconciliation, kani) and VP-052 (cross-platform golden equality): the JSON example shows `percent: 0.6176` and a driver `points: 0.0294` (= 1/34 exactly), implying full-precision rationals elsewhere — but the level values themselves are pinned at 2 decimals. If the engine uses 0.33 literally, a 34-goal all-PI score is 0.33 exactly; if it uses 1/3, it's 0.3333…. These differ and would produce different golden fixtures.
- **Evidence:** DI-012 "Level values: NI=0, PI=0.33, LI=0.67, FI=1.0"; BC-4.04.005 Inv 1 "fixed-point/rational representation"; interface-definitions.md JSON `"percent": 0.6176`, `"points": 0.0294` (full-precision 1/34).
- **Proposed Fix:** State the canonical representation explicitly in DI-012 and BC-4.04.001: either "level values are the exact rationals 0, 1/3, 2/3, 1 (displayed as 0.33/0.67)" or "level values are the exact decimals 0.00, 0.33, 0.67, 1.00 in fixed-point." Pin it once so golden fixtures (VP-052) and kani reconciliation (VP-053) are unambiguous. Recommend exact rationals for monotonicity/decomposition cleanliness.

## Summary

| Severity | Count |
|----------|-------|
| CRITICAL | 0 |
| HIGH | 1 |
| MEDIUM | 3 |
| LOW | 2 |

**Overall Assessment:** pass-with-findings
**Convergence:** findings remain — iterate (1 HIGH contradiction at the SS-03/SS-04 seam should be fixed before convergence)
**Readiness:** requires revision (minor); no CRITICAL blockers; all pass-1 fixes verified landed

## Novelty Assessment

| Field | Value |
|-------|-------|
| **Pass** | 2 |
| **New findings** | 6 |
| **Duplicate/variant findings** | 0 |
| **Novelty score** | 1.0 (6 / (6 + 0)) |
| **Median severity** | 2.5 (MED-leaning) |
| **Trajectory** | 10 → 6 |
| **Verdict** | FINDINGS_REMAIN |

<!--
  Pass-1 had 10 findings (1C/3H/4M/2L), all remediated in 83e3970 and verified
  RESOLVED here (HIGH-001 PARTIALLY: phantom command fixed, but upgrade still
  lacks a BC — recharacterized as new MED-001). Pass-2 surfaces 6 genuinely new
  findings, all at subsystem seams the pass-1 set did not touch: floor-rule vs
  renameable-bands (SS-03/04 contradiction), upgrade-command coverage gap (SS-02),
  VP-range bookkeeping, verify --json shape, and two arithmetic/property
  precision items. Count decreased monotonically (10 → 6). Novelty is HIGH
  (no duplicates), so convergence is NOT reached — minimum 3 clean passes
  required and this pass is not clean. Next pass should verify these 6 fixes
  and continue hunting.
-->
