# Wave Schedule: otposture v0.1.0-greenfield

> 10 waves, 23 stories, 134 points. Wave gates (full suite + adversarial diff review +
> holdout evaluation) run between waves per /vsdd-factory:wave-gate.

| Wave | Stories | Points | Theme | Gate focus |
|------|---------|--------|-------|-----------|
| 1 | S-1.01 | 5 | Workspace + CI gates | deny/purity/dep-graph gates green on empty workspace |
| 2 | S-1.02 | 5 | Shared types + rationals | float-free op-score; type invariants |
| 3 | S-1.03, S-2.01, S-3.01 | 18 | Formula · parser · codec | first Kani proofs (VP-040/041/044); determinism properties |
| 4 | S-1.04, S-2.02, S-3.02 | 19 | Floor rule · digest · atomicity | VP-046..050 proofs; fault-injection matrix green |
| 5 | S-1.05, S-1.06, S-2.03, S-3.03 | 27 | Delta/guard · NBA · CPG content · snapshots | flagship VP-053 proof; CPG content human review; HS-001/002/010 |
| 6 | S-3.04, S-4.01, S-5.01 | 18 | Migration · answers · status | HS-004/009; disclosure chain snapshots |
| 7 | S-3.05, S-4.02, S-4.03, S-5.02 | 16 | Upgrade · session · overrides · diff | HS-003/008; kill-injection |
| 8 | S-5.03 | 8 | Exec report | HS-007; print review (manual gate) |
| 9 | S-6.01 | 8 | Full command surface | exit-code matrix; help-vs-spec goldens; HS-005 |
| 10 | S-6.02, S-6.03 | 10 | JSON · offline audit · release | HS-006; schema validation; 4-target release dry-run |

## Resource Notes

- Solo maintainer + factory agents: waves 3–7 carry 3–4 parallel stories — the factory delivers them sequentially or in parallel worktrees as capacity allows; wave boundaries are the sync points, not story start times.
- Critical path: E-1 chain (waves 1–5) → E-5 chain (6–8) → E-6 (9–10) ≈ 21 serial days.
- S-2.03 (CPG content) is the only story needing the human review gate mid-wave (verbatim-text spot check).

## Wave-Gate Reminders

- Holdout scenarios are evaluated at the wave listed in HS-INDEX.md — never shown to implementer/test-writer agents.
- DTU validation: not applicable (no third-party service clones in this product).
- Demo evidence: VHS terminal recordings from wave 6 onward (status/diff/assess); HTML report artifact from wave 8.
