---
document_type: adversarial-review
level: ops
version: "1.0"
status: complete
producer: adversary
timestamp: 2026-06-05T13:20:00
phase: 1d
inputs: [product-brief.md, "domain-spec/", prd.md, "prd-supplements/", "behavioral-contracts/"]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: prd.md
pass: 1
previous_review: null
---

# Adversarial Review: otposture Phase 1 Spec Package (Pass 1)

**Scope:** full. Target: specs. Reviewed product-brief.md, domain-spec/ (L2-INDEX + 10
sections), prd.md, prd-supplements/ (4 files), behavioral-contracts/ (BC-INDEX + 32 BCs
across ss-01..ss-06). No `.factory/policies.yaml` present — policy rubric auto-load skipped
(no project policies registered).

## Finding ID Convention

`ADV-P1GREEN-P01-<SEV>-<SEQ>`.

## Part B — New Findings (Pass 1)

### CRITICAL

#### ADV-P1GREEN-P01-CRIT-001: Orphan-answer contract contradiction between assessment entry and store load
- **Severity:** CRITICAL
- **Category:** contradictions
- **Location:** BC-3.03.001 (Invariants 2, VP-030), DEC-011, vs BC-2.02.004 EC-004
- **Description:** Two BCs make opposite guarantees about whether an Answer can reference a
  goal absent from the pinned catalog. BC-3.03.001 Invariant 2 states "An Answer never
  references a goal absent from the pinned catalog (DEC-011)" and VP-030 asserts "Assessment
  never contains an orphan goal_id" (a property to be proven). DEC-011 says answering an
  absent goal is "Rejected with nearest-match suggestion; Assessment never contains orphan
  Answers." But BC-2.02.004 EC-004 says "Unknown goal_id in an Answer (catalog/store drift) →
  Load succeeds with warning; scoring surfaces it per DEC-011/DEC-012 rules." If entry can
  never create an orphan answer and the store is append-only/atomic, the load-time orphan case
  should be impossible — yet EC-004 treats it as a normal recoverable state and even cites
  DEC-011, which is about *rejection at entry*, not tolerated drift. This is load-bearing for
  the scoring core: VP-030 cannot hold if the store legitimately loads orphan answers.
- **Evidence:** BC-3.03.001:46 "An Answer never references a goal absent from the pinned
  catalog (DEC-011)"; BC-3.03.001 VP-030 "Assessment never contains an orphan goal_id";
  BC-2.02.004:54 EC-004 "Load succeeds with warning; scoring surfaces it per DEC-011/DEC-012".
- **Proposed Fix:** Decide the model explicitly: (a) orphan answers are *impossible by entry
  invariant* and a loaded orphan is a corruption/integrity error (E-STORE-002 class), not a
  warning — then rewrite BC-2.02.004 EC-004 to route through corruption handling and stop
  citing DEC-011; OR (b) orphan answers are *tolerable drift* (hand-edited human-readable
  store, catalog swapped) — then weaken BC-3.03.001 VP-030 to "the *answer command* never
  *creates* an orphan" and define how scoring treats a loaded orphan (ignored? counted? this is
  undefined today). Cite the correct DEC in each direction.

### HIGH

#### ADV-P1GREEN-P01-HIGH-001: `store upgrade` is a phantom command outside the fixed command surface
- **Severity:** HIGH
- **Category:** interface-gaps
- **Location:** BC-2.02.004 EC-003, BC-2.02.005 Preconditions, vs BC-6.06.001, interface-definitions.md
- **Description:** BC-2.02.004 EC-003 specifies "on-disk migration only with explicit
  `store upgrade` command" and BC-2.02.005 lists "store-upgrade" among the mutating commands
  that acquire the lock. But BC-6.06.001 declares the v1 command surface "fixed and small" and
  enumerates exactly ten commands plus `--help`/`--version` — `store upgrade` is not among
  them, and it is absent from interface-definitions.md's USAGE block. Either an entire
  command is missing from the authoritative surface, or these two references are stale. This
  also affects VP-080 ("every E-code maps to exactly one exit class; integration tests cover
  each class") and the help-completeness rule (BC-6.06.001 post-2), which would have to cover
  a command that does not exist in the surface spec.
- **Evidence:** BC-2.02.004:53; BC-2.02.005:34 "(assess/snapshot/override/migrate/store-upgrade)";
  BC-6.06.001:30 "`init`, `assess`, `answer`, `override`, `snapshot`, `status`, `diff`,
  `report`, `verify`, `migrate`"; interface-definitions.md:20-46 (no `store upgrade`).
- **Proposed Fix:** Either add `store upgrade` (or fold store-schema upgrade into `migrate`)
  to the command surface in both interface-definitions.md and BC-6.06.001 with its own
  `--help`/exit behavior, or remove the references from BC-2.02.004 EC-003 / BC-2.02.005 and
  specify how an older-schema store is upgraded on disk (e.g., implicitly on first mutation).

#### ADV-P1GREEN-P01-HIGH-002: `assess --only` token set contradicts the interface definition and re-introduces out-of-scope staleness
- **Severity:** HIGH
- **Category:** contradictions
- **Location:** BC-3.03.002 Postcondition 1 & EC-001, vs interface-definitions.md (USAGE + Flag Interactions)
- **Description:** BC-3.03.002 specifies the filter as "`--only unknown|stale` (default
  `unknown`)" plus a separate "`--all`" flag. interface-definitions.md specifies
  "`--only <unknown|all>` (default: unknown)" — i.e., `all` is a *value of* `--only`, not a
  separate flag, and there is no `stale` token. Two contradictions: (1) flag shape (`--all`
  flag vs `--only all` value); (2) the `stale` token. Worse, `stale` depends on per-Goal
  freshness windows, which are CAP-012 "Evidence & staleness (reserved)" — explicitly P2 /
  post-MVP and out of MVP scope (brief Out of Scope; capabilities.md CAP-012). Shipping a
  `--only stale` filter in v1 would require staleness data the v1 schema only *reserves*.
- **Evidence:** BC-3.03.002:38 "`--only unknown|stale` (default `unknown`)... or `--all`";
  BC-3.03.002:51 EC-001 "use `--all` to revisit"; interface-definitions.md:29
  "`--only <unknown|all>`"; interface-definitions.md:140 "`assess --only all`";
  capabilities.md CAP-012 (P2, reserved).
- **Proposed Fix:** Align BC-3.03.002 to interface-definitions.md: `--only <unknown|all>`,
  default `unknown`; drop the separate `--all` flag and the `stale` token from v1. If a
  resume-by-staleness mode is desired, defer it to CAP-012 and note it as a reserved future
  value, not a v1 contract.

#### ADV-P1GREEN-P01-HIGH-003: JSON error object is emitted on stdout, violating the stdout/stderr separation invariant
- **Severity:** HIGH
- **Category:** contradictions
- **Location:** BC-6.06.004 EC-001 & interface-definitions.md, vs BC-6.06.001 Invariant 3
- **Description:** BC-6.06.001 Invariant 3 mandates "Diagnostics go to stderr; data/results to
  stdout (pipeable)." BC-6.06.004 EC-001 specifies that under `--json`, an error such as a
  comparability refusal emits "JSON error object on stdout (`{"error": {...}}`), same exit
  code", and interface-definitions.md:92 reinforces the error shape "(any command with
  `--json`)". An error is a diagnostic; routing it to stdout directly contradicts the
  separation invariant and will confuse pipelines that parse stdout as the success document
  (a consumer doing `status --json | jq .score` would silently get an error object). The spec
  does not state which stream wins.
- **Evidence:** BC-6.06.001:46 Inv 3; BC-6.06.004:51 EC-001 "JSON error object on stdout";
  interface-definitions.md:92.
- **Proposed Fix:** Pick one and make it explicit across all three docs. Recommended: JSON
  error objects go to **stderr** (preserving Inv 3); stdout stays the success-document channel
  and is empty on error. If product intent is genuinely "machine consumers read one stream,"
  carve an explicit, documented exception into BC-6.06.001 Inv 3 for `--json` error objects so
  the contradiction is resolved rather than latent.

### MEDIUM

#### ADV-P1GREEN-P01-MED-001: Next-Best-Actions ranking mandates `impact` metadata it never consumes
- **Severity:** MEDIUM
- **Category:** verification-gaps
- **Location:** BC-4.04.008 Postcondition 2, BC-1.01.002 rule (h), CAP-009, ubiquitous-language.md
- **Description:** CAP-009 and ubiquitous-language.md define a Next Best Action as ranked by
  "potential score impact ÷ catalog effort rating." BC-4.04.008's actual formula is
  `priority = potential_gain / effort_rank`, where `potential_gain` is *recomputed* via the
  real scoring formula and `effort_rank` comes from catalog Ease metadata. The catalog's
  `impact` rating (CISA Impact) is therefore never used in the ranking — yet BC-1.01.002 rule
  (h) makes "impact/effort metadata present for every goal" a *hard catalog-validation
  requirement* ("CAP-009 dependency"), and ASM-005 frames impact metadata as seeding NBA
  ranking. The result: a catalog is rejected at load for missing `impact` data that the engine
  does not read. Either the validation rule is over-strict, or the ranking is supposed to use
  catalog impact and the formula is wrong.
- **Evidence:** BC-4.04.008:39 "priority = potential_gain / effort_rank"; BC-1.01.002:40 rule
  (h) "impact/effort metadata present for every goal (CAP-009 dependency)"; CAP-009
  "potential score impact ÷ catalog effort metadata"; ASM-005.
- **Proposed Fix:** Decide whether NBA uses (a) computed `potential_gain` only (then drop
  `impact` from the mandatory validation rule (h), or downgrade it to optional/advisory and
  update ASM-005), or (b) catalog `impact` as a factor (then fix BC-4.04.008's formula and its
  VP-058 statement, which currently ties potential_gain directly to realized percent gain).

#### ADV-P1GREEN-P01-MED-002: `migrate --series-break` confirmation semantics undefined for the non-interactive path
- **Severity:** MEDIUM
- **Category:** ambiguous-language
- **Location:** BC-2.02.006 EC-003, BC-2.02.007 Invariant 2, interface-definitions.md migrate flags
- **Description:** BC-2.02.007 Invariant 2: "Creating a break requires explicit confirmation in
  interactive use, or an explicit flag in scripted use — never implicit." interface-definitions
  lists `migrate --series-break` as the "explicit opt-in when unmappable" and makes
  `--mapping` and `--series-break` mutually exclusive. But BC-2.02.006 EC-003 (no mapping table
  available) says migration "records a Series Break instead ... after explicit user
  confirmation" — describing only the interactive path. The contract does not state what
  happens when `migrate --catalog X` is run non-interactively with neither `--mapping` nor
  `--series-break` and no mapping is shippable: does it error (exit 2/6), or auto-break? Inv 2
  forbids implicit breaks, so it must error, but no E-code or behavior is specified for this
  case. Scripts (the persona's CI) will hit this.
- **Evidence:** BC-2.02.007:45 Inv 2; BC-2.02.006:54 EC-003; interface-definitions.md:44,139.
- **Proposed Fix:** Specify the non-interactive no-mapping-no-flag outcome explicitly: refuse
  with a named E-code (e.g., E-CAT/E-VAL) instructing the user to pass `--series-break` or
  `--mapping`. Add it as an edge case/test vector in BC-2.02.006 or BC-2.02.007.

#### ADV-P1GREEN-P01-MED-003: Goal-ID splitting (`3.Q.a`) collides with case-insensitive-unique ID rule and the verbatim-ID golden test
- **Severity:** MEDIUM
- **Category:** contradictions
- **Location:** BC-1.01.004 EC-002, BC-2.02.006 EC-001, vs BC-1.01.002 EC-002 & VP-008
- **Description:** BC-1.01.004 EC-002 and BC-2.02.006 EC-001 allow a compound goal to be
  *split* into sub-items under the official ID ("3.Q.a/3.Q.b"). But BC-1.01.004 VP-008 requires
  "Goal IDs exactly match the official CPG 2.0 ID set" (golden test), and the bundled catalog
  postcondition fixes exactly 34 goals with IDs 1.A..6.A. A split introduces IDs (`3.Q.a`) not
  in the official 34-ID set, breaking VP-008, changing the goal count from 34, and shifting the
  per-function counts asserted in BC-1.01.004 Postcondition 1 (PROTECT 19). The interaction
  between "split allowed" and "IDs must exactly match official set / count is 34" is
  unresolved.
- **Evidence:** BC-1.01.004:53 EC-002 "Goal split into sub-items under the official ID
  (e.g., 3.Q.a/3.Q.b)"; BC-1.01.004 VP-008 "Goal IDs exactly match the official CPG 2.0 ID
  set"; BC-1.01.004 Postcondition 1 "exactly 34 goals ... PROTECT 19".
- **Proposed Fix:** Clarify that splitting is a *post-authoring catalog-version event* (e.g.,
  2.0.1) with its own ID set and counts, and that VP-008/the 34-goal assertion are pinned to
  the *as-shipped* bundled catalog version, not invariant across versions. Or forbid splitting
  in v1 (ASM-002 is already validated as "every goal assessable as one ordinal question").

#### ADV-P1GREEN-P01-MED-004: Product brief floor-rule wording omits the `Unknown` trigger present everywhere downstream
- **Severity:** MEDIUM
- **Category:** spec-fidelity
- **Location:** product-brief.md:36, vs DI-003, BC-4.04.004, ubiquitous-language Floor Rule
- **Description:** The brief states the floor rule as "band is capped while any Critical-tier
  control is **Not Implemented**" — omitting `Unknown`. Every downstream artifact (DI-003,
  ubiquitous-language Floor Rule, BC-4.04.004 Postcondition 1, BC-3.03.003 Postcondition 3)
  triggers the floor on Critical-tier `Not Implemented` **or `Unknown`**. Since "honest-and-low
  first scores" with mostly-Unknown stores are a core onboarding story, whether a Critical
  Unknown caps the band is product-defining, and the brief's omission could mislead a reader
  taking the brief as canonical.
- **Evidence:** product-brief.md:36; DI-003 (invariants.md:23); BC-4.04.004:38.
- **Proposed Fix:** Update the brief's floor-rule parenthetical to "Not Implemented or
  Unknown" to match DI-003 and BC-4.04.004. Low effort, removes a credibility-relevant
  ambiguity at the top of the document tree.

### LOW

#### ADV-P1GREEN-P01-LOW-001: L2-INDEX Document Map caption says "DI-001..012" but invariants define DI-001..013
- **Severity:** LOW
- **Category:** spec-fidelity
- **Location:** L2-INDEX.md Document Map row (invariants.md), vs invariants.md, L2-INDEX ID Registry
- **Description:** The L2-INDEX Document Map describes invariants.md as "DI-001..012 business
  rules", but invariants.md defines DI-001 through DI-013 (DI-013 = No PII by design), and the
  L2-INDEX ID Registry table itself correctly lists DI count = 13. Internal off-by-one in the
  index caption.
- **Evidence:** L2-INDEX.md:41 "DI-001..012"; L2-INDEX.md ID Registry "DI-NNN | 13";
  invariants.md DI-001..DI-013.
- **Proposed Fix:** Change the Document Map caption to "DI-001..013".

#### ADV-P1GREEN-P01-LOW-002: `verify` is read-only per locking BC but recomputes/recaches — clarify it writes nothing
- **Severity:** LOW
- **Category:** ambiguous-language
- **Location:** BC-2.02.005 Invariant 1 / EC-list, BC-2.02.004 Postcondition 1/3, interface-definitions.md
- **Description:** BC-2.02.005 lists `verify` among read-only commands that "never take the
  lock and never fail due to it." BC-2.02.004 describes `verify` as recomputing cached Scores
  and flagging mismatches (E-STORE-003). It is implied but never stated that `verify` does not
  *repair*/rewrite cached scores (which would be a mutation needing the lock). Given DI-002
  (append-only, never silently rescored) this is almost certainly read-only, but the contract
  should say so to prevent an implementer adding a "fix cached scores" side effect.
- **Evidence:** BC-2.02.005:44; BC-2.02.004:40 (E-STORE-003 is a flag, not a fix).
- **Proposed Fix:** Add an explicit invariant to BC-2.02.004 (or the `verify` description):
  "`verify` never writes — it reports mismatches; correcting a cached score is not an
  operation (DI-002)."

## Summary

| Severity | Count |
|----------|-------|
| CRITICAL | 1 |
| HIGH | 3 |
| MEDIUM | 4 |
| LOW | 2 |

**Overall Assessment:** pass-with-findings
**Convergence:** findings remain — iterate
**Readiness:** requires revision before architecture begins

The spec package is unusually strong: traceability is dense and mostly bidirectional, the
scoring core has crisp purity/determinism contracts with Kani-targeted VPs, disclosure chains
(floor/override) are consistently threaded across surfaces, and DI-007/DI-013 are enforced
end-to-end. The findings are concentrated at subsystem seams — CLI surface vs BC text
(commands, flags, streams) and the assessment↔store orphan-answer boundary — exactly where
fresh-context review is expected to find drift. None block the *architecture* of the scoring
core; CRIT-001, HIGH-001, and HIGH-002 should be resolved before story decomposition since they
change command/flag/state contracts.

## Novelty Assessment

| Field | Value |
|-------|-------|
| **Pass** | 1 |
| **New findings** | 10 |
| **Duplicate/variant findings** | 0 |
| **Novelty score** | 1.0 |
| **Median severity** | 2.5 (MED) |
| **Trajectory** | 10 |
| **Verdict** | FINDINGS_REMAIN |
