---
document_type: adversarial-review
level: ops
version: "1.0"
status: complete
producer: adversary
timestamp: 2026-06-05T15:05:00
phase: 1d
inputs: [product-brief.md, "domain-spec/", prd.md, "behavioral-contracts/", "prd-supplements/"]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: prd.md
pass: 6
previous_review: pass-5.md
---

# Adversarial Review: otposture (Pass 6)

Scope: **full**. Fresh-context review of the complete Phase 1 spec package — product-brief.md,
domain-spec/ (10 sections + L2-INDEX), prd.md, prd-supplements/ (4), behavioral-contracts/
(BC-INDEX + 33 BCs). Cycle: p1-greenfield. Reviewed at commit **c71c6cf** (pass-5 remediation).

> Fresh-context note: this pass re-derived understanding from the artifacts directly. Prior-pass
> finding bodies (pass-1..pass-5.md) were NOT read; only the pass-5 INDEX was consulted to
> identify which fixes to verify, as required by the persistence/verification protocol.

## Finding ID Convention

`ADV-P1GF-P06-<SEV>-<SEQ>` (cycle prefix P1GF = p1-greenfield).

## Part A — Fix Verification (pass-5 remediations)

| ID | Previous Severity | Status | Notes |
|----|-------------------|--------|-------|
| ADV-P1GF-P05-MED-001 (`verify` exit semantics) | MEDIUM | **RESOLVED** | BC-6.06.001 Inv 2 now documents the single `verify` exception: exit 0 only when `ok: true`, exit 4 when score mismatches / quarantined orphans / integrity errors are reported (CI-gate semantics). Fix fully propagated to error-taxonomy E-STORE-003 and E-STORE-005 ("0 as load-time warning; **4 when reported by `verify`**") and to interface-definitions verify block ("exit 0 = all checks pass; exit 4 = findings reported — CI-gate semantics, BC-6.06.001"). Three-location coherence confirmed. |
| ADV-P1GF-P05-LOW-001 (VP-056 omitted from Kani list) | LOW | **RESOLVED** | VP-056 now appears in NFR-008 Kani list and PRD §7 Kani list. VP-056 is owned by BC-4.04.007 with proof method `kani/proptest`, so its inclusion in the Kani target list is justified. Cross-checked the full Kani list (VP-040/041/044/046/047/048/049/050/053/056): all resolve to BCs declaring kani or kani/proptest. |
| ADV-P1GF-P05-LOW-002 (complexity vs Ease polarity) | LOW | **RESOLVED** | ASM-002 and ASM-005 renamed "Complexity" to "Ease of Implementation" with explicit inverse-polarity warning ("Simple is easiest; `effort_rank` wiring must not invert it"). Downstream wiring verified correct: BC-4.04.008 PC2 maps `effort_rank` Simple=1/Moderate=2/Complex=3 and divides `potential_gain / effort_rank`, so Simple (rank 1) correctly yields highest priority — polarity respected. |

All three pass-5 remediations confirmed RESOLVED with full propagation. **Trajectory monotonicity preserved.**

## Part B — New Findings

### CRITICAL

None.

### HIGH

#### ADV-P1GF-P06-HIGH-001: Level-value display annotation contradicts Unknown=0 in BOTH source-of-truth locations (DI-012 and BC-4.04.001)
- **Severity:** HIGH
- **Confidence:** HIGH
- **Category:** contradictions / ambiguous-language
- **Location:** `domain-spec/invariants.md` DI-012 (line 32); `behavioral-contracts/ss-04/BC-4.04.001.md` Description (line 30)
- **Description:** Both the authoritative domain invariant DI-012 and the percent-computation contract BC-4.04.001 state the level valuation as: `NI=0, PI=1/3, LI=2/3, FI=1, Unknown=0 (displayed as 0.33/0.67)`. The parenthetical "(displayed as 0.33/0.67)" is appended immediately after `Unknown=0`, so the clause literally reads **"Unknown=0 (displayed as 0.33/0.67)"** — asserting that Unknown displays as a non-zero value. This directly contradicts BC-3.03.003 ("Unknown … scored 0") and the formula's own `Unknown=0`. The "0.33/0.67" display pair is the rounded presentation of PI=1/3 and LI=2/3, not of Unknown. The annotation is attached to the wrong level.
- **Evidence:**
  - DI-012: "Level values are **exact rationals**: NI=0, PI=1/3, LI=2/3, FI=1, Unknown=0 (displayed as 0.33/0.67); engine arithmetic is rational/fixed-point…"
  - BC-4.04.001 Description: "NI=0, PI=1/3, LI=2/3, FI=1, Unknown=0 (displayed as 0.33/0.67)"
  - BC-3.03.003 Description: "Goals without an explicit Answer behave as `Unknown` everywhere: scored 0"
  - BC-3.03.003 VP-034: "Unknown and Not Implemented are score-identical but flag-distinct" (i.e., both are 0)
- **Why HIGH (not LOW):** Blast radius = 2 source-of-truth files (DI-012 is the canonical invariant; BC-4.04.001 is the most formally-verified function and the one an implementer/test-writer reads to build the scoring core and its golden test vectors). A test-writer transcribing this literally could encode a wrong Unknown display value. Per the Partial-Fix Regression Discipline axis, a defect propagated across 2+ files in the same layer is HIGH.
- **Proposed Fix:** Reposition the parenthetical so it modifies PI/LI only. E.g.: "NI=0, PI=1/3 (displayed 0.33), LI=2/3 (displayed 0.67), FI=1, Unknown=0". Apply identically to DI-012 and BC-4.04.001 (and grep for any third occurrence). Do not leave the annotation adjacent to `Unknown=0`.

### MEDIUM

#### ADV-P1GF-P06-MED-001: events.md still advertises implicit snapshots that the PRD explicitly decided against
- **Severity:** MEDIUM
- **Confidence:** HIGH
- **Category:** contradictions
- **Location:** `domain-spec/events.md` SnapshotTaken row (line 25); contradicts `prd.md` Open Question 5 (line 106)
- **Description:** The events.md `SnapshotTaken` trigger reads: "`snapshot` command **(or implied by report/diff on dirty state — PRD decision)**". It defers to a "PRD decision," but the PRD has now made that decision the **opposite** way: PRD OQ5 states the "current decision: **no implicit snapshots**; `report` warns if Assessment differs from the latest snapshot." events.md was never updated to reflect the decision it points to, so it still describes auto-snapshot-on-report/diff behavior that the product has rejected.
- **Evidence:**
  - events.md line 25: "SnapshotTaken | `snapshot` command (or implied by report/diff on dirty state — PRD decision)"
  - prd.md line 106: "**`snapshot` auto-take on `report`** … current decision: **no implicit snapshots**; `report` warns if Assessment differs from the latest snapshot."
- **Impact:** An implementer working from events.md (the domain-event/pipeline source) could build implicit snapshot-on-report/diff behavior, then have it rejected against the PRD — or worse, ship it. The two source documents disagree on observable behavior.
- **Proposed Fix:** Update the events.md SnapshotTaken Trigger cell to: "`snapshot` command only (no implicit snapshots; `report`/`diff` on dirty state emit a warning, not a snapshot — PRD OQ5)". Keep the cross-reference but state the resolved decision.

#### ADV-P1GF-P06-MED-002: PRD §3 says "10 commands" but the authoritative surface defines 11
- **Severity:** MEDIUM
- **Confidence:** HIGH
- **Category:** spec-fidelity / contradictions
- **Location:** `prd.md` §3 (line 64); contradicts `prd-supplements/interface-definitions.md` COMMANDS block and `behavioral-contracts/ss-06/BC-6.06.001.md` Description
- **Description:** PRD §3 states: "See `prd-supplements/interface-definitions.md` — CLI surface (**10 commands**)…". The authoritative command surface (BC-6.06.001 is explicitly named "authoritative command surface" by interface-definitions) lists **11** commands: `init, assess, answer, override, snapshot, status, diff, report, verify, migrate, upgrade` (excluding `--help`/`--version`). interface-definitions.md's COMMANDS block enumerates the same 11. The PRD count is off by one against its own cited source.
- **Evidence:**
  - prd.md line 64: "CLI surface (10 commands)"
  - BC-6.06.001 line 30: "init, assess, answer, override, snapshot, status, diff, report, verify, migrate, upgrade, plus --help/--version" → 11 commands
  - interface-definitions.md COMMANDS block: init, assess, answer, override, snapshot, status, diff, report, verify, migrate, upgrade → 11
- **Impact:** Low functional risk (commands are fully enumerated downstream), but the headline PRD count is wrong, which undermines the "transparent / by-the-numbers" credibility posture and is the kind of stale count that recurs across passes if not fixed at the source. Likely `upgrade` (or `verify`) was added to the surface after the PRD line was written.
- **Proposed Fix:** Change PRD §3 to "CLI surface (11 commands)". Verify no other doc hardcodes the old count.

### LOW

#### ADV-P1GF-P06-LOW-001: Error-taxonomy E-code sequence starts at E-VAL-002 (no E-VAL-001) — undocumented gap
- **Severity:** LOW
- **Confidence:** MEDIUM
- **Category:** completeness
- **Location:** `prd-supplements/error-taxonomy.md` Error Catalog (line 36, first VAL row is E-VAL-002)
- **Description:** The VAL error sequence begins at E-VAL-002; there is no E-VAL-001 anywhere in the catalog. Other categories start at their `-001` (E-STORE-001, E-CAT-001, E-LOCK-001, E-FS-001, E-RPT-001, E-SCORE-001). The VAL gap may be intentional (a retired/reserved code) but is undocumented, so a reader cannot tell whether a contract referencing E-VAL-001 was dropped or the code is simply reserved.
- **Evidence:** error-taxonomy.md Error Catalog: first VAL row is "E-VAL-002 | broken | 3 | Site name contains control characters"; no E-VAL-001 row; no reservation note. A `grep` for `E-VAL-001` across specs returns nothing.
- **Proposed Fix:** Either renumber VAL to start at E-VAL-001, or add a one-line note ("E-VAL-001 reserved/retired — see <reason>") so the gap is intentional and auditable. `(pending intent verification)` — adjudicate whether the gap is deliberate.

## Summary

| Severity | Count |
|----------|-------|
| CRITICAL | 0 |
| HIGH | 1 |
| MEDIUM | 2 |
| LOW | 1 |

**Total new findings: 4.**

**Overall Assessment:** pass-with-findings.
**Convergence:** findings remain — iterate. **NOT a clean pass.**
**Readiness:** requires revision before Phase 2.

## Novelty Assessment

| Field | Value |
|-------|-------|
| Pass | 6 |
| New findings | 4 (HIGH-001, MED-001, MED-002, LOW-001) |
| Duplicate/retread findings | 0 |
| Novelty score | 1.0 (4 new / 0 duplicate) |
| Trajectory | 10 → 6 → 3 → 3 → 3 → **4** |
| Severity ceiling | CRIT → HIGH → HIGH → MED → MED → **HIGH** |
| Clean-pass counter | **0** (this pass is not clean) |

### Trajectory Monotonicity ALERT

Finding count rose **3 → 4** and the severity ceiling rose **MED → HIGH** versus pass 5. Per the
adversarial-review monotonicity rule, an increase must be explained before further convergence
passes proceed. **Root-cause analysis of this regression:**

- HIGH-001 (the Unknown-display contradiction in DI-012/BC-4.04.001) is **not a pass-5 fix
  regression** — it is a pre-existing latent defect in source-of-truth artifacts that all five
  prior passes missed. The input-hash on these files (`317fa65…` / `d0490fef…`) is unchanged,
  confirming the text was not introduced by the c71c6cf remediation. This is the expected
  "fresh-context compounding value" effect: a defect in the most-read scoring contract surfaced
  only on an independent re-derivation. It does NOT indicate a bad fix.
- MED-001, MED-002, LOW-001 are likewise pre-existing cross-document drift (events.md↔PRD,
  PRD↔interface, error-taxonomy numbering) not touched by c71c6cf.
- **Conclusion:** the count increase is driven by newly-surfaced latent defects, not by a
  pass-5 fix defect or an unvalidated scope addition. The three pass-5 fixes are all confirmed
  RESOLVED (Part A). Convergence may proceed after these 4 findings are remediated — but the
  clean-pass counter remains at **0** and the minimum of **3 consecutive clean passes** has not
  begun.

Novelty remains HIGH (genuine contradictions with file:line evidence, not wording nitpicks).
The spec has **not** converged.
