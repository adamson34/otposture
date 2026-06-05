---
document_type: adversarial-review
level: ops
version: "1.0"
status: complete
producer: adversary
timestamp: 2026-06-05T14:50:00
phase: 1d
inputs: [product-brief.md, "domain-spec/", prd.md, "behavioral-contracts/", "prd-supplements/"]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: prd.md
pass: 5
previous_review: pass-4.md
---

# Adversarial Review: otposture (Pass 5)

Scope: full. Fresh-context review of the complete Phase 1 spec package — product-brief.md,
domain-spec/ (10 sections + L2-INDEX), prd.md, prd-supplements/ (4), behavioral-contracts/
(BC-INDEX + 33 BCs). Cycle: p1-greenfield. Reviewed at commit bd2ec6a (pass-4 remediation).

## Finding ID Convention

`ADV-P1GF-P05-<SEV>-<SEQ>` (cycle prefix P1GF = p1-greenfield).

## Part A — Fix Verification (pass-4 remediations)

| ID | Previous Severity | Status | Notes |
|----|-------------------|--------|-------|
| Pass-4: coherent JSON example arithmetic | MEDIUM | RESOLVED | interface-definitions.md JSON example is internally coherent: percent 0.6765 ∈ [.65,.85) ⇒ band_uncapped `managed`, floor-capped to `developing`; driver points 0.0294 = 1/34 under 34 equal weights. assessed(30)+unknown(4)=total(34) with na_count⊂assessed is a consistent reading. |
| Pass-4: `potential_gain` field name | MEDIUM | RESOLVED | `potential_gain` used consistently in BC-4.04.008 (postcondition 2/3, tie-break, VP-058) and interface-definitions.md `next_actions`. No `potential_score_gain`/other variants remain. |
| Pass-4: L2 decision hedge (ASM-006) | LOW/MED | RESOLVED | BC-4.04.003 description carries the "(ASM-006 hedge)" band-rename note; L2-INDEX Key Domain Decision 2 and DI-008 align (band names are catalog data, floor-cap positional). |
| Pass-4: override / floor-gap example contradiction | MEDIUM | RESOLVED | DEC-005, BC-4.04.004 EC-002, BC-3.03.004 EC-001, BC-5.05.001 EC-002, BC-5.05.002 EC-002 now consistently state: override may lift the cap, disclosure mandatory, prior snapshots keep their capped band (events.md rule 4). No contradiction found. |

All four pass-4 fixes verified landed and correct.

## Part B — New Findings

### MEDIUM

#### ADV-P1GF-P05-MED-001: `verify` reports `ok: false` but exits 0 — defeats its stated CI-gate purpose
- **Severity:** MEDIUM
- **Category:** interface-gaps
- **Location:** prd-supplements/interface-definitions.md:95-107 (verify-results JSON, `"ok": false`); prd-supplements/error-taxonomy.md:47,49 (E-STORE-003/E-STORE-005 = `degraded` → exit 0); behavioral-contracts/ss-06/BC-6.06.001.md inv. 2; behavioral-contracts/ss-06/BC-6.06.004.md postcondition 2 / BC-2.02.004 postcondition 3
- **Description:** The `verify` command's primary documented consumers are "CI gates" (BC-6.06.004 description: "for scripting (CI gates...)"). When `verify` finds cached-score mismatches (E-STORE-003) or quarantined orphans (E-STORE-005), it emits `"ok": false` in JSON — but both error codes are classified `degraded` → **exit 0**, and BC-6.06.001 invariant 2 states "Warnings never change a success exit code." No spec text anywhere gives `verify` a non-zero exit when it finds problems. Consequence: a CI gate keyed on the **exit code** (the idiomatic shell/CI pattern, and BC-6.06.001 explicitly says "scripts and CI depend on" exit codes) treats a tampered/drifted store as a pass, while a CI gate parsing the `ok` field sees a failure. The two machine-consumption paths contradict each other for the exact use case the command exists to serve.
- **Evidence:**
  - interface-definitions.md:105 — `"ok": false` (because score_mismatches is non-empty)
  - error-taxonomy.md:47 — `E-STORE-003 | degraded | 0 (warning via verify) | Cached score mismatch`
  - error-taxonomy.md:49 — `E-STORE-005 | degraded | 0 (warning)`
  - BC-6.06.001 inv. 2 — "Warnings never change a success exit code (severity `degraded` ⇒ exit 0 + stderr warning)"
  - BC-6.06.001 description — "Exit codes are stable, documented contract (scripts and CI depend on them)"
  - Confirmed absence: grep for any non-zero exit spec for `verify` returns nothing.
- **Proposed Fix:** Decide and specify `verify`'s exit semantics explicitly. Recommended: give `verify` a dedicated non-zero exit when `ok == false` (e.g., reuse exit class 4 store-integrity, or introduce a documented "verify-failed" exit code), and carve a documented exception to BC-6.06.001 inv. 2 for `verify` (parallel to the existing `--json` exception in inv. 3). Update BC-6.06.004, BC-2.02.004, BC-6.06.001, and error-taxonomy.md so the exit code and the `ok` field agree. If the deliberate decision is "verify always exits 0, gate on `ok`," state that explicitly in BC-6.06.004 and warn against exit-code gating — but that contradicts BC-6.06.001's "CI depends on exit codes," so the exception must be written down either way.

### LOW

#### ADV-P1GF-P05-LOW-001: Scoring-core Kani enumeration omits VP-056, which a BC marks `kani` and which lives in the SS-04 scoring subsystem
- **Severity:** LOW
- **Category:** verification-gaps
- **Location:** prd-supplements/nfr-catalog.md:29 (NFR-008 Kani list); prd.md:87 (§7 Kani list); behavioral-contracts/ss-04/BC-4.04.007.md (VP-056, proof method `kani/proptest`)
- **Description:** NFR-008 and PRD §7 both enumerate the formal-verification (Kani) targets as exactly "VP-040/041/044/046/047/048/049/050/053," scoped to the scoring core. However VP-056 (comparability-guard soundness, BC-4.04.007) is in the **same SS-04 scoring subsystem** and is specified with proof method `kani/proptest` in its BC, yet it is absent from the authoritative Kani enumeration. The enumeration is presented as the complete Kani target list (PRD §7: "The scoring core is the formal-verification target (Kani on VP-...)"), so a downstream formal-verifier working from NFR-008/PRD will not prove VP-056, while a verifier working from the BC will. (VP-004 catalog and VP-015 store are also `kani/proptest` but sit outside the scoring core, so their omission from a scoring-core-scoped list is defensible; VP-056 is the one true mismatch because it is in-scope.)
- **Evidence:**
  - nfr-catalog.md:29 — "Kani proofs on VP-040/041/044/046/047/048/049/050/053"
  - prd.md:87 — "(Kani on VP-040/041/044/046/047/048/049/050/053)"
  - BC-4.04.007 VP-056 — "Guard soundness: any pair it admits shares an effective catalog digest | kani/proptest"
- **Proposed Fix:** Either add VP-056 to the NFR-008 and PRD §7 Kani enumerations (if comparability-guard soundness is intended to be Kani-proven), or downgrade VP-056's proof method in BC-4.04.007 to `proptest` (if Kani on the guard is not intended). Make the BC proof method and the project-level Kani list agree.

#### ADV-P1GF-P05-LOW-002: ASM-002/ASM-005 call the CISA third rating "complexity" while every downstream artifact calls it "Ease"
- **Severity:** LOW
- **Category:** ambiguous-language
- **Location:** domain-spec/assumptions.md:22 (ASM-002 "cost/impact/complexity"), :25 (ASM-005 "cost/impact/complexity"); vs. BC-1.01.004:39, BC-4.04.008 (`effort_rank` from "Ease" Simple/Moderate/Complex), interface-definitions.md:149 (`ease:`), prd.md:98 ("Cost/Impact/Ease")
- **Description:** The CISA per-goal third metadata axis is named inconsistently across the spec corpus. ASM-002 and ASM-005 (assumption bodies) call it "complexity," but their own validation notes and every L3 consumer (BC-1.01.004, BC-4.04.008 effort_rank, interface-definitions catalog `ease` field, PRD assumptions line) call it "Ease" with values Simple/Moderate/Complex. "Complexity" and "Ease" are inverse framings of the same axis, so a catalog author or implementer could misread the polarity (high complexity vs. high ease) when wiring `effort_rank` (Simple=1 … Complex=3). Not a contradiction in the scoring math (which is pinned to Ease/Simple=1), but a naming-polarity hazard.
- **Evidence:**
  - assumptions.md:22 — "per-goal cost/impact/**complexity** ratings"; validation note in the same row says "**Cost/Impact/Ease** ratings"
  - assumptions.md:25 — "cost/impact/**complexity** metadata"
  - BC-4.04.008 postcondition 2 — "`effort_rank` from catalog **Ease** metadata (Simple=1, Moderate=2, Complex=3)"
  - interface-definitions.md:149 — "`ease: simple   # CISA rating`"
- **Proposed Fix:** Standardize on "Ease" (the CISA rating's published name, per the ASM-002 validation note) throughout ASM-002 and ASM-005 bodies; or, if "complexity" is retained anywhere, state the polarity mapping explicitly (Ease Simple ⇔ low complexity) so `effort_rank` cannot be wired inverted.

## Summary

| Severity | Count |
|----------|-------|
| CRITICAL | 0 |
| HIGH | 0 |
| MEDIUM | 1 |
| LOW | 2 |

**Overall Assessment:** pass-with-findings
**Convergence:** findings remain — iterate (clean-pass counter stays at 0)
**Readiness:** requires revision (one MEDIUM contract gap on `verify` exit semantics)

The package is high quality and internally coherent; SS-04 (scoring) and SS-02 (store) are
rigorous and the pass-4 fixes are all confirmed. The remaining findings are a real
machine-interface contract gap (verify exit code vs. `ok`) and two low-severity
consistency items. Severity ceiling fell from MEDIUM (pass 4) to MEDIUM (pass 5), with the
single MEDIUM being a genuinely new issue not raised in prior passes.

## Novelty Assessment

| Field | Value |
|-------|-------|
| **Pass** | 5 |
| **New findings** | 3 |
| **Duplicate/variant findings** | 0 |
| **Novelty score** | 1.0 (3 / (3 + 0)) |
| **Median severity** | 2.0 (LOW=1, MED=2, LOW=1 → median LOW–MED boundary; reported as 2.0) |
| **Trajectory** | 10 → 6 → 3 → 3 → 3 |
| **Verdict** | FINDINGS_REMAIN |

Note: trajectory holds flat at 3 (not a regression — monotonicity preserved: 3 ≤ 3).
All 3 findings are genuinely new (novelty 1.0), none are restatements of pass 1–4
findings. This is NOT a clean pass; the minimum 3 consecutive clean passes has not begun.
