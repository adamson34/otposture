---
document_type: architecture-section
level: L3
section: "purity-boundary-map"
version: "1.0"
status: draft
producer: architect
timestamp: 2026-06-05T14:58:33
phase: 1b
inputs: [domain-spec/L2-INDEX.md, prd.md]
input-hash: "1ccddf836bc64b23fe870d3dfd9df37d"
traces_to: ARCH-INDEX.md
---

# Purity Boundary Map

> Classification: **pure core** (no effects, fully deterministic, proof-eligible) /
> **boundary** (pure logic adjacent to effects; property-testable with fakes) /
> **effectful shell** (I/O; integration-tested, fault-injected).

| Module / Submodule | Classification | Effects present | Verification consequence |
|--------------------|---------------|-----------------|--------------------------|
| op-score (all) | **pure core** | none — no I/O deps, no clock, no RNG, rational arithmetic only | Kani + proptest; mutation kill ≥95%; cross-platform golden determinism (VP-052) |
| op-catalog::parse/validate/digest | **pure core** | none (operates on `&str`/structs) | proptest round-trip (VP-001), fuzz parser (VP-002), Kani on validation soundness (VP-004) |
| op-catalog::bundled | boundary | `include_str!` at compile time only | build-time golden tests (VP-007/008) |
| op-store::codec | **pure core** | none (bytes ↔ structs) | proptest determinism/round-trip (VP-022/023/024), fuzz (VP-016) |
| op-store::fs | **effectful shell** | filesystem, fsync, rename, locks | fault-injection proptest (VP-010/012/014/026), process-level lock tests (VP-018) |
| op-assess::SessionState/step | **pure core** | none (state machine) | proptest on session invariants (VP-032 via kill-injection at the shell) |
| op-assess::Prompt impl | effectful shell (lives in op-cli) | TTY read/write | snapshot/integration tests |
| op-render (all) | **pure core** | none (data → String; askama compile-time) | golden tests (VP-074), escaping fuzz (VP-075), disclosure snapshot tests (VP-070) |
| op-cli::commands | **effectful shell** | argv, env, file read/write, exit codes | integration matrix (VP-080/081), syscall trace (VP-083) |
| op-cli::json | boundary | serde serialization only | schema validation (VP-085), shared-source goldens (VP-086) |

## The four load-bearing boundaries

1. **`op-store::codec` ↔ `op-store::fs`** — bytes cross; codec never sees paths. *Bug class trapped here:* partial-write corruption (FM-001/003).
2. **`op-assess::step` ↔ `Prompt`** — one trait object carries all TTY effects. *Bug class:* lost answers on kill (FM-007).
3. **`op-render` String ↔ `op-cli` write** — content complete before I/O starts. *Bug class:* truncated exec reports (FM-004/BC-5.05.004).
4. **timestamps as data** — `answered_at`/`taken_at` are *parameters* passed from op-cli into pure functions; nothing below the shell calls a clock. *Bug class:* clock-skew nondeterminism (FM-005, DEC-006).

## Rules enforced by CI

- **op-score:** `#![forbid(unsafe_code)]` + no I/O-capable crates in its dependency tree — enforceable at crate granularity via cargo-deny (VP-051), since op-score depends on nothing in the workspace.
- **op-store::codec, op-assess (minus Prompt), op-render:** `#![forbid(unsafe_code)]` + **submodule-level lint discipline** (clippy import lints denying `std::fs`/`std::net`/`std::process` in these modules + review gate). cargo-deny cannot apply here — these crates legitimately depend on the op-store *crate* (for types), whose `fs` submodule carries filesystem I/O; the ban is on what the *modules* import, not what the crate graph contains.
- Any new module must be classified in this file before merge (review gate).
