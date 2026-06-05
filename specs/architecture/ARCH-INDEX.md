---
document_type: architecture-index
level: L3
version: "1.1"
status: draft
producer: architect
timestamp: 2026-06-05T14:58:33
phase: 1b
inputs: [domain-spec/L2-INDEX.md, prd.md]
input-hash: "1ccddf836bc64b23fe870d3dfd9df37d"
traces_to: prd.md
deployment_topology: single-service
---

# Architecture Index: otposture

> **Context Engineering:** lightweight index. Agents load ONLY the section files they need.

## Document Map

| Section | File | Primary Consumer | Purpose |
|---------|------|-----------------|---------|
| System Overview | system-overview.md | orchestrator, all agents | Vision, principles, hard constraints, workspace shape |
| Module Decomposition | module-decomposition.md | story-writer, implementer | Crate catalog, responsibilities, BC mapping |
| Dependency Graph | dependency-graph.md | story-writer, consistency-validator | Crate dependencies, topological build order |
| API Surface | api-surface.md | test-writer, implementer | Crate public APIs + CLI/JSON external surface |
| Verification Architecture | verification-architecture.md | formal-verifier, architect | Provable Properties Catalog, proof strategy |
| Purity Boundary Map | purity-boundary-map.md | implementer, formal-verifier | Pure core / boundary / effectful shell per module |
| Tooling Selection | tooling-selection.md | formal-verifier | Kani, proptest, cargo-mutants, cargo-deny config |
| Verification Coverage | verification-coverage-matrix.md | consistency-validator | VP-to-module coverage |

## Cross-References

| If you need... | Read these together |
|----------------|-------------------|
| Implementation plan for a module | module-decomposition.md + dependency-graph.md + api-surface.md |
| Verification plan for a module | verification-architecture.md + purity-boundary-map.md + tooling-selection.md |
| Full module picture | module-decomposition.md + purity-boundary-map.md + verification-coverage-matrix.md |
| Story decomposition input | module-decomposition.md + dependency-graph.md |

## Subsystem Registry

| SS ID | Name | Architecture Doc | Implementing Modules | Phase Introduced |
|-------|------|-----------------|---------------------|-----------------|
| SS-01 | Framework Catalog | module-decomposition.md | `op-catalog` | Phase 1 |
| SS-02 | Posture Store | module-decomposition.md | `op-store` | Phase 1 |
| SS-03 | Assessment | module-decomposition.md | `op-assess` | Phase 1 |
| SS-04 | Scoring Engine | module-decomposition.md | `op-score` | Phase 1 |
| SS-05 | Reporting | module-decomposition.md | `op-render` | Phase 1 |
| SS-06 | CLI Surface | module-decomposition.md | `op-cli` (bin: `otposture`) | Phase 1 |

## Architecture Decisions

| ID | Decision | Rationale |
|----|----------|-----------|
| ADR-001 | Cargo workspace; `op-score` pure core at dependency root | Kani/proptest need an I/O-free crate (NFR-009); decisions/ADR-001-workspace-pure-core.md |
| ADR-002 | TOML store format, deterministic serializer, `[[snapshot]]` appends | BC-2.02.008 constraints; user-validated; decisions/ADR-002-toml-store-format.md |
| ADR-003 | Exact rational arithmetic (num-rational, i128) | DI-012 exact rationals, cross-platform determinism; decisions/ADR-003-rational-arithmetic.md |
| ADR-004 | Bundled band thresholds 0.40/0.65/0.85 with published rationale | Resolves PRD OQ2; decisions/ADR-004-band-thresholds.md |
| ADR-005 | Uniform goal weights in v1; criticality via tiers + floor rule | Resolves PRD OQ3; decisions/ADR-005-uniform-weights.md |
| ADR-006 | clap (CLI) + askama (compile-time HTML templates) | Single-binary, no runtime template files; decisions/ADR-006-cli-and-templating.md |
| ADR-007 | Offline guarantee enforced by cargo-deny ban + CI syscall trace | NFR-001 as build-time + test-time gates; decisions/ADR-007-offline-enforcement.md |
| ADR-008 | Static linking: musl (Linux), static CRT (Windows) | ASM-007 no-admin portability, NFR-005; decisions/ADR-008-static-linking.md |
