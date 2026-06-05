# Behavioral Contract Index: otposture

> Master index of all BCs. Status: draft until adversarial review passes.
> Numbering: BC-S.SS.NNN — S = PRD section, SS = subsystem shard (`ss-NN/`), NNN sequential.

## SS-01 Framework Catalog (HIGH) — `ss-01/`

| BC | Title | Capability | Status |
|----|-------|-----------|--------|
| BC-1.01.001 | Parse Well-Formed Framework Catalog | CAP-001 | draft |
| BC-1.01.002 | Catalog Validation Rejects Defective Catalogs with Itemized Defects | CAP-001 | draft |
| BC-1.01.003 | Catalog Identity, Version Exposure, and Immutability | CAP-001 | draft |
| BC-1.01.004 | Bundled CPG 2.0 Catalog Content Correctness | CAP-001 | draft |

## SS-02 Posture Store (CRITICAL) — `ss-02/`

| BC | Title | Capability | Status |
|----|-------|-----------|--------|
| BC-2.02.001 | Posture Store Initialization | CAP-002 | draft |
| BC-2.02.002 | Atomic Store Mutations | CAP-002 | draft |
| BC-2.02.003 | Append-Only Snapshot Persistence | CAP-005 | draft |
| BC-2.02.004 | Load-Time Validation and Integrity Verification | CAP-002 | draft |
| BC-2.02.005 | Single-Writer Locking | CAP-002 | draft |
| BC-2.02.006 | Back-Cast Migration Storage | CAP-011 | draft |
| BC-2.02.007 | Series Break Recording and Enforcement | CAP-011 | draft |
| BC-2.02.008 | Deterministic Human-Readable Serialization | CAP-002 | draft |

## SS-03 Assessment (HIGH) — `ss-03/`

| BC | Title | Capability | Status |
|----|-------|-----------|--------|
| BC-3.03.001 | Record Answer | CAP-003 | draft |
| BC-3.03.002 | Guided Assessment Session | CAP-003 | draft |
| BC-3.03.003 | Unknown-by-Default Semantics | CAP-003 | draft |
| BC-3.03.004 | Tier Override Lifecycle | CAP-010 | draft |

## SS-04 Scoring Engine (CRITICAL, pure) — `ss-04/`

| BC | Title | Capability | Status |
|----|-------|-----------|--------|
| BC-4.04.001 | Weighted Percent Computation | CAP-004 | draft |
| BC-4.04.002 | Per-Function Sub-Scores | CAP-004 | draft |
| BC-4.04.003 | Maturity Band Derivation | CAP-004 | draft |
| BC-4.04.004 | Floor Rule | CAP-004 | draft |
| BC-4.04.005 | Scoring Determinism and Purity | CAP-004 | draft |
| BC-4.04.006 | Delta Computation with Driver Attribution | CAP-006 | draft |
| BC-4.04.007 | Comparability Guard | CAP-006 | draft |
| BC-4.04.008 | Next-Best-Actions Ranking | CAP-009 | draft |

## SS-05 Reporting (HIGH) — `ss-05/`

| BC | Title | Capability | Status |
|----|-------|-----------|--------|
| BC-5.05.001 | Status View | CAP-007 | draft |
| BC-5.05.002 | Diff Display | CAP-006 | draft |
| BC-5.05.003 | Executive Report Generation | CAP-008 | draft |
| BC-5.05.004 | Report Output Path Safety | CAP-008 | draft |

## SS-06 CLI Surface (MEDIUM) — `ss-06/`

| BC | Title | Capability | Status |
|----|-------|-----------|--------|
| BC-6.06.001 | Command Surface and Exit Codes | CAP-007 | draft |
| BC-6.06.002 | Error Presentation | CAP-007 | draft |
| BC-6.06.003 | Offline Guarantee | CAP-002 (cross-cutting) | draft |
| BC-6.06.004 | JSON Output Mode | CAP-007 | draft |

## Coverage Summary

- **Total BCs:** 32 (4 / 8 / 4 / 8 / 4 / 4 per subsystem)
- **Capability coverage:** CAP-001..011 all covered; CAP-012/013 deliberately uncovered (P2 schema-reserved, post-MVP)
- **VP ranges reserved:** SS-01: VP-001..009 · SS-02: VP-010..029 · SS-03: VP-030..039 · SS-04: VP-040..069 · SS-05: VP-070..079 · SS-06: VP-080..089
