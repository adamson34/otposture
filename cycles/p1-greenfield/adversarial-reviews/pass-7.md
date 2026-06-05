---
document_type: adversarial-review
level: ops
version: "1.0"
status: complete
producer: adversary
timestamp: 2026-06-05T15:30:00
phase: 1d
inputs: [product-brief.md, "domain-spec/", prd.md, "prd-supplements/", "behavioral-contracts/", "architecture/", "verification-properties/"]
input-hash: "1ccddf836bc64b23fe870d3dfd9df37d"
traces_to: prd.md
pass: 7
previous_review: pass-6.md
---

# Adversarial Review: otposture (Pass 7)

Scope: **full**. Fresh-context review of the complete Phase 1 spec package — product-brief.md,
domain-spec/ (10 sections + L2-INDEX), prd.md, prd-supplements/ (4), behavioral-contracts/
(BC-INDEX + 33 BCs), **architecture/ (ARCH-INDEX + 8 sections + 8 ADRs)**, and
**verification-properties/ (VP-INDEX + 10 standalone formal VP files)**. Cycle: p1-greenfield.
Reviewed at commit **f418093** (architecture layer added after pass-6 remediation at 5ca4b17).

> **Scope expansion note:** the architecture/ and verification-properties/ layers entered the
> corpus at commit f418093 — *after* pass-6 was authored against c71c6cf. This is the FIRST
> adversarial pass to see them. Per the skill's "Pre-validate New Scope Additions" lesson, newly
> added scope reliably introduces findings because it was not produced against the converged
> invariant set; all three findings below live in this new layer. Trajectory monotonicity must be
> read against this: the count rise is a scope addition, not a fix regression (see Trajectory note).

> **Fresh-context note:** this pass re-derived understanding from the artifacts directly. Prior-pass
> finding bodies (pass-1..pass-6.md) were NOT read. Only the pass INDEX files were consulted to
> determine the next pass number and to identify which pass-6 fixes to verify, as required by the
> persistence/verification protocol.

## Finding ID Convention

`ADV-P1GF-P07-<SEV>-<SEQ>` (cycle prefix P1GF = p1-greenfield).

## Part A — Fix Verification (pass-6 remediations)

| ID | Previous Severity | Status | Notes |
|----|-------------------|--------|-------|
| ADV-P1GF-P06-HIGH-001 (level-value display annotation attached to `Unknown=0`) | HIGH | **RESOLVED** | DI-012 (invariants.md) and BC-4.04.001 Description now read "NI=0, PI=1/3 (displays as 0.33), LI=2/3 (displays as 0.67), FI=1, Unknown=0". The "displays as" parenthetical now modifies PI/LI only; nothing non-zero is attached to Unknown. Both source-of-truth files coherent. No third occurrence found. |
| ADV-P1GF-P06-MED-001 (events.md implicit snapshots vs PRD OQ5) | MEDIUM | **RESOLVED** | events.md SnapshotTaken row now reads "`snapshot` command only — never implicit (PRD OQ5 decision: `report` warns on a dirty Assessment instead of auto-snapshotting)", matching PRD §Open-Questions item 5 ("no implicit snapshots; `report` warns"). Aligned. |
| ADV-P1GF-P06-MED-002 (PRD "10 commands" vs 11-command surface) | MEDIUM | **RESOLVED** | PRD §3 now says "11 commands"; BC-6.06.001, interface-definitions, ARCH api-surface.md, and module-decomposition.md all consistently enumerate 11 (`init, assess, answer, override, snapshot, status, diff, report, verify, migrate, upgrade`). |
| ADV-P1GF-P06-LOW-001 (E-VAL-001 gap undocumented) | LOW | **RESOLVED** | error-taxonomy.md now carries an explicit row: "E-VAL-001 — Reserved (unallocated; kept vacant to avoid renumbering — do not assign)". The gap is now intentional and documented. |

All four pass-6 remediations confirmed **RESOLVED** with full propagation. No fix-induced regressions
detected in the pre-existing BC/domain/PRD layer.

## Part B — New Findings

All three new findings are in the **newly added architecture + verification-properties layer**
(first review exposure). The previously-converged BC/domain/PRD/supplement corpus produced **zero**
new findings this pass.

### CRITICAL

None.

### HIGH

#### ADV-P1GF-P07-HIGH-001: VP total miscounted as 70 in three places; actual BC-defined count is 67 (self-contradicting within the coverage matrix)

- **Severity:** HIGH
- **Confidence:** HIGH
- **Category:** spec-fidelity / contradictions
- **Location:** `architecture/verification-coverage-matrix.md` line 32 (Coverage Accounting);
  `verification-properties/VP-INDEX.md` line 3 (sharding-model header) and line 39 (Status summary table).
- **Description:** Three artifacts in the new layer assert the project defines **70 VPs**. The actual
  count of distinct VP IDs defined across all 33 BCs is **67**. The discrepancy is not a hidden gap —
  it is internally self-refuting: the coverage matrix's own range enumeration on the same line
  (`001–008, 010–027, 030–036, 040–059, 070–076, 080–086`) sums to exactly **67**, yet the prose
  preceding it on the same line says "Total VPs: **70**". The matrix BODY (per-module rows) also
  enumerates exactly 67. VP-INDEX repeats "all 70 VPs" in its header and reports "draft (harness not
  yet written): 70" in its status table — a count that can never reconcile against the 67 VPs that
  actually exist.
- **Evidence (range arithmetic):**
  - VP-001..008 = 8 · VP-010..027 = 18 · VP-030..036 = 7 · VP-040..059 = 20 · VP-070..076 = 7 · VP-080..086 = 7
  - Sum = 8 + 18 + 7 + 20 + 7 + 7 = **67** (verified by enumerating every `VP-0NN` token defined in
    `behavioral-contracts/`: exactly 67 unique IDs, no gaps within the listed ranges, no extras).
  - `verification-coverage-matrix.md:32`: "**Total VPs:** 70 defined across 33 BCs (ranges: 001–008,
    010–027, 030–036, 040–059, 070–076, 080–086)." — the listed ranges total 67, contradicting the "70".
  - `VP-INDEX.md:3`: "all 70 VPs are *defined* in their owning BCs".
  - `VP-INDEX.md:39`: status table "draft (harness not yet written) | 70".
  - module-criticality.md independently and correctly totals to 67 across its per-module VP-count column
    (8 catalog + 18 store + 20 scoring + 7 assessment + 7 reporting + 7 cli), making the "70" the outlier.
- **Why HIGH (not MED):** (1) It is a hard internal contradiction *within a single line* of the
  authoritative coverage matrix — the prose total disagrees with the ranges next to it. (2) The matrix
  declares itself the enforcement source: line 33 says "The consistency-validator should fail any future
  VP added to a BC without a row here." A coverage-accounting artifact whose own headline total is wrong
  by 3 cannot be trusted as the validator's oracle, and a downstream consistency-validator seeded with
  "70" would either flag 3 phantom-missing VPs forever or mask a real future gap. (3) The defect spans
  3 locations across 2 files (Partial-Fix Regression Discipline axis ⇒ HIGH). (4) A prior pass (pass-2
  index) had already established 67 as the correct count for the BC layer; the new architecture/VP-INDEX
  layer regressed it to 70.
- **Proposed Fix:** Read the authoritative source (the 33 BCs) and set the count to **67** in all three
  locations. Do not patch only the prose — re-derive: `verification-coverage-matrix.md:32` → "Total VPs:
  67"; `VP-INDEX.md:3` → "all 67 VPs"; `VP-INDEX.md:39` status row → "67". Then grep the whole
  `specs/` tree for any other "70" VP reference to prevent recurrence. (Note: line 34 of the matrix also
  states "10 formal proofs + **19** property suites" for the two CRITICAL modules — op-score has 20 VPs
  and op-store has 18; "19 property suites" is itself an unexplained figure and should be recomputed and
  made consistent in the same edit.)

### MEDIUM

#### ADV-P1GF-P07-MED-001: Per-crate cargo-deny I/O ban is unsatisfiable for op-render and op-assess — both depend on the I/O-bearing op-store crate

- **Severity:** MEDIUM
- **Confidence:** HIGH
- **Category:** architecture-contradiction / verification-gaps
- **Location:** `architecture/purity-boundary-map.md` line 43 (CI-enforced rules);
  `architecture/dependency-graph.md` lines 43–44 (edge list); `architecture/decisions/ADR-001-workspace-pure-core.md`
  (Context + Decision); `architecture/module-decomposition.md` (op-render `disclosures(score, store: &LoadedStore)`).
- **Description:** purity-boundary-map.md line 43 states the enforcement rule: "op-score, op-store::codec,
  op-assess (minus Prompt), **op-render**: `#![forbid(unsafe_code)]` + **no I/O-capable crates in their
  dependency trees (cargo-deny per-crate policy)**." But the dependency-graph edge list (lines 43–44)
  shows **op-assess depends on `op-score, op-store`** and **op-render depends on `op-score, op-store`**,
  and module-decomposition gives op-render the signature `disclosures(score, store: &LoadedStore)` — i.e.
  op-render must link op-store to obtain its `LoadedStore` type. `op-store` is a single crate (workspace
  shape in system-overview.md shows one `op-store/` directory) whose `fs` submodule is the effectful
  shell: it pulls in `fs2` (advisory locks), fsync, and rename. cargo-deny operates at **crate**
  granularity, not submodule granularity (ADR-001 explicitly chose crate-level enforcement: "purity
  enforced by `cargo metadata`/cargo-deny; crate granularity matches the SS-01..06 BC structure"). Therefore
  a per-crate "no I/O-capable crates in the dependency tree" ban on op-render and op-assess is
  **unsatisfiable** — op-store (with `fs2`/filesystem I/O) is unavoidably in their dependency trees.
- **Evidence:**
  - purity-boundary-map.md:43 — the CI rule extends the dependency-tree I/O ban to op-render and op-assess.
  - dependency-graph.md edge list — op-assess → {op-score, op-store}; op-render → {op-score, op-store}; op-store external deps include `fs2 (lock)`.
  - module-decomposition.md — op-render `disclosures(score, store: &LoadedStore)` and "diff display"/"status view model" consume op-store load results.
  - system-overview.md workspace shape — a single `op-store/` crate ("codec (pure) + fs shell (effectful)"), not two crates.
  - Contrast: VP-051 in verification-coverage-matrix.md is correctly scoped to **op-score only** ("purity lint"); the over-broad rule is the prose at purity-boundary-map.md:43, which contradicts how the matrix actually assigns the purity gate.
- **Why MEDIUM (not HIGH):** The pure-core proof target that actually matters (op-score, the Kani target,
  NFR-009/VP-051) is correctly isolated at the dependency root with zero workspace deps — that boundary is
  sound. This finding is about an *over-claimed* CI enforcement rule on two secondary crates, not a break
  in the load-bearing scoring-core purity. But it is a real contradiction that will surface at toolchain
  provisioning: an implementer wiring cargo-deny per the purity map will write a per-crate I/O ban for
  op-render/op-assess that cannot pass, then either silently drop it (eroding the documented guarantee) or
  block the build.
- **Proposed Fix:** Decide the intended granularity and make the three artifacts agree. Two coherent options:
  (a) **Submodule discipline (no crate split):** restate purity-boundary-map.md:43 so the I/O ban applies
  to op-score (whole crate) and to the *pure submodules* op-store::codec, op-assess::step, op-render via a
  **module-level lint / code-audit gate** (clippy `disallowed-methods` / import audit), explicitly NOT a
  cargo-deny dependency-tree ban — and note that op-render/op-assess legitimately depend on op-store for
  its types. (b) **Crate split:** extract op-store's `codec` (pure) into a separate `op-store-codec` crate
  that op-render/op-assess depend on instead of the full op-store, restoring a clean per-crate cargo-deny
  ban — but this contradicts ADR-001's explicit "one crate per subsystem" decision and would need an ADR
  amendment. Recommend (a). Whichever is chosen, reconcile dependency-graph.md, purity-boundary-map.md,
  and the cargo-deny config description in tooling-selection.md so "per-crate I/O ban" is claimed only
  where it is actually achievable (op-score), and VP-051's op-score-only scope is preserved.

### LOW

#### ADV-P1GF-P07-LOW-001: BC `Architecture Module` traceability fields left as "(filled by architect)" after the architecture layer landed

- **Severity:** LOW
- **Confidence:** HIGH
- **Category:** spec-fidelity / completeness
- **Location:** Every BC's Traceability table, e.g. `behavioral-contracts/ss-04/BC-4.04.001.md`
  ("Architecture Module | (filled by architect)") and the same placeholder across all 33 BCs; also the
  per-BC "Stories | (filled by story-writer)" rows (the latter is legitimately still pending Phase 2).
- **Description:** The architecture layer (module-decomposition.md) now authoritatively maps every BC to
  an owning crate (BC-1.01.001..004 → op-catalog; BC-2.02.* → op-store; BC-3.03.* → op-assess; BC-4.04.* →
  op-score; BC-5.05.001..003 → op-render; BC-5.05.004/6.06.* → op-cli). But each BC's own Traceability
  table still carries the literal placeholder "Architecture Module | (filled by architect)". The mapping
  exists; it just was not back-filled into the BCs after the architect produced it at commit f418093.
- **Why LOW:** The authoritative BC→module mapping is present and unambiguous in module-decomposition.md
  (and the ARCH-INDEX Subsystem Registry). The placeholder is a stale-cross-reference cosmetic gap, not a
  contradiction or a missing decision. It becomes a paper-cut for an implementer who reads a single BC in
  isolation and finds no architecture pointer.
- **Proposed Fix:** Back-fill the "Architecture Module" row in each BC from the module-decomposition
  BC→Module table (one crate name per BC). The "Stories" placeholder is correctly deferred to Phase 2 and
  should be left as-is. Optionally treat this as a Phase-2 decomposition prerequisite rather than a
  Phase-1d blocker.

## Convergence Status

- **Verdict:** FINDINGS_REMAIN
- **Novelty score:** 1.0 (3 new / 0 duplicate) — all three findings are in the architecture +
  verification-properties layer, which had never been adversarially reviewed before this pass.
- **Trajectory:** 10 → 6 → 3 → 3 → 3 → 4 → 3
- **Severity ceiling:** CRIT → HIGH → HIGH → MED → MED → HIGH → HIGH
- **Clean-pass counter:** 0 (this pass is NOT clean — minimum 3 consecutive clean passes not yet begun).

### Trajectory Monotonicity Note

Count moved 4 → 3 (decreasing) and the previously-converged BC/domain/PRD/supplement corpus produced
**zero** new findings — the four pass-6 fixes are all verified RESOLVED with no regression. However,
the trajectory must be read with the **scope expansion** in mind: commit f418093 added the entire
architecture/ (8 sections + 8 ADRs) and verification-properties/ (VP-INDEX + 10 files) layer AFTER
pass-6. This pass is the first adversarial exposure of that layer, and all 3 findings originate there.
Per the skill's "Pre-validate New Scope Additions" lesson, this is the expected effect of un-converged
scope entering the corpus (each new artifact set reliably yields findings), not a bad-fix regression.
The pre-existing corpus is converging cleanly; the new layer must now go through its own remediation
and clean-pass cycle.

### Recommended Next Steps

1. Remediate HIGH-001 by **re-deriving** the VP count from the 33 BCs (= 67) and correcting all three
   locations plus the "19 property suites" figure — do not patch only the prose total (root-cause fix per
   the Fix-Root-Causes lesson; a prior pass already established 67 as authoritative).
2. Remediate MED-001 by reconciling purity-boundary-map.md:43, dependency-graph.md, and
   tooling-selection.md so the per-crate cargo-deny I/O ban is claimed only for op-score (recommend
   option (a), submodule-lint discipline, to avoid an ADR-001 amendment).
3. Back-fill LOW-001 BC→module traceability rows (or formally defer to Phase 2).
4. Re-run the adversary on the full corpus (including the architecture/VP layer) until **3 consecutive
   clean passes** are achieved — the clean-pass counter is still 0.
