# Story Dependency Graph: otposture

> Mirrors the crate topology (architecture/dependency-graph.md). Verified acyclic:
> every edge points from a higher wave to a lower wave.

## Dependencies (story → prerequisites)

```
S-1.02 → S-1.01
S-1.03 → S-1.02
S-1.04 → S-1.03
S-1.05 → S-1.04
S-1.06 → S-1.04
S-2.01 → S-1.02
S-2.02 → S-2.01
S-2.03 → S-2.02
S-3.01 → S-1.02
S-3.02 → S-3.01
S-3.03 → S-3.02, S-1.04
S-3.04 → S-3.03, S-1.05
S-3.05 → S-3.03
S-4.01 → S-3.03, S-1.04
S-4.02 → S-4.01
S-4.03 → S-4.01
S-5.01 → S-1.05, S-1.06, S-3.03
S-5.02 → S-5.01
S-5.03 → S-5.02
S-6.01 → S-2.03, S-3.04, S-3.05, S-4.02, S-4.03, S-5.03
S-6.02 → S-6.01
S-6.03 → S-6.01
```

## Independent Groups (wave-parallel sets)

```
[S-1.01]                              # Wave 1  — foundation
[S-1.02]                              # Wave 2  — shared types
[S-1.03, S-2.01, S-3.01]              # Wave 3  — three crates fan out
[S-1.04, S-2.02, S-3.02]              # Wave 4
[S-1.05, S-1.06, S-2.03, S-3.03]      # Wave 5  — widest wave
[S-3.04, S-4.01, S-5.01]              # Wave 6
[S-3.05, S-4.02, S-4.03, S-5.02]      # Wave 7  (S-3.05 dep-eligible for W6; placed W7 for load balance)
[S-5.03]                              # Wave 8
[S-6.01]                              # Wave 9  — integration choke point (by design: thin shell wired last)
[S-6.02, S-6.03]                      # Wave 10
```

## Acyclicity Verification

Topological order exists (waves above). Programmatic check: every `depends_on` entry has `wave(dep) < wave(story)` — validated in STORY-INDEX coverage table. No story depends on a same-or-later-wave story.

## Critical Path

S-1.01 → S-1.02 → S-1.03 → S-1.04 → S-1.05 → S-5.01 → S-5.02 → S-5.03 → S-6.01 → S-6.02/03
(10 waves; ~21 estimated days of serial work on the critical path, ~46 story-days total)
