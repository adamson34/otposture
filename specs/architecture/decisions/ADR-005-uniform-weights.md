# ADR-005: Uniform Goal Weights in v1; Criticality via Tiers + Floor Rule

**Status:** accepted (resolves PRD OQ3)
**Date:** 2026-06-05

**Context:** The bundled catalog needs per-goal weights for DI-012. Differentiated weights imply an editorial claim ("3.H matters 2.3× more than 1.A") that must be defended goal-by-goal (R-007).

**Options considered:**
1. CISA-impact-derived weights (High=3/Moderate=2/Low=1) — uses official data, but double-counts impact once next-best-actions also display it, and makes the % harder to re-derive mentally.
2. **Uniform weights (1.0 each)** — "X% ≈ weighted share of 34 goals implemented" is explainable in one breath; criticality is expressed where it's categorical, not scalar: the Critical tier + floor rule (DI-003) makes a fatal gap cap the band regardless of weight.
3. Editorial OT-opinionated weights — maximum opinion, maximum attack surface for credibility challenges.

**Decision:** Option 2 for the bundled v1 catalog. The engine remains weight-capable (DI-012 is weighted; BC-1.01.002 validates weights) — uniform is a *content* choice, not an engine limitation.

**Rationale:** The floor rule already delivers the "criticality changes the answer" property with a mechanism that cannot be averaged away — strictly stronger than weight skew, and defensible in one sentence. Uniform weights make every percent hand-checkable (`23/34`), serving KD-003 transparency.

**Consequences:** `potential_gain` in next-best-actions differentiates by current level only (plus effort divisor + impact display) in v1. If community calibration later wants weights, that's a catalog version bump with back-cast (DI-001), not a code change. Document in README's scoring-rationale section.
