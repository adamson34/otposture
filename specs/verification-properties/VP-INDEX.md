# Verification Property Index: otposture

> **Sharding model:** all 67 VPs are *defined* in their owning BCs
> (`specs/behavioral-contracts/ss-NN/`). The 10 formal (Kani) targets additionally
> have standalone L4 files here with proof harness skeletons and feasibility
> assessments — they are the proof-lock artifacts for Phase 6 formal hardening.
> Module/method mapping for every VP: `architecture/verification-coverage-matrix.md`.

## Formal proof targets (standalone files in this directory)

| VP | Property | Source BC | Module | File |
|----|----------|-----------|--------|------|
| VP-040 | Percent bounds | BC-4.04.001 | op-score | VP-040.md |
| VP-041 | Single-level monotonicity | BC-4.04.001 | op-score | VP-041.md |
| VP-044 | Decomposition identity | BC-4.04.002 | op-score | VP-044.md |
| VP-046 | Band monotonicity | BC-4.04.003 | op-score | VP-046.md |
| VP-047 | Band totality | BC-4.04.003 | op-score | VP-047.md |
| VP-048 | Floor soundness | BC-4.04.004 | op-score | VP-048.md |
| VP-049 | Floor completeness | BC-4.04.004 | op-score | VP-049.md |
| VP-050 | Critical-gap exactness | BC-4.04.004 | op-score | VP-050.md |
| VP-053 | Delta reconciliation (flagship) | BC-4.04.006 | op-score | VP-053.md |
| VP-056 | Comparability-guard soundness | BC-4.04.007 | op-score | VP-056.md |

## BC-defined VPs (no standalone file; definition lives in the BC)

| Range | Subsystem | Method mix | Owning BCs |
|-------|-----------|-----------|------------|
| VP-001..008 | SS-01 Catalog | proptest, fuzz, kani (VP-004), CI goldens | BC-1.01.001..004 |
| VP-010..027 | SS-02 Store | proptest, fault-injection, fuzz, audit | BC-2.02.001..009 |
| VP-030..036 | SS-03 Assessment | proptest, kill-injection, snapshots | BC-3.03.001..004 |
| VP-042..045, 051..052, 054..055, 057..059 | SS-04 Scoring (non-formal) | proptest, CI lint/golden, review | BC-4.04.001..008 |
| VP-070..076 | SS-05 Reporting | snapshot/golden, fuzz, fault-injection | BC-5.05.001..004 |
| VP-080..086 | SS-06 CLI | CI matrix, trace, schema validation | BC-6.06.001..004 |

## Status summary

| Status | Count |
|--------|-------|
| draft (harness not yet written) | 67 |
| in-development | 0 |
| verified (locked) | 0 |
