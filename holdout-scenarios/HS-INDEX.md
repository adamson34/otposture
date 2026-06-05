# Holdout Scenario Index: otposture (cycle v0.1.0-greenfield)

> **NEVER expose scenario contents to implementer or test-writer agents.**
> Evaluated by the holdout-evaluator at the wave gates listed below.

| ID | Title | Category | Priority | Epic | Evaluate at wave | BCs |
|----|-------|----------|----------|------|------------------|-----|
| HS-001 | Floor rule not averaged away | behavioral-subtleties | must-pass | E-1 | 6 (first full-surface wave) | 4.04.003/004 |
| HS-002 | Delta attribution incl. regression | behavioral-subtleties | must-pass | E-1 | 7 | 4.04.006, 5.05.002 |
| HS-003 | Kill mid-assessment | edge-case-combinations | must-pass | E-4 | 7 | 3.03.002, 2.02.002/005 |
| HS-004 | Cross-version honesty (breaks/back-cast) | integration-boundaries | must-pass | E-3 | 7 | 4.04.007, 2.02.006/007 |
| HS-005 | Hand-edited store (legit/clumsy/malicious) | edge-case-combinations | must-pass | E-3 | 9 (verify exit semantics live) | 2.02.004, 6.06.001 |
| HS-006 | Air-gap parity | security-probes | must-pass | E-6 | 10 | 6.06.003, 5.05.003 |
| HS-007 | Exec report vs hostile content + re-runs | security-probes | must-pass | E-5 | 10 | 5.05.003/004 |
| HS-008 | Override transparency (anti-gaming) | behavioral-subtleties | must-pass | E-4 | 8 | 3.03.004, 5.05.x |
| HS-009 | Honest zero state → first climb | behavioral-subtleties | must-pass | E-5 | 9 | 3.03.003, 5.05.001/003 |
| HS-010 | Catalog tamper detection | security-probes | must-pass | E-2 | 9 | 1.01.003, 4.04.007 |

## Coverage Notes

- 10 scenarios, all must-pass; categories: behavioral-subtleties ×4, security-probes ×3, edge-case-combinations ×2, integration-boundaries ×1.
- Risk linkage: R-002 (HS-001, HS-008), R-003 (HS-004), R-004 (HS-010), R-005 (HS-002), R-006 (HS-003, HS-005); ASM-004 (HS-009).
- Every scenario phrases behavior black-box (CLI in, observable out) and differently from its source BC text — derived from assumptions/risks/failure-modes per the L2 cross-reference table.
