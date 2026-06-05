# ADR-003: Exact Rational Arithmetic (num-rational over i128)

**Status:** accepted
**Date:** 2026-06-05

**Context:** DI-012 pins level values as exact rationals (1/3, 2/3); DI-005 requires drivers to sum *exactly* to deltas; NFR-003 requires bit-identical scores across platforms; VP-044/053 are equational identities that Kani must prove.

**Options considered:**
1. f64 — fast and familiar, but 1/3 is unrepresentable; driver-sum reconciliation becomes epsilon-tolerance reasoning; cross-platform bit-identity is fragile (FMA, optimization levels); Kani equality proofs become inequality bounds.
2. Fixed-point (scaled integers, e.g. ×10⁶) — deterministic but 1/3 still truncates; reconciliation reintroduces residue bookkeeping.
3. **num-rational `Ratio<i128>`** — exact for every value the domain produces; equality is equality; deterministic everywhere; cost: overflow risk on pathological weights and unfamiliarity.

**Decision:** Option 3, with overflow made impossible by validation: BC-1.01.002 already bounds the domain (weights > 0; Kani bounds ≤64 goals); add a documented weight cap (≤ 10⁶) to rule (c) feasibility margin — denominators stay ≪ i128 range.

**Rationale:** Every "defensible number" claim (KD-002/003) reduces to arithmetic a skeptic can re-derive by hand. Rationals make the spec's equations literally true in code, and make VP-044/053 provable as equations rather than approximations.

**Consequences:** clippy `float_arithmetic` deny in op-score. Percent display rounding happens once at the presentation edge (DEC-008, `display_round`). Store serializes rationals as `"n/d"` strings. JSON emits both `"percent": 0.6765` (display convenience, documented as rounded) and `"percent_exact": "23/34"`.
