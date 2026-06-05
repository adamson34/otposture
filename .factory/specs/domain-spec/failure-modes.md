---
document_type: domain-spec-section
level: L2
section: "failure-modes"
version: "1.0"
status: draft
producer: business-analyst
timestamp: 2026-06-05T12:10:44
phase: 1a
inputs: [product-brief.md, planning/research-report.md]
input-hash: "d0490fef4b5ce75c2b7fc3c0e135aeb5"
traces_to: L2-INDEX.md
---

# Failure Modes

> **Sharded L2 section (DF-021).** Navigate via `L2-INDEX.md`.
> Edge cases = unusual input (see `edge-cases.md`). Failure modes = what breaks at runtime.

| ID | Subsystem | Failure Mode | Impact | Detection | Recovery |
|----|-----------|-------------|--------|-----------|----------|
| FM-001 | Posture Store | Store file corrupted, truncated, or hand-edited into invalidity (power loss, sync-tool conflict, editor mishap) | Current Assessment unreadable; history at risk | Schema validation + per-snapshot integrity check on every load; recomputed Score must match cached Score (DI-010) | Append-only layout means earlier Snapshots usually survive; tool reports the first bad record, loads the valid prefix read-only, and names the damaged file/line for manual repair (human-readable format) |
| FM-002 | Catalog | Catalog file missing, unreadable, or failing validation at load | No scoring or assessment possible | CAP-001 validation with itemized defects (DEC-007) | Abort with the expected catalog `(id, version)` named and where to obtain it; never score against a partial catalog |
| FM-003 | Posture Store | Disk full / write interrupted mid-operation | Risk of half-written store | All writes are atomic: write temp file, fsync, rename; failure leaves the previous state intact | Re-run the command after freeing space; no repair step needed by design |
| FM-004 | Reporting | HTML template/rendering failure (malformed template, filesystem error at output path) | Report not produced | Render errors caught and reported with the failing element | Data layer untouched (report is a pure projection, CAP-008); fix path/template and re-run; never blocks snapshot/score operations |
| FM-005 | Posture Store | System clock wrong or moved backwards (common on air-gapped plant workstations) | Misleading timestamps on Answers/Snapshots | `seq` ordering is authoritative (DEC-006); suspicious regressions (taken_at < previous) flagged at snapshot time | Warn and proceed — wall-clock is informational; trend and diff are seq-based and unaffected |
| FM-006 | Posture Store | Concurrent invocation (two terminals, shared drive) racing on the store | Lost update or interleaved writes | Advisory lockfile acquired per mutating command; stale-lock detection by PID/age | Second process fails fast with a clear "store in use" message; ASM-008 holds this rare; append-only snapshots bound the blast radius |
| FM-007 | CLI / guided assessment | Process killed mid-assessment session (SSH drop, terminal closed) | Partial session loss | Each AnswerRecorded is persisted atomically as it is given (FM-003) — no end-of-session save | Re-run `assess`; already-answered Goals show current values; at most the in-flight answer is lost |
| FM-008 | Reporting | Output path is unwritable or collides with an existing file the user didn't intend to overwrite | Report lost or unintended overwrite | Pre-flight check on output path; explicit `--force` semantics for overwrite | Choose a new path; generated artifacts are always reproducible from Snapshots |

## Cross-Cutting Detection Principle

Every load validates; every score recomputes verifiably (DI-010); every write is atomic (FM-003). The Posture Store must survive the realities of plant engineering workstations: abrupt power-offs, no admin rights, wrong clocks, USB-stick transfers, and folder-sync tools.
