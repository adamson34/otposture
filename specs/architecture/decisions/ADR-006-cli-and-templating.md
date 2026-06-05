# ADR-006: clap for CLI; askama for Compile-Time HTML Templates

**Status:** accepted
**Date:** 2026-06-05

**Context:** BC-6.06.001 fixes an 11-command surface with strict exit-code classes; BC-5.05.003 requires a self-contained single-file HTML report with mandatory escaping (VP-075) from a single static binary (no runtime template files).

**Options considered (CLI):** clap (derive) — ecosystem standard, native suggestions ("did you mean"), exit 2 on usage errors out of the box; vs. hand-rolled parsing — full control, but reimplements help/suggestions/validation for no benefit.
**Options considered (HTML):** askama — templates compiled into the binary, type-checked, autoescaping; maud — macro-based, also compile-time, but templates-as-Rust hurts contributor accessibility; runtime engines (tera/handlebars) — require embedding + parsing template text at runtime, weaker type safety.

**Decision:** clap (derive API) + askama (autoescape on).

**Rationale:** Both put contract enforcement at compile time: clap makes the command surface declarative and diffable against interface-definitions.md; askama makes a missing escape a build failure rather than an injection bug (NFR-007). Both are pure-Rust, no-network, static-linking friendly (ADR-007/008 compatible).

**Consequences:** EC-001 ("did you mean") comes free. The exec report template lives in `op-render/templates/exec.html` and is reviewed like code. CSS is inlined at compile time (VP-073: zero external resources). clap's generated help text is golden-tested against interface-definitions.md to prevent drift.
