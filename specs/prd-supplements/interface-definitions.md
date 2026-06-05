---
document_type: prd-supplement-interface-definitions
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

# Interface Definitions: otposture

> Referenced by: implementer, test-writer, devops-engineer. Authoritative command surface per BC-6.06.001.

## CLI Interface

```
otposture — track OT security posture, offline

USAGE: otposture <COMMAND> [OPTIONS]

COMMANDS:
  init      Create a posture store for one site
              --site <NAME> (required)  --sector <SECTOR>  --catalog <PATH> (default: bundled CPG 2.0)
  assess    Guided interactive assessment (TTY required)
              --function <FN>  --only <unknown|all> (default: unknown)
  answer    Record one answer non-interactively
              <GOAL_ID> <LEVEL: not|partial|large|full|na|unknown>  --note <TEXT ≤16KiB>
  override  Reclassify a goal's criticality tier for this site
              <GOAL_ID> <TIER: critical|high|standard>  --rationale <TEXT> (required)  --clear
  snapshot  Freeze current assessment + score into history
              --label <SINGLE_LINE>
  status    Current band/%, trend, movers, gaps, next actions   [--json]
  diff      Compare two snapshots
              <FROM> <TO>  (seq | label | YYYY-MM-DD)  --top <N>  [--json]
  report    Generate executive one-pager
              --exec  --out <PATH> (default: ./otposture-report-<date>-s<seq>.html)
              --from <REF> --to <REF> (default: previous vs latest)  --force
  verify    Recompute and check all cached scores & integrity    [--json]
              (read-only: reports mismatches and orphans, never repairs or rewrites)
  migrate   Move store to a new catalog version
              --catalog <PATH>  --mapping <PATH>  --series-break (explicit opt-in when unmappable)
              (no mapping + no --series-break => E-VAL-010 refusal; breaks are never implicit)
  upgrade   Migrate the store schema to this binary's version (explicit, in-place, prior files backed up)

GLOBAL: --store <DIR> (default: ./)  --debug  --no-color  --help  --version
```

## Exit Code Semantics

| Code | Meaning | Source |
|------|---------|--------|
| 0 | Success (incl. "no change", warnings) | — |
| 1 | Internal error (bug) | caught panics, E-SCORE-001 |
| 2 | CLI usage error | arg parser |
| 3 | Input validation | E-VAL-* |
| 4 | Store integrity/state | E-STORE-* |
| 5 | Lock contention | E-LOCK-* |
| 6 | Catalog error | E-CAT-* |
| 7 | Comparability refusal | E-SCORE-002 |
| 8 | Filesystem / report output | E-FS-*, E-RPT-* |

## JSON Output Schema (outline — full JSON Schema published with the binary)

```json
{
  "schema_version": "1.0.0",
  "site": {"name": "...", "sector": "..."},
  "catalog": {"id": "cisa-cpg", "version": "2.0.0", "digest": "sha256:..."},
  "score": {
    "scoreable": true,
    "percent": 0.6765,
    "band": "developing",
    "band_uncapped": "managed",
    "floor_rule": {"triggered": true, "gaps": [{"goal": "3.H", "level": "unknown", "tier_source": "default"}]},
    "functions": [{"id": "govern", "percent": 0.55, "scoreable": true}],
    "unknown_count": 4, "na_count": 1, "assessed": 30, "total": 34
  },
  "delta": {
    "from_seq": 3, "to_seq": 5, "percent_delta": 0.08,
    "band_change": {"from": "foundational", "to": "developing"},
    "drivers": [{"goal": "2.E", "from": "not", "to": "full", "points": 0.0294}],
    "composition_adjustment": 0.0,
    "derived_via_backcast": false
  },
  "overrides": [{"goal": "3.N", "tier": "high", "rationale": "...", "created_at": "..."}],
  "series_breaks": [{"at_seq": 12, "from_version": "2.0.0", "to_version": "2.1.0", "bridged": false}],
  "next_actions": [{"goal": "3.D", "potential_gain": 0.06, "effort": "simple", "impact": "high", "floor_gating": false}]
}
```

`verify --json` emits a distinct check-results document (not the score/delta projection):

```json
{
  "schema_version": "1.0.0",
  "kind": "verify-results",
  "snapshots_checked": 12,
  "score_mismatches": [{"seq": 7, "field": "percent", "cached": "2/3", "recomputed": "1/2", "engine_version_at_write": "0.3.0"}],
  "orphan_answers": [{"goal": "2.F", "reason": "absent-from-pinned-catalog"}],
  "integrity_errors": [],
  "ok": false
}
```

Error shape (any command with `--json`): `{"error": {"code": "E-SCORE-002", "message": "...", "help": "..."}}`

No personal-data fields exist in any schema (DI-013). Numbers full-precision; `percent: null` + `scoreable: false` when undefined.

## Store Layout (format chosen at architecture; constraints per BC-2.02.008)

```
<store>/
  otposture.toml|yaml      # marker + site + schema_version + catalog pin
  assessment.*             # current answers (one block per goal)
  snapshots.*              # append-only records (seq-ordered)
  overrides.*              # append-only override history
  migrations.*             # back-cast records, series breaks, mapping digests
  .lock                    # advisory lock (PID, host, start time)
```

All text, LF endings, deterministic serialization, header comments with schema version and non-PII notice.

## Catalog File (OSCAL-shaped; outline)

```yaml
catalog_id: cisa-cpg
version: 2.0.0
framework_name: "CISA Cross-Sector Cybersecurity Performance Goals"
licensing_provenance: "CPG Report 2.0 (Dec 2025), TLP:CLEAR, retrieved 2026-06-05"
bands:                      # ordered low->high; N >= 2; first band has no threshold (starts at 0)
  - name: foundational      # thresholds: rationale required (PRD OQ2)
  - {name: developing, threshold: 0.40}
  - {name: managed, threshold: 0.65}
  - {name: resilient, threshold: 0.85}
functions: [{id: govern, name: GOVERN, order: 1}, ...]
goals:
  - goal_id: "1.A"
    title: "Establish Cybersecurity Responsibilities"
    function: govern
    weight: 1.0
    default_tier: high
    tier_rationale: "..."
    impact: high          # CISA rating
    cost: low             # CISA rating
    ease: simple          # CISA rating
    outcome: "verbatim CPG outcome text..."
    ot_note: "verbatim OT-specific guidance..."
```

## Flag Interactions

| Flag A | Flag B | Interaction | Resolution |
|--------|--------|-------------|-----------|
| `--json` | `--no-color` | JSON implies no decoration | `--json` wins; `--no-color` redundant |
| `report --force` | output path in store dir | Conflict | E-RPT-003 refusal wins over `--force` |
| `migrate --mapping` | `--series-break` | Mutually exclusive intents | Usage error (exit 2) if both |
| `assess --only all` | `--function X` | Compose | Intersection: all goals of function X |
