# ADR-008: Fully Static Release Binaries

**Status:** accepted
**Date:** 2026-06-05

**Context:** ASM-007: the binary must run on locked-down plant workstations with no admin rights, no installer, no runtime prerequisites — often air-gapped, often Windows, sometimes transferred by USB stick. NFR-005 caps size (<25 MB) and cold start (<200 ms).

**Options considered:**
1. Default dynamic linking — smaller, but glibc-version and VC++-redistributable failures are exactly the "ask IT for help" moment that kills adoption with this persona.
2. **Static: musl target for Linux (`x86_64`/`aarch64-unknown-linux-musl`), `+crt-static` for Windows MSVC, default for macOS** — one file, copy-and-run anywhere.
3. Container/installer distribution — admin rights and infrastructure the persona doesn't have; contradicts the no-footprint thesis.

**Decision:** Option 2.

**Rationale:** "Download one file, run it" is the entire installation story — a differentiator vs. CSET's desktop installer (research Q3) and a requirement for air-gapped transfer. The dependency set (ADR-006: clap, askama, num-rational, toml, serde, sha2, fs2) is all pure Rust — no C library friction with musl.

**Consequences:** Release CI builds the 4-target matrix with size gates (tooling-selection.md). `--version` output includes target triple for support. SHA-256 checksums published per release artifact (air-gap transfer integrity — the same canonicalize-and-hash habit as catalog digests).
