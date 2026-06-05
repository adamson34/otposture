---
document_type: adversarial-review-index
level: ops
version: "1.0"
status: in-review
producer: adversary
timestamp: 2026-06-05T13:20:00
phase: 1d
pass: 1
inputs: [product-brief.md, "domain-spec/", prd.md, "prd-supplements/", "behavioral-contracts/"]
traces_to: prd.md
total_findings: 10
severity_distribution: { CRIT: 1, HIGH: 3, MED: 4, LOW: 2 }
---

# Adversarial Review -- Pass 1

## Finding Catalog

| ID | Severity | Category | Title | Status | Depends On | Blocks |
|----|----------|----------|-------|--------|-----------|--------|
| ADV-P1GREEN-P01-CRIT-001 | CRITICAL | contradictions | Orphan-answer contract contradiction (entry vs load) | open | -- | story-decomposition |
| ADV-P1GREEN-P01-HIGH-001 | HIGH | interface-gaps | `store upgrade` phantom command outside fixed surface | open | -- | -- |
| ADV-P1GREEN-P01-HIGH-002 | HIGH | contradictions | `assess --only` token/flag mismatch + out-of-scope `stale` | open | -- | -- |
| ADV-P1GREEN-P01-HIGH-003 | HIGH | contradictions | JSON error on stdout violates stdout/stderr separation | open | -- | -- |
| ADV-P1GREEN-P01-MED-001 | MEDIUM | verification-gaps | NBA ranking mandates `impact` metadata it never consumes | open | -- | -- |
| ADV-P1GREEN-P01-MED-002 | MEDIUM | ambiguous-language | `migrate --series-break` non-interactive semantics undefined | open | -- | -- |
| ADV-P1GREEN-P01-MED-003 | MEDIUM | contradictions | Goal-ID splitting collides with exact-ID-set / 34-goal golden | open | -- | -- |
| ADV-P1GREEN-P01-MED-004 | MEDIUM | spec-fidelity | Brief floor-rule wording omits `Unknown` trigger | open | -- | -- |
| ADV-P1GREEN-P01-LOW-001 | LOW | spec-fidelity | L2-INDEX caption "DI-001..012" vs DI-001..013 | open | -- | -- |
| ADV-P1GREEN-P01-LOW-002 | LOW | ambiguous-language | `verify` write-nothing guarantee not stated | open | -- | -- |

## Dependency Graph

```text
All findings are independent (no inter-finding blocking).
CRIT-001, HIGH-001, HIGH-002 should resolve before Phase 2 story decomposition
(they change command/flag/state contracts).
```

## Category Groups

| Category | Finding IDs | Can Triage in Parallel? |
|----------|------------|------------------------|
| contradictions | CRIT-001, HIGH-002, HIGH-003, MED-003 | Yes |
| interface-gaps | HIGH-001 | Yes |
| verification-gaps | MED-001 | Yes |
| ambiguous-language | MED-002, LOW-002 | Yes |
| spec-fidelity | MED-004, LOW-001 | Yes |
