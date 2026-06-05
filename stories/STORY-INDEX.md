# Story Index: otposture (cycle v0.1.0-greenfield)

| ID | Title | Epic | Wave | Points | Status | Dependencies | BCs |
|----|-------|------|------|--------|--------|-------------|-----|
| S-1.01 | Workspace Scaffolding & CI Gates | E-1 | 1 | 5 | draft | none | 6.06.003¹ |
| S-1.02 | Domain Types & Rational Arithmetic | E-1 | 2 | 5 | draft | S-1.01 | 4.04.005 |
| S-1.03 | Weighted Percent & Function Sub-Scores | E-1 | 3 | 8 | draft | S-1.02 | 4.04.001, 4.04.002 |
| S-1.04 | Bands, Floor Rule & Effective Tiers | E-1 | 4 | 8 | draft | S-1.03 | 4.04.003, 4.04.004 |
| S-1.05 | Delta, Drivers & Comparability Guard | E-1 | 5 | 8 | draft | S-1.04 | 4.04.006, 4.04.007 |
| S-1.06 | Next-Best-Actions Ranking | E-1 | 5 | 3 | draft | S-1.04 | 4.04.008 |
| S-2.01 | Catalog Parser & Validation | E-2 | 3 | 5 | draft | S-1.02 | 1.01.001, 1.01.002 |
| S-2.02 | Canonicalization & Content Digest | E-2 | 4 | 3 | draft | S-2.01 | 1.01.003 |
| S-2.03 | Bundled CPG 2.0 Catalog Content | E-2 | 5 | 8 | draft | S-2.02 | 1.01.004 |
| S-3.01 | Deterministic TOML Codec | E-3 | 3 | 5 | draft | S-1.02 | 2.02.008 |
| S-3.02 | Store Init, Atomic Writes & Locking | E-3 | 4 | 8 | draft | S-3.01 | 2.02.001, 2.02.002, 2.02.005 |
| S-3.03 | Append-Only Snapshots & Load Validation | E-3 | 5 | 8 | draft | S-3.02, S-1.04 | 2.02.003, 2.02.004 |
| S-3.04 | Migration — Back-Cast & Series Breaks | E-3 | 6 | 8 | draft | S-3.03, S-1.05 | 2.02.006, 2.02.007 |
| S-3.05 | Store Schema Upgrade | E-3 | 7 | 5 | draft | S-3.03 | 2.02.009 |
| S-4.01 | Answer Recording & Unknown Semantics | E-4 | 6 | 5 | draft | S-3.03, S-1.04 | 3.03.001, 3.03.003 |
| S-4.02 | Guided Assessment Session | E-4 | 7 | 5 | draft | S-4.01 | 3.03.002 |
| S-4.03 | Tier Override Lifecycle | E-4 | 7 | 3 | draft | S-4.01 | 3.03.004 |
| S-5.01 | Status View & Disclosure Chain | E-5 | 6 | 5 | draft | S-1.05, S-1.06, S-3.03 | 5.05.001 |
| S-5.02 | Diff Display | E-5 | 7 | 3 | draft | S-5.01 | 5.05.002 |
| S-5.03 | Executive HTML Report | E-5 | 8 | 8 | draft | S-5.02 | 5.05.003 |
| S-6.01 | Command Surface, Exit Codes & Errors | E-6 | 9 | 8 | draft | S-2.03, S-3.04, S-3.05, S-4.02, S-4.03, S-5.03 | 6.06.001, 6.06.002 |
| S-6.02 | JSON Output Mode | E-6 | 10 | 5 | draft | S-6.01 | 6.06.004 |
| S-6.03 | Output Safety, Offline Harness & Release | E-6 | 10 | 5 | draft | S-6.01 | 5.05.004, 6.06.003¹ |

¹ BC-6.06.003 split: S-1.01 owns the build-time deny layer; S-6.03 owns the syscall-trace layer and final verification.

## BC Coverage (Iron Law check — 33/33)

| Subsystem | BCs | Covering stories | Gaps |
|-----------|-----|-----------------|------|
| SS-01 | 1.01.001–004 | S-2.01, S-2.02, S-2.03 | none |
| SS-02 | 2.02.001–009 | S-3.01..S-3.05 | none |
| SS-03 | 3.03.001–004 | S-4.01..S-4.03 | none |
| SS-04 | 4.04.001–008 | S-1.02..S-1.06 | none |
| SS-05 | 5.05.001–004 | S-5.01..S-5.03, S-6.03 | none |
| SS-06 | 6.06.001–004 | S-6.01, S-6.02, S-1.01+S-6.03 | none |

## Totals

- **23 stories, 6 epics, 10 waves; 134 points; ~46 estimated story-days**
- Largest story: 8 points (7 stories) — none above the 13-point limit
- Largest context budget: S-2.03 at 26% — under the 30% ceiling
- tdd_mode: strict ×21, facade ×2 (S-1.01 scaffolding, S-2.03 content)
