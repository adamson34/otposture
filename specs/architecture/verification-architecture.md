---
document_type: architecture-section
level: L3
section: "verification-architecture"
version: "1.0"
status: draft
producer: architect
timestamp: 2026-06-05T14:58:33
phase: 1b
inputs: [domain-spec/L2-INDEX.md, prd.md]
input-hash: "1ccddf836bc64b23fe870d3dfd9df37d"
traces_to: ARCH-INDEX.md
---

# Verification Architecture

> **Iron Law compliance:** every architecture decision below includes a feasibility
> assessment. The design was shaped *by* provability — rational arithmetic, the pure
> core, and timestamps-as-data exist because their alternatives are unprovable.

## Provable Properties Catalog (formal proofs — Kani, bounded)

All ten formal targets live in `op-score` on bounded inputs (catalog ≤ 64 goals, ≤ 8 functions, ≤ 8 bands — the bundled catalog is 34/6/4, so bounds exceed reality with margin):

| VP | Property | Why feasible |
|----|----------|--------------|
| VP-040 | percent ∈ [0,1] | i128 rationals, bounded sums — no overflow within bounds (denominator ≤ 64 × max_weight) |
| VP-041 | single-level monotonicity | finite enum lattice; direct symbolic check |
| VP-044 | function/overall decomposition identity | exact rational equality — no epsilon reasoning needed |
| VP-046 | band function monotone in percent | ordered thresholds; symbolic comparison |
| VP-047 | band function total (no gaps/overlaps) | partition of [0,1] by N−1 strict thresholds |
| VP-048 | floor soundness (gating ⇒ band ≤ floor-cap) | positional cap = index comparison |
| VP-049 | floor completeness (no gating ⇒ uncapped) | complement of VP-048 |
| VP-050 | critical_gaps = exactly the gating set | set equality over bounded goal array |
| VP-053 | driver sum + composition adjustment = delta | exact rational arithmetic makes this an equation, not an approximation |
| VP-056 | comparability guard soundness | digest equality + break-interval check; small state |

**Feasibility verdict:** all ten FEASIBLE precisely because of ADR-003 (rationals — equational reasoning) and ADR-001 (no I/O in scope). Estimated harness cost: small; each property is a single pure function with bounded symbolic inputs.

## Property-Test Layer (proptest — unbounded random)

- Round-trips: catalog parse/serialize (VP-001), store codec (VP-022/023/024), JSON schema (VP-085)
- Metamorphic: permutation invariance (VP-043), N/A independence (VP-042), sub-score independence (VP-045), NBA gain truthfulness (VP-058)
- Stateful: append-only byte stability (VP-014), seq gaplessness (VP-015), session durability monotonicity (VP-032), upgrade content preservation (VP-025/027)

## Fault-Injection Layer (custom harness over op-store::fs)

Deterministic fault scheduler wraps every fs syscall; for each operation, every single fault point is exercised: VP-010 (init atomicity), VP-012 (crash consistency), VP-026 (upgrade crash safety), VP-076 (report output safety). *Feasible because:* the fs layer is small (≤ ~8 syscall sites per operation) and operations are short.

## Fuzz Layer (cargo-fuzz)

Parser targets: catalog TOML (VP-002), store files (VP-016), HTML-escape path with adversarial store content (VP-075).

## Environment-Gate Layer (CI)

Syscall-trace harness for the offline guarantee (VP-083), cargo-deny dependency bans (VP-084), cross-platform golden score matrix (VP-052), exit-code matrix (VP-080), purity lint — no I/O crates under op-score (VP-051).

## Testable-Only (explicitly not provable, and why)

| Concern | Why not provable | Mitigation |
|---------|------------------|-----------|
| TTY interaction (BC-3.03.002 UX) | inherently effectful/human | state machine extracted pure; snapshot tests on the shell |
| fsync actually durable on given hardware | OS/hardware promise | discipline + documented backup guidance (R-006) |
| HTML renders legibly at print width | visual judgment | golden HTML + manual review gate |
| <15-min session (ASM-004/NFR-004) | human-timing claim | dogfood measurement |

## Engine-Version Discipline

`op-score` exposes `ENGINE_VERSION`; snapshots record it (BC-4.04.005 EC-002). Any change to scoring behavior bumps it, and `verify` distinguishes engine drift from corruption — making even bugfixes honest in history.
