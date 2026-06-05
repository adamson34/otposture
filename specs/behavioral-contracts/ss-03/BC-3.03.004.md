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
subsystem: "SS-03"
capability: "CAP-010"
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

# Behavioral Contract BC-3.03.004: Tier Override Lifecycle

## Description

`override` reclassifies a Goal's Criticality Tier for this Site, with a mandatory rationale (DI-006). Overrides are auditable, reversible (by a new override or explicit clear — also rationale'd), and disclosed in every surface that shows a band. The anti-gaming design: you can change the rules, but never quietly.

## Preconditions

1. Store initialized, lock held; goal exists in pinned catalog; target tier ∈ {Critical, High, Standard}; rationale is non-empty after trimming.

## Postconditions

1. Active override recorded with goal_id, tier, rationale, created_at; at most one active override per goal (a new one supersedes, with the superseded record retained as history).
2. Effective tier = active override else catalog default; next scoring reflects it; existing snapshots' cached Scores unchanged (events.md ordering rule 4).
3. Clearing an override (`override 1.A --clear --rationale "..."`) restores the catalog default, also as a recorded event.

## Invariants

1. Override records are append-only history like snapshots (supersede, never edit).
2. Rationale is documented non-PII free text (DI-013) and is rendered wherever the override's effect is visible (DEC-005).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Override demotes a Critical goal currently triggering the floor rule | Permitted; band may rise; status/report show "floor rule cleared by override: <goal> <rationale>" (DEC-005) |
| EC-002 | Override to the same tier as catalog default | Accepted and recorded (documents a considered decision); effective behavior unchanged |
| EC-003 | Rationale of only whitespace | E-VAL-008 — rationale required (DI-006) |
| EC-004 | Override on a goal later dropped by catalog migration | Preserved in originals; orphaned-override notice in migration report (DEC-012) |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| `override 3.H standard --rationale "air-gapped site, no remote access exists"` | Effective tier Standard; floor recalculated next score; disclosure in status | happy-path |
| Same goal overridden twice | Second active; first retained as superseded history | edge-case |
| `override 3.H standard` (no rationale) | E-VAL-008, nothing recorded | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-035 | Effective-tier resolution: override iff active, else default — for all goals and override histories | proptest |
| VP-036 | Every surface displaying a floor-rule state with an active relevant override also displays that override | manual/snapshot tests |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-010 |
| L2 Domain Invariants | DI-006, DI-013 |
| L2 Edge Cases | DEC-005 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-4.04.004 — composes with (floor rule consumes effective tiers)
- BC-5.05.003 — composes with (report disclosure)
