# ADR-007: Offline Guarantee Enforced at Build Time and Test Time

**Status:** accepted
**Date:** 2026-06-05

**Context:** DI-007/NFR-001/BC-6.06.003 make "zero network syscalls, ever" a product boundary OT buyers can audit. A promise in documentation is not auditable; the enforcement mechanism is the feature.

**Options considered:**
1. Convention + review — free, invisible, erodes with contributors.
2. **Build-time dependency ban + test-time syscall trace** — cargo-deny denies networking crates (reqwest, hyper, ureq, curl, tokio's net features, async-std net, native-tls/rustls-as-transport...) across the whole workspace feature graph (VP-084); CI runs the full integration suite under a syscall filter asserting zero socket/connect/DNS calls (VP-083).
3. OS-level sandboxing at runtime (seccomp/AppContainer) — strongest, but adds platform-specific code and failure modes on the exact locked-down workstations we target.

**Decision:** Option 2. Option 3 declined for v1 (a sandbox failing on a plant workstation is worse than the threat it mitigates, given option 2 already proves the binary *cannot* speak network protocols).

**Rationale:** Two independent layers — the binary cannot *contain* network code (deny) and is observed making no network syscalls (trace). Both produce CI artifacts a skeptical buyer can inspect, turning the marketing claim into evidence (KD-005).

**Consequences:** The cargo-deny ban list is a reviewed, versioned file; adding any networking dependency fails CI by design and requires an ADR superseding this one (per the brief: networking is a major-version product decision, effectively never). Windows trace gap documented in verification-coverage-matrix.md Gap Register.
