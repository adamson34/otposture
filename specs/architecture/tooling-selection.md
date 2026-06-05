---
document_type: architecture-section
level: L3
section: "tooling-selection"
version: "1.0"
status: draft
producer: architect
timestamp: 2026-06-05T14:58:33
phase: 1b
inputs: [domain-spec/L2-INDEX.md, prd.md]
input-hash: "1ccddf836bc64b23fe870d3dfd9df37d"
traces_to: ARCH-INDEX.md
---

# Tooling Selection

> Versions pinned at toolchain-provisioning time (Phase 3 preflight); rows here fix the
> tool and its role, not the patch version.

| Tool | Role | Scope | Config notes |
|------|------|-------|--------------|
| rustc (stable, pinned via rust-toolchain.toml) | build | workspace | `#![forbid(unsafe_code)]` in pure crates |
| Kani | bounded model checking | op-score (10 VPs) | bounds: ≤64 goals, ≤8 functions, ≤8 bands; harnesses in `op-score/proofs/` |
| proptest | property tests | all pure crates | seed-pinned regressions checked in |
| cargo-fuzz (libFuzzer) | fuzzing | op-catalog::parse, op-store::codec::parse, op-render escaping | corpora committed under `fuzz/corpus/` |
| cargo-mutants | mutation testing | kill-rate gates per module-criticality.md (95/90/80) | CI nightly job, PR job on touched crates |
| cargo-deny | dependency policy | workspace | bans networking crates (ADR-007 list), license allowlist, per-crate I/O bans for pure core |
| custom fault-injection harness | crash consistency | op-store::fs | deterministic fault scheduler; every syscall site × every fault |
| syscall-trace harness | offline guarantee (VP-083) | otposture binary | Linux: strace filter; macOS: dtrace; Windows: documented manual gate |
| clap | CLI parsing | op-cli | derive API; exit code 2 from parser |
| askama | HTML templates | op-render | compile-time; autoescaping ON (VP-075 backstop) |
| num-rational (i128) | exact arithmetic | op-score | no f64 in scoring paths (clippy lint `float_arithmetic` deny in op-score) |
| serde / toml / serde_json | codecs | catalog, store, JSON | emission goes through the custom deterministic emitter, not toml::to_string (VP-022) |
| sha2 | catalog digests | op-catalog | canonicalize-then-hash (VP-005/006) |
| fs2 | advisory file locks | op-store::fs | O_EXCL-style create semantics (BC-2.02.005 EC-003) |
| lefthook + just | local gates / task runner | repo | mirrors CI jobs locally |

## CI Job Matrix (summary)

| Job | Gates |
|-----|-------|
| build+test (linux-x86_64, linux-aarch64, macos-aarch64, windows-x86_64) | unit/property/integration; **cross-platform golden scores byte-compared** (VP-052) |
| kani | 10 proof harnesses green (NFR-008) |
| fuzz (time-boxed PR job + long nightly) | no crashes/panics |
| mutants | kill-rate thresholds per criticality tier |
| deny + purity-lint | dependency bans; no I/O crates under op-score (VP-051/084) |
| syscall-trace | zero network syscalls across the integration suite (VP-083) |
| release (tags) | musl static build, Windows static CRT, binary-size gate < 25 MB (NFR-005) |
