# ADR-002: TOML Store Format with a Custom Deterministic Emitter

**Status:** accepted (user-validated 2026-06-05)
**Date:** 2026-06-05

**Context:** BC-2.02.008 requires deterministic, human-readable, comment-capable, git-diffable serialization; BC-2.02.003 requires clean appends; the persona hand-repairs files on plant workstations (FM-001 recovery path). PRD OQ1 left YAML/TOML/JSON open.

**Options considered:**
1. YAML (strict subset) — familiar; block style diffs well; but the spec invites footguns (implicit typing, the Norway problem), "strict subset" is a discipline not a guarantee, and Rust YAML emitters are weaker on deterministic output.
2. **TOML** — appending a `[[snapshot]]` table is a literal file append (perfect append-only fit); comments; first-class Rust ecosystem; unambiguous typing; slightly verbose nesting.
3. NDJSON log + TOML config hybrid — purest appends, but two formats to document and JSON forbids comments (store headers carry the schema/non-PII notices).

**Decision:** Option 2 — TOML everywhere (store files *and* catalog files, one format to learn), with a **custom deterministic emitter** (stable key order, fixed number/datetime formatting, LF) rather than `toml::to_string` (VP-022 requires byte-stable output that generic emitters don't promise).

**Rationale:** The append-only snapshot log maps exactly onto appended `[[snapshot]]` tables — the storage layout *is* the honesty invariant (DI-002). Rationals serialize as `"23/34"` strings, sidestepping float representation entirely.

**Consequences:** `op-store::codec` owns emission; nobody else serializes store content. Catalog and store share canonicalization logic (also feeds the SHA-256 digest, VP-005). File layout: `otposture.toml`, `assessment.toml`, `snapshots.toml`, `overrides.toml`, `migrations.toml`, `.lock`.
