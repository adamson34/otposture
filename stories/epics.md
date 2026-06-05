# Epics: otposture v0.1.0-greenfield

> BC → epic grouping follows the crate topology (ARCH-INDEX Subsystem Registry).
> Wave order follows dependency-graph.md (op-score first; op-cli last).

## E-1: Scoring Core & Foundation
- **Goal:** The provable heart — a user can trust that any number otposture shows is exact, deterministic, and floor-ruled. Plus the workspace/CI skeleton everything builds in.
- **BCs:** BC-4.04.001..008, BC-6.06.003 (deny-layer), (BC-3.03.004's effective-tier resolution)
- **Subsystems touched:** SS-04 (op-score), workspace infra
- **Stories:** S-1.01..S-1.06 (6)

## E-2: Framework Catalog
- **Goal:** CPG 2.0 as validated, digest-pinned data — the user assesses against the real framework, legally embedded.
- **BCs:** BC-1.01.001..004
- **Subsystems touched:** SS-01 (op-catalog)
- **Stories:** S-2.01..S-2.03 (3)

## E-3: Posture Store
- **Goal:** Posture history that survives plant workstations — atomic, append-only, human-readable, migratable, never silently rewritten.
- **BCs:** BC-2.02.001..009
- **Subsystems touched:** SS-02 (op-store)
- **Stories:** S-3.01..S-3.05 (5)

## E-4: Assessment
- **Goal:** The minutes-scale answering loop — guided, resumable, kill-safe, with auditable tier overrides.
- **BCs:** BC-3.03.001..004
- **Subsystems touched:** SS-03 (op-assess), SS-06 (TTY shell)
- **Stories:** S-4.01..S-4.03 (3)

## E-5: Reporting
- **Goal:** The deliverables — honest status/diff views and the self-writing exec one-pager.
- **BCs:** BC-5.05.001..003
- **Subsystems touched:** SS-05 (op-render)
- **Stories:** S-5.01..S-5.03 (3)

## E-6: CLI & Distribution
- **Goal:** The 11-command surface with contract exit codes, JSON twins, the audited offline guarantee, and copy-one-file distribution.
- **BCs:** BC-6.06.001, BC-6.06.002, BC-6.06.004, BC-6.06.003 (trace-layer), BC-5.05.004
- **Subsystems touched:** SS-06 (op-cli)
- **Stories:** S-6.01..S-6.03 (3)

**Total: 6 epics, 23 stories, 10 waves.** BC coverage: 33/33 (see STORY-INDEX.md coverage table).
