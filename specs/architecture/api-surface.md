---
document_type: architecture-section
level: L3
section: "api-surface"
version: "1.0"
status: draft
producer: architect
timestamp: 2026-06-05T14:58:33
phase: 1b
inputs: [domain-spec/L2-INDEX.md, prd.md]
input-hash: "1ccddf836bc64b23fe870d3dfd9df37d"
traces_to: ARCH-INDEX.md
---

# API Surface

## External surface (the product contract)

Authoritative spec: `prd-supplements/interface-definitions.md` (11 commands, exit classes 0–8, JSON schemas). Architecture adds nothing to it; op-cli implements it 1:1. The exec HTML report and the `--json` documents are the only machine-consumable artifacts; both are projections of the same computed structures (BC-6.06.004 Inv 1).

## Internal crate APIs (story/test anchors)

### op-score (pure — every function below is a Kani/proptest candidate)

```rust
pub fn score(catalog: &Catalog, answers: &AnswerSet, overrides: &TierOverrides) -> ScoreResult;
pub fn delta(catalog: &Catalog, from: &FrozenAssessment, to: &FrozenAssessment) -> Delta;
pub fn check_comparability(a: &SnapshotMeta, b: &SnapshotMeta, breaks: &[SeriesBreak]) -> Result<EffectivePair, NotComparable>;
pub fn next_best_actions(catalog: &Catalog, score: &ScoreResult, n: usize) -> Vec<Action>;
pub fn effective_tier(goal: &Goal, overrides: &TierOverrides) -> Tier;
pub fn display_round(delta: &Delta) -> DisplayDelta;   // largest-remainder reconciliation
```

`ScoreResult` carries: `percent: Option<Rational>`, `band: BandRef`, `band_uncapped: BandRef`, `floor: FloorStatus { triggered, gaps }`, `functions: Vec<FunctionScore>`, counts (unknown/na/assessed/total). All fields derivable, none settable independently (DI-010).

### op-catalog

```rust
pub fn parse(toml_src: &str) -> Result<Catalog, ParseError>;
pub fn validate(c: &Catalog) -> Result<ValidCatalog, Vec<Defect>>;   // 8 rules, all defects
pub fn digest(c: &ValidCatalog) -> ContentDigest;                     // canonicalized SHA-256
pub fn bundled() -> &'static ValidCatalog;                            // CPG 2.0, build-time validated
```

### op-store

```rust
// codec (pure)
pub fn emit(state: &StoreState) -> String;            // deterministic (VP-022)
pub fn parse(files: &StoreFiles) -> Result<LoadedStore, LoadReport>;  // salvage-aware
// fs (effectful)
pub fn init(path, site, catalog) -> Result<Store>;
pub fn open(path) -> Result<Store, LoadReport>;       // read-only on damage
pub fn record_answer(&mut self, ...) -> Result<()>;   // atomic per answer
pub fn take_snapshot(&mut self, score, label) -> Result<Seq>;
pub fn record_override(&mut self, ...) -> Result<()>;
pub fn migrate(&mut self, new_catalog, mapping: Option<Mapping>, allow_break: bool) -> Result<MigrationReport>;
pub fn upgrade_schema(&mut self) -> Result<UpgradeReport>;   // backup-first (BC-2.02.009)
pub fn verify(&self) -> VerifyReport;                 // read-only
```

### op-assess

```rust
pub struct SessionState { ... }
pub enum SessionInput { Answer(Level, Option<Note>), Skip, Back, Quit }
pub fn step(state: SessionState, input: SessionInput) -> (SessionState, Vec<Effect>);
pub fn normalize_goal_id(catalog: &Catalog, raw: &str) -> Result<GoalId, Suggestion>;
```

### op-render

```rust
pub fn status(view: &StatusView) -> String;           // terminal
pub fn diff(view: &DiffView) -> String;
pub fn exec_report(view: &ReportView) -> String;      // self-contained HTML, escaped
pub fn disclosures(score: &ScoreResult, store: &LoadedStore) -> Disclosures;  // shared chain
```

## Stability promises

- op-score function signatures are **frozen after wave 1** (proof harnesses bind to them).
- The `--json` schema_version governs external machine consumers; internal APIs may change freely until v1.0.
