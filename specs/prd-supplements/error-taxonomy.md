---
document_type: prd-supplement-error-taxonomy
level: L3
version: "1.0"
status: draft
producer: product-owner
timestamp: 2026-06-05T12:50:04
phase: 1a
inputs: [prd.md]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: prd.md
---

# Error Taxonomy: otposture

> Referenced by: implementer, test-writer. Exit classes per BC-6.06.001; rendering per BC-6.06.002.

## Error Categories

| Category Code | Category | Exit Class | Description |
|--------------|----------|-----------|-------------|
| VAL | Input validation | 3 | Bad user input: levels, goal IDs, rationales, labels, references |
| STORE | Store integrity/state | 4 | Corruption, schema version, init conflicts, tamper indicators |
| LOCK | Concurrency | 5 | Store in use |
| CAT | Catalog | 6 | Parse/validation/identity failures |
| SCORE | Scoring/comparability | 7 | Comparability refusals; engine integrity assertions |
| FS | Filesystem/environment | 8 | Disk full, permissions, sync-tool locks |
| RPT | Report output | 8 | Output path conflicts and safety refusals |
| (usage) | CLI usage | 2 | Unknown command/flag — handled by arg parser, no E-code |
| (internal) | Internal error | 1 | Bugs: caught panics, broken invariants |

## Error Catalog

| Error Code | Severity | Exit | Message gist / Remedy |
|-----------|----------|------|----------------------|
| E-VAL-001 | — | — | Reserved (unallocated; kept vacant to avoid renumbering — do not assign) |
| E-VAL-002 | broken | 3 | Site name contains control characters / choose a printable name |
| E-VAL-003 | broken | 3 | Label contains newline or control characters / single-line labels only |
| E-VAL-004 | broken | 3 | Unknown goal ID / nearest-match suggestions listed |
| E-VAL-005 | broken | 3 | Note exceeds 16 KiB cap / shorten or attach as evidence (post-MVP) |
| E-VAL-006 | broken | 3 | Invalid implementation level / lists valid tokens incl. unknown-vs-na distinction |
| E-VAL-007 | broken | 3 | `assess` needs a TTY / use `answer` for scripting |
| E-VAL-008 | broken | 3 | Override requires a non-empty rationale (DI-006) |
| E-VAL-009 | broken | 3 | Ambiguous snapshot reference / candidates listed by seq |
| E-VAL-010 | broken | 3 | Migration has no mapping for this version pair and `--series-break` was not passed / supply `--mapping` or explicitly opt into a series break |
| E-STORE-001 | broken | 4 | Store already exists; `init` never overwrites |
| E-STORE-002 | broken | 4 | Corrupt record at <file:record>; valid prefix loadable read-only / repair guidance |
| E-STORE-003 | degraded | 0 as load-time warning; **4 when reported by `verify`** (findings are verify's results — BC-6.06.001 Inv 2 exception) | Cached score mismatch — tamper or engine drift; details listed |
| E-STORE-004 | broken | 4 | Store written by newer otposture / upgrade the binary |
| E-STORE-005 | degraded | 0 as load-time warning; **4 when reported by `verify`** (BC-6.06.001 Inv 2 exception) | Orphan answer at load (goal absent from pinned catalog) — quarantined from scoring; listed by status/verify until removed or migrated |
| E-LOCK-001 | broken | 5 | Store in use by PID since T / wait or investigate; stale locks auto-break |
| E-CAT-001 | broken | 6 | Catalog structurally unparseable / check path & format |
| E-CAT-002 | broken | 6 | Catalog schema newer than binary / upgrade binary |
| E-CAT-003 | broken | 6 | Catalog validation failed; all defects itemized |
| E-CAT-004 | broken (warning on load) | 6 | Catalog digest mismatch for claimed version / obtain pristine catalog or migrate |
| E-SCORE-001 | broken | 1 (internal) | Engine invariant violated (e.g., weight ≤ 0 reached scoring) — bug report requested |
| E-SCORE-002 | broken | 7 | Snapshots not comparable (version/digest/series break) / `migrate` or compare within a series |
| E-FS-001 | broken | 8 | Target not writable / path + permission detail |
| E-FS-002 | broken | 8 | Disk full during write; prior state intact |
| E-FS-003 | broken | 8 | Persistent rename lock (folder-sync tool?) / retry guidance |
| E-RPT-001 | broken | 8 | Output exists; use `--force` to replace |
| E-RPT-002 | broken | 8 | Output path is a directory / suggest filename |
| E-RPT-003 | broken | 8 | Refusing to write report inside the Posture Store |

## Severity Definitions

| Severity | Meaning | Exit Impact |
|----------|---------|-------------|
| broken | Operation cannot complete; state unchanged | Non-zero (class per category) |
| degraded | Operation completed with caveats (warnings, salvage mode) | 0 with stderr warnings |
| cosmetic | Display-only imperfection | 0 |

## Recovery Strategies

- **VAL:** fix input and re-run; nothing was written (BC-3.03.001 atomicity).
- **STORE corruption:** read-only prefix salvage (BC-2.02.004); manual repair viable because the format is human-readable (BC-2.02.008); recommend git/backup in docs (R-006).
- **LOCK:** retry after holder exits; stale locks self-clear (BC-2.02.005).
- **CAT:** re-obtain pristine catalog; digests detect tampering (BC-1.01.003).
- **SCORE comparability:** `migrate` to bridge, or diff within a series (BC-2.02.006/007).
- **FS/RPT:** free space / fix path / `--force`; all writes atomic so nothing is half-done (BC-2.02.002, BC-5.05.004).
- **Internal (exit 1):** store guaranteed untouched (pure-core boundary, BC-6.06.002 EC-002); file a bug with the non-PII diagnostic block.

## User-Facing vs Internal

All E-codes above are user-facing with mandatory `help:` remedies (BC-6.06.002). Internal assertions (E-SCORE-001 class) render as apologetic exit-1 reports without stack traces unless `--debug`.
