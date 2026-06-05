---
document_type: architecture-section
level: L3
section: "dependency-graph"
version: "1.0"
status: draft
producer: architect
timestamp: 2026-06-05T14:58:33
phase: 1b
inputs: [domain-spec/L2-INDEX.md, prd.md]
input-hash: "1ccddf836bc64b23fe870d3dfd9df37d"
traces_to: ARCH-INDEX.md
---

# Dependency Graph

> Strictly acyclic. `op-score` is the root: it depends on nothing in the workspace.

```
                 ┌──────────┐
                 │ op-score │  (pure core, domain types)
                 └────▲─────┘
        ┌─────────┬───┴────┬──────────┐
   ┌────┴────┐ ┌──┴──────┐ ┌┴────────┐│
   │op-catalog│ │op-store │ │op-assess││
   └────▲────┘ └──▲──────┘ └▲────────┘│
        │         │         │    ┌────┴────┐
        │         │         │    │op-render│
        │         │         │    └────▲────┘
        └─────────┴────┬────┴─────────┘
                  ┌────┴───┐
                  │ op-cli │  (bin: otposture)
                  └────────┘
```

## Edge List (allowed dependencies only)

| Crate | Depends on (workspace) | Key external deps |
|-------|------------------------|-------------------|
| op-score | — | num-rational, serde (derive only) |
| op-catalog | op-score | toml, sha2, serde |
| op-store | op-score | toml, fs2 (lock), serde |
| op-assess | op-score, op-store | — |
| op-render | op-score, op-store | askama |
| op-cli | all of the above | clap, serde_json, anyhow (shell only) |

## Rules

1. **No edge may point downward** in the table above (e.g., op-score may never gain a workspace dependency). CI check: `cargo metadata` graph assertion.
2. **op-render never depends on op-assess** (reporting reads committed state, not session state).
3. **Banned everywhere:** networking crates (reqwest, hyper, tokio-net family — ADR-007), chrono `now()`/`std::time::SystemTime` inside op-score (timestamps enter as data).
4. **Topological build/test order:** op-score → op-catalog, op-store → op-assess, op-render → op-cli. Story waves follow this order (input to Phase 2 wave scheduling).

## Boundary Justification (where pure meets impure)

- `op-store::codec` ↔ `op-store::fs`: bytes cross here; codec never sees paths, fs never interprets content.
- `op-assess::SessionState` ↔ `Prompt` trait: TTY effects isolated behind one trait object.
- `op-render` output `String` ↔ `op-cli` file write: report content is fully formed before any I/O begins (FM-004 — render failure cannot corrupt output files).
