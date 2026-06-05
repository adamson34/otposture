# ADR-004: Bundled Band Thresholds 0.40 / 0.65 / 0.85

**Status:** accepted (resolves PRD OQ2)
**Date:** 2026-06-05

**Context:** The bundled CPG 2.0 catalog needs default thresholds for Foundational/Developing/Managed/Resilient, with a written rationale that survives CISO scrutiny (R-002). Thresholds are catalog data (DI-008) — this decision sets defaults, not engine behavior.

**Options considered:**
1. Even quartiles (0.25/0.50/0.75) — symmetric but meaningless: 26% of CPGs "Largely Implemented" should not read as the second band when critical hygiene is absent.
2. **0.40 / 0.65 / 0.85** — anchored to level semantics: 0.40 ≈ everything at least Partially + a third Largely (a real program exists); 0.65 ≈ the Largely-Implemented average (2/3 — the catalog's own "Largely" value, a natural anchor); 0.85 ≈ most goals Fully with a few Largely (mature, sustained).
3. Defer to community calibration post-release — honest but ships an unscoreable v1.

**Decision:** Option 2, encoded in the bundled catalog with this rationale embedded as catalog comments, explicitly marked as revisable defaults.

**Rationale:** Each threshold is explainable in one sentence tied to the answer scale itself rather than arbitrary percentiles — consistent with the transparency thesis (KD-003). The middle threshold equaling the "Largely" level value (2/3 ≈ 0.65, displayed boundary set at 0.65 exactly) gives execs an intuitive reading: "Managed means you're at least 'largely implemented' on a weighted basis."
**Floor interaction:** the cap band is positional (DI-003); these thresholds only place the uncapped band.

**Consequences:** Thresholds appear in the catalog file with rationale comments (BC-1.01.004 inv 2 spirit). Changing them is a catalog version bump → snapshot pinning and back-cast rules apply (DI-001), so even threshold tuning is honest in history. Community feedback can revise in CPG catalog v2.0.x without engine changes.
