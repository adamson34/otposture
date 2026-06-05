---
document_type: adversarial-review
level: ops
version: "1.0"
status: complete
producer: adversary
timestamp: 2026-06-05T15:31:45
phase: 1d
inputs: [product-brief.md, "domain-spec/", prd.md, "prd-supplements/", "behavioral-contracts/", "architecture/", "verification-properties/"]
input-hash: "66f4478"
traces_to: prd.md
pass: 8
previous_review: ADV-P7-INDEX.md
---

# Adversarial Review: otposture (Pass 8)

> **Scope:** full. Cycle p1-greenfield. Commit reviewed: 66f4478 (post pass-7 remediation).
> Corpus: product-brief + 10 domain-spec sections + PRD + 4 supplements + 33 BCs + architecture
> (ARCH-INDEX + 8 sections + 8 ADRs) + verification-properties (VP-INDEX + 10 formal VP files).

## Finding ID Convention

`ADV-P1GF-P08-<SEV>-<SEQ>` (cycle prefix `P1GF`, pass `P08`).

## Part A — Fix Verification (pass-7 remediations @ 66f4478)

| ID | Previous Severity | Status | Notes |
|----|-------------------|--------|-------|
| ADV-P1GF-P07-HIGH-001 | HIGH | RESOLVED | VP total is now consistently **67** everywhere: VP-INDEX header ("all 67 VPs"), VP-INDEX status table (draft=67), verification-coverage-matrix Coverage Accounting line ("Total VPs: 67 … 8+18+7+20+7+7"), BC-INDEX reserved ranges. Re-derived the arithmetic independently: 001–008 (8) + 010–027 (18) + 030–036 (7) + 040–059 (20) + 070–076 (7) + 080–086 (7) = 67. No residual "70" anywhere. |
| ADV-P1GF-P07-MED-001 | MEDIUM | RESOLVED | The per-crate cargo-deny I/O ban is now correctly scoped. purity-boundary-map "Rules enforced by CI" distinguishes (a) **op-score** — crate-granular cargo-deny ban (VP-051), feasible because op-score has zero workspace deps; from (b) **op-store::codec / op-assess / op-render** — submodule-level clippy import-lint + review gate, explicitly noting cargo-deny "cannot apply here" because these crates legitimately depend on the op-store crate (whose `fs` submodule carries I/O). tooling-selection.md now reads "per-crate I/O bans **for pure core**". ADR-007's whole-workspace ban is correctly limited to *networking* crates, not I/O generally. No contradiction remains. |
| ADV-P1GF-P07-LOW-001 | LOW | RESOLVED | Every BC sampled (BC-1.01.002/004, BC-3.03.003/004, BC-4.04.001–008, BC-5.05.001/003, BC-6.06.001/004) now carries a concrete `Architecture Module` row pointing at the owning crate + module-decomposition.md; no "(filled by architect)" placeholders remain in the traceability tables. (The `Stories` row remains "(filled by story-writer)", which is correct for phase 1d — stories are Phase 2.) |

All three pass-7 findings verified RESOLVED with full propagation. No regressions detected in the previously-converged BC/domain/PRD/supplement corpus.

## Part B — New Findings

All three new findings are localized to the JSON Output Schema example in
`prd-supplements/interface-definitions.md` (the authoritative machine-surface artifact, BC-6.06.004).
This region was first exercised against literal arithmetic in pass-4; the pass-4 fix corrected the
band/percent mapping but did not cross-check the denominator convention or the delta reconciliation.
These are distinct latent defects in the same example block, each of which would seed a contradictory
JSON golden fixture for downstream consumers (test-writer, implementer, VP-085/086).

### HIGH

#### ADV-P1GF-P08-HIGH-001: JSON example `percent_exact` divides by total goals, not the applicable (non-N/A) set — contradicts DI-012 / BC-4.04.001
- **Severity:** HIGH
- **Category:** contradictions / spec-fidelity
- **Location:** `prd-supplements/interface-definitions.md` JSON Output Schema, `score` block (the `percent_exact`, `na_count`, `total` fields)
- **Description:** The example `score` block states `"percent": 0.6765`, `"percent_exact": "23/34"`, `"na_count": 1`, `"total": 34`, `"assessed": 30`, `"unknown_count": 4`. The denominator in `percent_exact` is **34 (total goals)**. But DI-012, BC-4.04.001 (post-2), and BC-4.04.002 (post-1) all mandate that the percent formula divides by `Σ(weight)` over the **applicable** set = all goals **minus N/A-answered goals**. With `na_count = 1` and uniform v1 weights (ADR-005), the applicable denominator is `34 − 1 = 33`, so `percent_exact` must be of the form `X/33`, never `X/34`.
- **Evidence:**
  - DI-012 (domain-spec/invariants.md): "`percent = Σ(weight_g × level_value_g) / Σ(weight_g)` over applicable (non-N/A) Goals".
  - BC-4.04.001 post-2: "Applicable set = all catalog goals minus N/A-answered goals".
  - The example is internally self-masking: `23/34 = 0.67647… = 0.6765`, so the *displayed* `percent` matches the wrong fraction. A reader checking only `percent` vs `percent_exact` sees consistency; only checking against `na_count` exposes the defect.
  - `23/33 = 0.6970`, which would map to band `managed` (≥ 0.65) just like 0.6765 — so the band/floor narrative survives either way, which is exactly why this slipped past the pass-4 band-arithmetic fix.
- **Proposed Fix:** Recompute the example as a self-consistent vector. Pick a numerator over the **applicable** denominator 33 (e.g., `"percent_exact": "23/33"`, `"percent": 0.6970`, which still yields `band_uncapped: "managed"`), OR set `na_count: 0` and keep `/34`. Whichever is chosen, ensure `percent = numerator/denominator` AND `denominator = total − na_count` hold simultaneously, then re-derive `band`/`band_uncapped` from the new percent. Add an inline comment in the schema example noting "denominator excludes N/A goals (DI-012)" to prevent regression.

### MEDIUM

#### ADV-P1GF-P08-MED-001: JSON example `delta` block does not reconcile — single driver (0.0294) + composition_adjustment (0.0) ≠ percent_delta (0.08), violating DI-005 / BC-4.04.006 / VP-053
- **Severity:** MEDIUM
- **Category:** contradictions / spec-fidelity
- **Location:** `prd-supplements/interface-definitions.md` JSON Output Schema, `delta` block
- **Description:** The example `delta` block states `"percent_delta": 0.08`, `"composition_adjustment": 0.0`, and a single driver `{"goal": "2.E", "from": "not", "to": "full", "points": 0.0294}`. BC-4.04.006 (post-2) and DI-005 require **exact reconciliation**: `Σ driver_points + composition_adjustment = percent_delta`. Here `0.0294 + 0.0 = 0.0294 ≠ 0.08`. The flagship reconciliation property VP-053 ("driver sum + composition adjustment = delta") is literally falsified by the canonical schema example.
- **Evidence:**
  - BC-4.04.006 post-2: "Exact reconciliation at full precision: Σ driver points + composition_adjustment = percent_delta … No unexplained remainder, ever."
  - DI-005 (invariants.md): "the signed contributions of its Drivers sum exactly … to the total delta. A delta that cannot be reconciled to drivers is a computation error — fail loudly."
  - BC-4.04.006 canonical test vector line: "2.E: NI→FI (w=1), 34 equal weights | One driver +1/34; percent_delta = +0.0294…; displays as '+3%'." This is the *correct* single-driver value (0.0294) — but the JSON example pairs that same 0.0294 driver with a `percent_delta` of 0.08, an unexplained +0.0506 remainder with `composition_adjustment: 0.0`.
  - Rounding cannot rescue it: 0.0294 rounds to 0.03, not 0.08.
- **Proposed Fix:** Make the `delta` block reconcile. Either (a) set `percent_delta: 0.0294` to match the single 0.0294 driver and `composition_adjustment: 0.0`; or (b) if 0.08 is the intended total, list the full driver set whose points (plus any composition_adjustment) sum to 0.08 at full precision. Cross-check that `band_change.from/to` is consistent with the chosen `percent_delta` against the band thresholds.

### LOW

#### ADV-P1GF-P08-LOW-001: VP-085 method classification drifts between verification-architecture (proptest) and coverage-matrix / purity-boundary-map (CI schema validation)
- **Severity:** LOW
- **Category:** spec-fidelity
- **Location:** `architecture/verification-architecture.md` (Property-Test Layer) vs `architecture/verification-coverage-matrix.md` (op-cli row) and `behavioral-contracts/ss-06/BC-6.06.004.md` (VP-085)
- **Description:** verification-architecture.md "Property-Test Layer (proptest)" lists "JSON schema (VP-085)" as a proptest round-trip. But the verification-coverage-matrix places VP-080..086 under op-cli "CI gates" with the note "thin shell — everything here is an integration/CI gate", purity-boundary-map maps op-cli::json to "schema validation (VP-085)", and BC-6.06.004 declares VP-085 Proof Method = "CI schema validation". Three of four sources say CI schema validation; verification-architecture says proptest. The owning BC (BC-6.06.004) is authoritative and says CI.
- **Evidence:**
  - verification-architecture.md Property-Test Layer: "Round-trips: … JSON schema (VP-085)".
  - BC-6.06.004 VP table: "VP-085 | All JSON outputs validate against the published JSON Schema | **CI schema validation**".
  - verification-coverage-matrix op-cli row: "VP-080, 081, 082, 083, 084, 085, 086 | … thin shell — everything here is an integration/CI gate".
- **Proposed Fix:** Remove VP-085 from the verification-architecture proptest round-trip list (or recharacterize it as "CI schema-validation gate"), aligning with the owning BC. If a proptest *round-trip* on the JSON codec is genuinely intended in addition to the CI schema-validation gate, give it a distinct VP id in the SS-06 reserved range rather than overloading VP-085, and add a coverage-matrix row for it.

## Summary

| Severity | Count |
|----------|-------|
| CRITICAL | 0 |
| HIGH | 1 |
| MEDIUM | 1 |
| LOW | 1 |

**Overall Assessment:** pass-with-findings
**Convergence:** findings remain — iterate
**Readiness:** requires revision (HIGH-001 must resolve before story decomposition — it would seed a contradictory JSON golden fixture for the consistency-validator oracle and the test-writer)

## Novelty Assessment

| Field | Value |
|-------|-------|
| **Pass** | 8 |
| **New findings** | 3 |
| **Duplicate/variant findings** | 0 |
| **Novelty score** | 1.0 |
| **Median severity** | MEDIUM (2.0 on 1=LOW..4=CRIT) |
| **Trajectory** | 10 → 6 → 3 → 3 → 3 → 4 → 3 → 3 |
| **Verdict** | FINDINGS_REMAIN |

### Trajectory Monotonicity Note

Count 3 → 3 (held, no regression). Severity ceiling HIGH → HIGH (held). All three pass-7 findings are
verified RESOLVED at 66f4478; the three new findings are pre-existing latent defects in the JSON-schema
example (same artifact region pass-4 touched at the band/percent layer but did not cross-check at the
denominator and delta-reconciliation layers). This is the expected fresh-context-compounding effect, not
a bad-fix regression — input-hash on interface-definitions.md is unchanged and none of the pass-7 fixes
touched the JSON example block. **Clean-pass counter = 0** (this pass is not clean). Minimum 3 consecutive
CLEAN passes still required to converge; at least 3 more passes after remediation of HIGH-001/MED-001/LOW-001.
