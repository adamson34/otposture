---
document_type: behavioral-contract
level: L3
version: "1.1"
status: draft
producer: product-owner
timestamp: 2026-06-05T12:50:04
phase: 1a
inputs: [domain-spec/L2-INDEX.md, product-brief.md]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: domain-spec/L2-INDEX.md
origin: greenfield
subsystem: "SS-04"
capability: "CAP-004"
lifecycle_status: active
introduced: v0.1.0
modified: []
deprecated: null
deprecated_by: null
replacement: null
retired: null
removed: null
removal_reason: null
---

# Behavioral Contract BC-4.04.004: Floor Rule

## Description

The credibility centerpiece (DI-003): while any effective-Critical-tier goal is `Not Implemented` or `Unknown`, the headline band cannot exceed **Developing**. A fatal gap can never be averaged away — the direct answer to the canonical CISO objection.

## Preconditions

1. Pre-floor band derived (BC-4.04.003); effective tiers resolved (catalog defaults + active overrides, BC-3.03.004).

## Postconditions

1. If the gating set {goals: effective_tier = Critical ∧ level ∈ {NI, Unknown}} is non-empty: final band = min(band_uncapped, Developing); `floor_rule_triggered = true`; `critical_gaps[]` lists every gating goal with its level and effective-tier source (default vs. override).
2. If empty: final band = band_uncapped; `floor_rule_triggered = false`.
3. The percent is never altered by the floor rule — only the band (DEC-010: they may move in opposite directions, both shown).

## Invariants

1. Floor evaluation uses effective tiers at scoring time; historical snapshots are never retroactively re-floored (events.md rule 4).
2. `critical_gaps[]` is exhaustive — no surface may show "floor triggered" without the complete gap list available.
3. A Critical goal at PI or LI does **not** trigger the floor (partial implementation is progress; the floor targets absent/unknown).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Percent in Resilient range, one Critical Unknown | Band: Developing (capped); uncapped band retained for display; gap named |
| EC-002 | Override demotes the only gating goal mid-cycle | Next score uncapped; disclosure per DEC-005; prior snapshots keep their capped band |
| EC-003 | Critical goal answered N/A | N/A excludes it from scoring AND from the gating set (a goal that genuinely doesn't apply can't gate) — its N/A status + tier appear in detail views for auditability |
| EC-004 | All goals Critical, all Unknown | Floor triggered; band Foundational anyway (percent 0) — cap is a no-op but `floor_rule_triggered` still true with full gap list |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| 33 goals FI, one Critical NI, thresholds (.40,.65,.85) | percent ≈ .97, band_uncapped Resilient, band Developing, 1 gap | happy-path |
| Same but the NI goal is Standard-tier | Band Resilient; floor not triggered | happy-path |
| Critical goal at PI | Floor not triggered | edge-case |
| Critical goal N/A | Floor not triggered; goal excluded from percent | edge-case |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-048 | Floor soundness: gating set non-empty ⇒ final band ≤ Developing | kani |
| VP-049 | Floor completeness: gating set empty ⇒ final band = band_uncapped | kani |
| VP-050 | critical_gaps[] = exactly the gating set | kani/proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-004 |
| L2 Domain Invariants | DI-003 |
| L2 Edge Cases | DEC-005, DEC-010 |
| L2 Risks | R-002 (credibility) |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-4.04.003 — depends on (caps its output)
- BC-3.03.004 — depends on (effective tiers)
- BC-5.05.001/003 — blocks (disclosure surfaces)
