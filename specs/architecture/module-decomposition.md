---
document_type: architecture-section
level: L3
section: "module-decomposition"
version: "1.0"
status: draft
producer: architect
timestamp: 2026-06-05T14:58:33
phase: 1b
inputs: [domain-spec/L2-INDEX.md, prd.md]
input-hash: "1ccddf836bc64b23fe870d3dfd9df37d"
traces_to: ARCH-INDEX.md
---

# Module Decomposition

> One crate per subsystem; `op-score` additionally owns the shared domain types so the
> pure core sits at the dependency root (ADR-001).

## op-score (SS-04 Scoring Engine) — CRITICAL, pure

**Responsibility:** Domain types (`Catalog`, `Goal`, `Band`, `Tier`, `Level`, `Answer`, `Score`, `Delta`, `Driver`, `Rational`) and every computation: weighted percent (BC-4.04.001), function sub-scores (BC-4.04.002), band derivation (BC-4.04.003), floor rule (BC-4.04.004), determinism contract (BC-4.04.005), delta + driver attribution + largest-remainder display rounding (BC-4.04.006), comparability guard (BC-4.04.007), next-best-actions (BC-4.04.008), effective-tier resolution (consumed by BC-3.03.004).
**Forbidden:** any I/O-capable dependency, f64 in scoring paths, clocks, randomness.

## op-catalog (SS-01 Framework Catalog) — HIGH, pure-given-bytes

**Responsibility:** Catalog TOML parsing into `op-score` types (BC-1.01.001), 8-rule validation with itemized defects (BC-1.01.002), canonicalization + SHA-256 content digest (BC-1.01.003), bundled CPG 2.0 content embedded via `include_str!` with build-time golden validation (BC-1.01.004).
**Signature discipline:** `parse(&str) -> Result<Catalog, Vec<Defect>>` — file reading happens in op-cli.

## op-store (SS-02 Posture Store) — CRITICAL, split pure/effectful

**Responsibility:** Two submodules with a hard internal boundary:
- `codec` (pure): deterministic TOML serialization/parsing of store files (BC-2.02.008), schema validation, integrity model (BC-2.02.004 logic).
- `fs` (effectful): atomic temp+fsync+rename writes (BC-2.02.002), store init (BC-2.02.001), append-only snapshot persistence (BC-2.02.003), advisory locking (BC-2.02.005), back-cast migration storage (BC-2.02.006), series breaks (BC-2.02.007), schema upgrade with backup (BC-2.02.009).

## op-assess (SS-03 Assessment) — HIGH, pure state machine + prompt trait

**Responsibility:** Answer recording rules incl. goal-ID normalization/suggestion (BC-3.03.001), guided session as a pure state machine (`SessionState × Input → SessionState × Effect`) driven through a `Prompt` trait implemented by op-cli (BC-3.03.002), Unknown-default semantics helpers (BC-3.03.003), tier-override lifecycle records (BC-3.03.004).
**Why a state machine:** session resumability and kill-safety (FM-007) test deterministically without a TTY.

## op-render (SS-05 Reporting) — HIGH, pure

**Responsibility:** Status view model + terminal rendering (BC-5.05.001), diff display (BC-5.05.002), exec HTML via askama compile-time templates with mandatory HTML escaping of all store-sourced strings (BC-5.05.003), disclosure chain (floor caps, overrides, series breaks, completeness caveats) as a shared `Disclosures` struct consumed by every surface.
**Signature discipline:** `(Score/Delta/Store views) -> String` — output file writing happens in op-cli (BC-5.05.004 lives at that boundary).

## op-cli (SS-06 CLI Surface) — MEDIUM, effectful shell

**Responsibility:** clap command tree (11 commands, BC-6.06.001), exit-code mapping from the error taxonomy, error rendering (BC-6.06.002), JSON projections via serde of the same computed structures (BC-6.06.004), file reading/writing orchestration, lock acquisition ordering, the offline guarantee surface (BC-6.06.003 — enforced by what this crate *doesn't* link).

## BC → Module Traceability (summary)

| Module | BCs |
|--------|-----|
| op-catalog | BC-1.01.001..004 |
| op-store | BC-2.02.001..009 |
| op-assess | BC-3.03.001..004 |
| op-score | BC-4.04.001..008 (+ effective tiers for BC-3.03.004) |
| op-render | BC-5.05.001..003 |
| op-cli | BC-5.05.004, BC-6.06.001..004 |
