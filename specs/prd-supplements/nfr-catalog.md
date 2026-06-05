---
document_type: prd-supplement-nfr-catalog
level: L3
version: "1.0"
status: draft
producer: product-owner
timestamp: 2026-06-05T12:50:04
phase: 1a
inputs: [prd.md, domain-spec/L2-INDEX.md]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: prd.md
---

# Non-Functional Requirements Catalog: otposture

> Referenced by: architect, performance-engineer, formal-verifier.

## NFR Registry

| ID | Category | Requirement | Target | Validation Method | Priority | Risk Source |
|----|----------|-------------|--------|------------------|----------|-------------|
| NFR-001 | Security | Zero network syscalls under all commands and conditions | 0 syscalls, all platforms | CI syscall-trace harness + cargo-deny dependency policy (VP-083/084) | P0 | DI-007 |
| NFR-002 | Reliability | Crash consistency: any single fault (power loss, kill, ENOSPC) leaves the store loadable at the prior state | 100% of injected fault points | fault-injection proptest (VP-012) | P0 | R-006, FM-003 |
| NFR-003 | Reliability | Bit-identical scoring across platforms (Windows/Linux/macOS, x86_64/aarch64) | exact equality on golden fixtures | CI cross-platform golden matrix (VP-052) | P0 | R-005, DI-010 |
| NFR-004 | Performance | Returning-user session: update answers + snapshot + exec report | < 15 min human time; every command < 1 s on a 2015-era workstation | dogfood timing (ASM-004) + command benchmarks | P0 | brief success criteria |
| NFR-005 | Performance | Binary footprint & startup suitable for locked-down workstations | single static binary < 25 MB; cold start < 200 ms; no admin rights, no installer | CI size gate + portable-execution test (ASM-007) | P1 | ASM-007 |
| NFR-006 | Security | No PII anywhere: schema, store, reports, JSON, diagnostics | 0 personal-data fields; free-text documented non-PII | schema review + report/JSON audit (DI-013) | P0 | DI-013 |
| NFR-007 | Security | Store free text is untrusted input: HTML-escaped in reports, control-char-rejected at entry | 0 injection paths | fuzz store→render (VP-075) | P0 | BC-5.05.003 EC-004 |
| NFR-008 | Reliability | Scoring-core quality gate: CRITICAL-tier mutation kill rate | ≥ 95% (scoring core, store); Kani proofs on VP-040/041/044/046/047/048/049/050/053/056 | cargo-mutants + Kani in CI | P0 | R-005 |
| NFR-009 | Maintainability | Pure core / effectful shell separation: scoring crate has no I/O dependency | crate-graph enforced | CI lint (VP-051) | P0 | DI-010 |
| NFR-010 | Scalability | Store performance over years of use | 500 snapshots × 34 goals: all commands < 1 s, store < 10 MB | synthetic-history benchmark | P1 | — |

## NFR Categories

| Category | Validation Agent |
|----------|-----------------|
| Security (NFR-001/006/007) | security-reviewer |
| Reliability (NFR-002/003/008) | formal-verifier |
| Performance (NFR-004/005/010) | performance-engineer |
| Maintainability (NFR-009) | code-reviewer |

## NFR-to-Module Mapping

| NFR | Affected Modules | Architectural Impact |
|-----|-----------------|---------------------|
| NFR-001 | all | no networking crates in dependency graph; no URL-constructing code |
| NFR-002 | posture-store | temp+fsync+rename everywhere; append-only file design |
| NFR-003, NFR-008, NFR-009 | scoring-core | separate pure crate; fixed-point/rational arithmetic; Kani-provable function signatures |
| NFR-005 | CLI shell | static linking (musl on Linux), minimal dependencies, lazy catalog parse |
| NFR-006, NFR-007 | store, reporting | schema discipline; escaping at the render boundary |
| NFR-010 | posture-store | seq-indexed append format; no full-history rewrite on mutation |
