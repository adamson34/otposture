---
document_type: behavioral-contract
level: L3
version: "1.1"
status: draft
producer: product-owner
timestamp: 2026-06-05T12:50:04
phase: 1a
inputs: [domain-spec/L2-INDEX.md, product-brief.md]
input-hash: "317fa65bf3226b80d4229c4e2ac2e6ba"
traces_to: domain-spec/L2-INDEX.md
origin: greenfield
subsystem: "SS-04"
capability: "CAP-009"
lifecycle_status: active
introduced: v0.1.0
modified: []
deprecated: null
deprecated_by: null
replacement: null
retired: null
removed: null
removal_reason: null
---

# Behavioral Contract BC-4.04.008: Next-Best-Actions Ranking

## Description

Ranks improvable goals by `potential_gain / effort` using static catalog metadata (v1 decision: no user-supplied estimates). Answers the persona's "where do I start" overwhelm with a defensible, explainable ordering.

## Preconditions

1. Current Score computed; catalog provides impact/effort metadata per goal (validated by BC-1.01.002 rule h).

## Postconditions

1. Candidate set = goals with level < FI and level ≠ N/A.
2. Each candidate scored: `potential_gain` = percent increase if raised to FI (computed via the real formula, not approximated); `effort_rank` from catalog Ease metadata (Simple=1, Moderate=2, Complex=3); priority = potential_gain / effort_rank.
3. Output: top-N (default 5) with goal, current level, potential gain (display-rounded), effort label, and CISA impact rating (displayed for context; **not** part of the v1 ranking formula — `potential_gain` already encodes weight-adjusted score impact, and double-counting impact would be indefensible). When the floor rule is active, gating critical goals are **always listed first regardless of ratio** (closing the floor is categorically the next best action).
4. Ties broken deterministically: higher potential_gain, then catalog order.

## Invariants

1. Pure computation; same inputs ⇒ same ranking (DI-010 extension).
2. Ranking inputs are entirely inspectable catalog data (DI-008) — a user can re-derive the ordering.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Everything FI (perfect score) | Empty list with "nothing to improve" |
| EC-002 | Unknown-heavy store | Unknown goals are candidates (level value 0, often top-ranked) — assessing them IS the next best action; output says so |
| EC-003 | Floor active with 3 critical gaps | All 3 head the list flagged "floor-gating", then ratio-ranked rest |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Mixed store, no floor | Top-5 by gain/effort; deterministic | happy-path |
| One Simple goal NI (w large) vs one Complex goal NI (w small) | Simple/large-gain ranks first | happy-path |
| Floor active | Gating goals first, flagged | edge-case |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-058 | Implementing any listed recommendation (raising that goal to FI, all else fixed) yields a percent gain exactly equal to its stated potential_gain — rank-position-independent, so floor-gating reordering cannot falsify it | proptest |
| VP-059 | Ranking determinism incl. tie-breaks | proptest |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-009 |
| L2 Domain Invariants | DI-008, DI-010 |
| L2 Assumptions | ASM-005 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-1.01.004 — depends on (CISA Cost/Impact/Ease metadata)
- BC-5.05.001/003 — blocks (rendered in status and exec report)
