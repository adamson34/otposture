# ADR-001: Cargo Workspace with `op-score` Pure Core at the Dependency Root

**Status:** accepted
**Date:** 2026-06-05

**Context:** NFR-009 requires a scoring core with no I/O dependency; NFR-008 requires Kani proofs on it; DI-010 requires bit-identical determinism. These are only enforceable if purity is a *compilation boundary*, not a convention.

**Options considered:**
1. Single crate with module-level discipline — simple, but purity is unenforceable (any module can `use std::fs`); lint-only.
2. **Workspace of 6 crates, one per subsystem, `op-score` at the root with zero workspace deps** — purity enforced by `cargo metadata`/cargo-deny; crate granularity matches the SS-01..06 BC structure; slower cold builds.
3. Two crates (core + shell) — enforces the one boundary that matters but loses per-subsystem traceability and per-crate mutation gates.

**Decision:** Option 2.

**Rationale:** The credibility thesis (R-002/R-005) hangs on provable scoring; a compiler-enforced boundary is categorically stronger than review discipline. Per-subsystem crates also give story decomposition natural seams and let cargo-mutants apply differentiated kill-rate gates (95/90/80).

**Consequences:** Shared domain types live in `op-score` (the root), so even the catalog parser produces core types. Cross-crate API changes are intentionally noisy. CI asserts the dependency edge list in dependency-graph.md.
