---
document_type: domain-spec-section
level: L2
section: "differentiators"
version: "1.0"
status: draft
producer: business-analyst
timestamp: 2026-06-05T12:10:44
phase: 1a
inputs: [product-brief.md, planning/research-report.md]
input-hash: "d0490fef4b5ce75c2b7fc3c0e135aeb5"
traces_to: L2-INDEX.md
---

# Competitive Differentiator Traceability

> **Sharded L2 section (DF-021).** Navigate via `L2-INDEX.md`.
> Maps each claimed differentiator to the capabilities/invariants that deliver it.
> If a differentiator has no capability backing, it's marketing, not engineering.

| Differentiator | Why It Matters | Supporting Capabilities |
|----------------|---------------|----------------------|
| **The ticker between assessments** — posture updated in minutes-scale sessions, not annual events | The core thesis: posture changes at operational cadence, but CSET/consultant assessments are point-in-time events stale within weeks (research Q3) | CAP-003 (incremental assessment), CAP-005 (snapshots), CAP-006 (diff), CAP-007 (status); ASM-004 (<15 min target) |
| **Defensible number** — band-first headline that cannot hide a fatal gap and cannot move without naming why | Counters the known failure modes of single-number scores that make CISOs distrust them (research Q2); survives skeptical scrutiny | CAP-004 + DI-003 (floor rule), DI-005 (driver attribution), DI-012 (single formula), DEC-010 (band/% divergence handled) |
| **Transparent, reproducible scoring** vs. proprietary commercial risk scores | Claroty/Dragos/Nozomi scores are unpublishable black boxes; otposture's weights/thresholds are inspectable data a user can re-derive by hand | DI-008 (parameters as data), DI-010 (determinism), CAP-001 (catalog as artifact) |
| **Self-writing exec one-pager** — the weekly/monthly C-suite update generates itself | The persona's sharpest pain: reporting upward is unpaid overhead; the deliverable IS the report (brainstorm) | CAP-008 (exec report), CAP-009 (next best actions), CAP-007 (status) |
| **Offline, no-footprint, data-sovereign** — no agents, appliances, cloud, or telemetry, ever | OT data sovereignty is a buying criterion; commercial platforms require footprint and budget the persona lacks; works air-gapped | DI-007 (hard boundary), FM-001..008 (plant-workstation resilience), single static binary (brief constraint) |
| **Git-friendly versioned posture history** | "Now vs. 6 months ago" is undiffable in PowerPoint/Excel; otposture history is plain-text, append-only, and survives catalog upgrades honestly | CAP-005 + DI-002 (append-only), DI-001 (version pinning), CAP-011 (back-cast / series break) |
| **Honest history** — never silently rescored, regressions shown at full prominence | Bitsight-style silent rubric shifts are the canonical credibility killer (research Q2); honesty is a feature competitors can't easily copy | DI-002, DEC-003 (series break), DEC-009 (negative drivers), DEC-005 (override disclosure) |
| **Free & open source** for the persona with no budget | 73% of orgs don't put OT budget under a CISO; small/mid plants can't fund appliances; CISA's free assessment capacity is shrinking (research Q3) | Entire product; DI-009 keeps the licensing posture clean |

## Differentiation Threats (from research)

| Threat | Watch Signal | Response Lever |
|--------|-------------|----------------|
| CSET CPG 2.0 module (Q1 2026) + Trend & Compare (R-001) | CSET release notes adding continuous/lightweight reassessment | Lean harder on cadence, driver attribution, exec report; CSET import |
| Evidence/staleness decay is the moat but is post-MVP | Early users asking "how do I prove this answer?" | Accelerate CAP-012 from the reserved schema |

> **PRD Seeding Note:** every row above must map to BC-NNN behavioral contracts in the PRD; a differentiator without contract backing is dropped from marketing claims.
