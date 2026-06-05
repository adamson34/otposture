---
document_type: adversarial-review
level: ops
version: "1.0"
status: complete
producer: adversary
timestamp: 2026-06-05T14:40:00
phase: 1d
inputs: [product-brief.md, "domain-spec/", prd.md, "prd-supplements/", "behavioral-contracts/"]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: prd.md
pass: 4
previous_review: ADV-P3-INDEX.md
---

# Adversarial Review: otposture (Pass 4)

> **Scope:** full. Cycle p1-greenfield. Reviewed at commit 13ca667.
> Artifacts: product-brief.md, all 10 domain-spec sections + L2-INDEX, prd.md,
> 4 prd-supplements, BC-INDEX + 33 behavioral contracts (ss-01..06).
> Fresh-context pass: prior pass *finding bodies* were not loaded.

## Finding ID Convention

`ADV-P1GREEN-P04-<SEV>-<SEQ>`

## Part A — Fix Verification (pass-3 remediations at commit 13ca667)

| ID | Previous Severity | Status | Notes |
|----|-------------------|--------|-------|
| ADV-P1GREEN-P03-HIGH-001 | HIGH | RESOLVED | Unified band model landed: BC-1.01.001 PC3, BC-1.01.002 rule (e), BC-4.04.003 precondition/PC1, BC-4.04.004, DI-003, and entities.md `bands[]` all state "N ≥ 2 bands / N−1 interior thresholds." No surviving "exactly 4 bands / 3 thresholds" engine-level assertion; the "4 bands, 3 thresholds" mentions are explicitly scoped "bundled catalog." Consistent. |
| ADV-P1GREEN-P03-MED-001 | MEDIUM | RESOLVED | Min-band-count guard present: BC-1.01.002 rule (e) "N ≥ 2 guarantees the positional floor-cap band exists" + EC-004 "Single-band catalog (N=1) → Defect (e)." Floor-cap defined positionally (second-lowest) in DI-003, BC-4.04.004 PC1, ubiquitous-language. |
| ADV-P1GREEN-P03-LOW-001 | LOW | RESOLVED | `migrate --mapping` XOR `--series-break` now BC-backed: BC-2.02.006 EC-004 ("Usage error, exit 2 — mutually exclusive per invocation"), mirrored in interface-definitions.md Flag Interactions table and the migrate synopsis; E-VAL-010 covers the no-mapping/no-break refusal consistently across BC EC-003, error-taxonomy, and CLI synopsis. |

All three pass-3 findings fully propagated. No partial-fix regressions detected (S-7.01 axis): frontmatter, sibling SS-01/SS-04 BCs, and prose references to band counts all updated together.

## Part B — New Findings

### CRITICAL

None.

### HIGH

None.

### MEDIUM

#### ADV-P1GREEN-P04-MED-001: JSON example in interface-definitions.md is arithmetically self-contradictory (band_uncapped vs percent)
- **Severity:** MEDIUM
- **Confidence:** HIGH
- **Category:** spec-fidelity
- **Location:** `prd-supplements/interface-definitions.md` lines 73–80 (JSON Output Schema, `score` block) vs the catalog band thresholds in the same file lines 134–138.
- **Description:** The illustrative `status --json` document shows `"percent": 0.6176`, `"band": "developing"`, `"band_uncapped": "managed"`, `"floor_rule": {"triggered": true, ...}`. The catalog example in the same file defines bundled thresholds developing≥0.40, managed≥0.65, resilient≥0.85. Per BC-4.04.003 PC1 (band i+1 iff t_i ≤ percent < t_{i+1}, boundary belongs to the higher band), a percent of 0.6176 falls in [0.40, 0.65) and therefore maps to **developing**, not managed. For `band_uncapped` to be "managed," the percent must be ≥ 0.65. The example therefore depicts a floor-cap (managed → developing) that its own percent cannot produce.
- **Evidence:**
  - interface-definitions.md L75–77: `"percent": 0.6176, "band": "developing", "band_uncapped": "managed"`.
  - interface-definitions.md L136–137: `{name: developing, threshold: 0.40}` / `{name: managed, threshold: 0.65}`.
  - BC-4.04.003.md L38 PC1: "band i+1 if t_i ≤ percent < t_{i+1} … a boundary always belongs to the higher band."
  - BC-4.04.003.md L60 canonical vector: "percent .649999 → Developing."
- **Why it matters:** interface-definitions.md is declared "Authoritative command surface" (L16) and is the named input for implementer and test-writer. A test-writer building JSON golden fixtures from this example would encode a contradictory case where the band-derivation function (the product's credibility centerpiece, formally verified by VP-046/047) is shown violating its own monotone partition. It also undermines KD-002/KD-003 ("reproducible by hand") in the one place a reader checks the wire format.
- **Proposed Fix:** Either raise the example percent into the managed range (e.g., `"percent": 0.6776`) so band_uncapped=managed is correct, or set `"band_uncapped": "developing"` and drop the floor-cap from this example (since a developing-band score cannot be capped further to developing). Recommended: bump percent to a value ≥ 0.65 so the example continues to demonstrate a live floor cap, which is the more instructive case. After fixing, re-confirm `assessed/total`, `unknown_count`, `na_count` still reconcile (currently 30 + 4 = 34, na 1 ⊆ assessed — internally fine).

### LOW

#### ADV-P1GREEN-P04-LOW-001: NBA JSON field name `gain` drifts from BC-4.04.008 canonical name `potential_gain`
- **Severity:** LOW
- **Confidence:** HIGH
- **Category:** spec-fidelity
- **Location:** `prd-supplements/interface-definitions.md` line 91 (`next_actions[]`) vs `behavioral-contracts/ss-04/BC-4.04.008.md` lines 39–41, 68.
- **Description:** The authoritative JSON schema outline names the next-best-action score field `"gain"`, while BC-4.04.008 (the owning contract) consistently calls the same quantity `potential_gain` in its postconditions, tie-break rule, and VP-058. Field-name drift in the wire schema vs the contract that defines the value.
- **Evidence:**
  - interface-definitions.md L91: `"next_actions": [{"goal": "3.D", "gain": 0.06, "effort": "simple", "impact": "high", "floor_gating": false}]`.
  - BC-4.04.008.md L39: "`potential_gain` = percent increase if raised to FI"; L41 "higher potential_gain, then catalog order"; L68 VP-058 "stated potential_gain."
- **Why it matters:** BC-6.06.004 inv. 2 mandates stable JSON field names and VP-085 validates JSON against a published schema. If the published schema is authored from the BC (`potential_gain`) but the interface doc shows `gain`, the two anchor documents disagree on the canonical field name before implementation begins. Low blast radius (one field, one consumer), but it is exactly the kind of cross-document field-name drift that surfaces as a golden/schema mismatch in Phase 3.
- **Proposed Fix:** Pick one name and align both documents. Recommend `potential_gain` (matches the contract and VP-058 wording, and is unambiguous against the rounded `gain` display label). Update interface-definitions.md L91 to `"potential_gain": 0.06`.

#### ADV-P1GREEN-P04-LOW-002: L2-INDEX "Key Domain Decision #2" states bands as a fixed four-name set without the catalog-data hedge that every other surface carries (pending intent verification)
- **Severity:** LOW
- **Confidence:** MEDIUM
- **Category:** ambiguous-language
- **Location:** `domain-spec/L2-INDEX.md` line 85.
- **Description:** L2-INDEX Key Domain Decision #2 reads "**Bands:** four named — Foundational / Developing / Managed / Resilient (thresholds in catalog data)." It pins band *names* and *count* as a domain decision while only the *thresholds* are flagged as catalog data. Every other surface that survived the pass-3 unification (DI-003, DI-008, BC-1.01.001/002, BC-4.04.003/004, ubiquitous-language Maturity Band entry, ASM-006 "rename bands; thresholds unchanged — names are catalog data, DI-008") treats band names AND count as catalog data, renameable without engine change. This single index line still reads as if four names are a fixed engine commitment.
- **Evidence:**
  - L2-INDEX.md L85: "Bands: four named — … (thresholds in catalog data)" — only thresholds hedged.
  - ASM-006 (assumptions.md L26): "rename bands; thresholds unchanged (names are catalog data, DI-008)."
  - DI-008 (invariants.md L28): "Scoring parameters — weights, band thresholds, tier defaults … live in catalog data, never in code."
- **Why it matters:** Not a hard contradiction — the line is defensibly read as "the bundled catalog's decision is four named bands." But it is the one place a reader sampling the domain index could conclude the band count/names are engine-level, contradicting the positional/N≥2 model. Because adjudicating whether the author intends this as a bundled-catalog default vs. an engine commitment depends on authorial intent, this is reported per the adversary's intent-adjudication rule.
- **Proposed Fix:** Add the catalog-data hedge to names+count, e.g., "Bands: the bundled CPG catalog defines four named bands (Foundational / Developing / Managed / Resilient); names, count (N ≥ 2), and thresholds are all catalog data (DI-008)." Orchestrator/human to confirm intent before applying.

## Summary

| Severity | Count |
|----------|-------|
| CRITICAL | 0 |
| HIGH | 0 |
| MEDIUM | 1 |
| LOW | 2 |

**Overall Assessment:** pass-with-findings
**Convergence:** findings remain — iterate (this pass is NOT clean)
**Readiness:** near-ready; one MEDIUM spec-fidelity defect in the authoritative interface doc should be fixed before Phase-2 story decomposition consumes the JSON schema.

## Novelty Assessment

| Field | Value |
|-------|-------|
| **Pass** | 4 |
| **New findings** | 3 |
| **Duplicate/variant findings** | 0 |
| **Novelty score** | 1.00 (3 / 3) |
| **Median severity** | 2.0 (LOW–MEDIUM) |
| **Trajectory** | 10 → 6 → 3 → 3 |
| **Verdict** | FINDINGS_REMAIN |

**Trajectory note:** count held at 3 (not increased) — no regression. Median severity fell (pass-3 had a HIGH; pass-4's max is MEDIUM). All 3 findings are genuinely new and concentrated in the JSON-schema / band-naming seam of the authoritative interface supplement — a region prior passes touched at the BC/DI level but did not cross-check against the literal JSON example's arithmetic. None are re-statements of prior findings. The unified band model from pass-3 is confirmed fully propagated.

**Convergence status:** Clean-pass counter remains 0 (this pass found defects). Minimum 3 consecutive CLEAN passes required. After remediation of MED-001/LOW-001/LOW-002, at least 3 more passes are required.
