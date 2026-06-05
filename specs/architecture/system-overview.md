---
document_type: architecture-section
level: L3
section: "system-overview"
version: "1.0"
status: draft
producer: architect
timestamp: 2026-06-05T14:58:33
phase: 1b
inputs: [domain-spec/L2-INDEX.md, prd.md]
input-hash: "1ccddf836bc64b23fe870d3dfd9df37d"
traces_to: ARCH-INDEX.md
---

# System Overview

otposture is a **single static Rust binary** implementing a local pipeline: validated framework catalog (data) → ordinal answers (TOML store) → pure deterministic scoring → append-only snapshot history → projections (terminal status/diff, exec HTML, JSON).

## Architecture Principles

1. **Functional core, imperative shell.** All scoring, ranking, diffing, banding, and validation logic lives in pure crates with no I/O capability. The shell (CLI, filesystem) is thin and dumb. This is load-bearing: DI-010 determinism, NFR-008 Kani proofs, and R-005 mitigation all assume it.
2. **Data over code.** Frameworks, weights, thresholds, tiers, and rating metadata are catalog *data* (DI-008). The engine is framework-agnostic; CPG 2.0 is the bundled default, not a special case.
3. **Honesty invariants are structural.** Append-only files, positional floor cap, driver-sum reconciliation, and series breaks are enforced by types and storage layout, not by convention.
4. **Plant-workstation realism.** Every write is atomic (temp+fsync+rename); loads validate and salvage; clocks are untrusted (seq ordering); no admin rights, no installer, no network — ever.

## Hard Constraints (non-negotiable, from specs)

| Constraint | Source | Enforcement |
|-----------|--------|-------------|
| Zero network syscalls | DI-007, NFR-001, BC-6.06.003 | cargo-deny crate ban + CI syscall trace (ADR-007) |
| No PII in schema/outputs | DI-013, NFR-006 | No identity fields exist in any type; review gate |
| Pure scoring core, no I/O deps | NFR-009, DI-010 | `op-score` crate has no I/O-capable dependencies (ADR-001) |
| Exact rational arithmetic | DI-012, NFR-003 | num-rational i128; no f64 in scoring paths (ADR-003) |
| Atomic writes, append-only history | DI-002, FM-003 | `op-store` write discipline (BC-2.02.002/003) |
| Deterministic human-readable store | BC-2.02.008 | Custom deterministic TOML emitter (ADR-002) |

## Workspace Shape

```
otposture/
├── Cargo.toml            # workspace
├── crates/
│   ├── op-score/         # SS-04 pure core — types + scoring (Kani target)
│   ├── op-catalog/       # SS-01 catalog parse/validate/digest (pure given bytes)
│   ├── op-store/         # SS-02 persistence: codec (pure) + fs shell (effectful)
│   ├── op-assess/        # SS-03 session state machine (pure) + prompt trait
│   ├── op-render/        # SS-05 status/diff text + exec HTML (pure: data → string)
│   └── op-cli/           # SS-06 bin `otposture`: clap, exit codes, JSON, wiring
└── assets/catalogs/cisa-cpg-2.0.0.toml   # bundled catalog (compiled in via include_str!)
```

## Primary Data Flow

```
catalog.toml ──parse/validate──▶ Catalog ─┐
store files  ──load/verify────▶ Store ────┼──▶ op-score::score() ──▶ Score
user input   ──assess/answer──▶ Answers ──┘         │
                                                    ├──▶ op-score::delta() ──▶ Delta
                                                    └──▶ op-render ──▶ terminal / HTML / JSON
```

All arrows left of `score()` are the only effectful operations; everything from `score()` rightward is pure until the final output write.
