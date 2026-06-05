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
subsystem: "SS-06"
capability: "CAP-007"
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

# Behavioral Contract BC-6.06.002: Error Presentation

## Description

Every user-facing error follows one format: stable code, one-line cause, affected element, and a concrete remedy. The persona is time-poor — errors must be self-service fixable without reading docs.

## Preconditions

1. Any error from the taxonomy reaches the CLI boundary.

## Postconditions

1. Format on stderr: `error[E-XXX-NNN]: <cause>` + `  --> <element>` (when applicable) + `  help: <remedy>` (always present).
2. Multi-defect errors (e.g., catalog validation) render each defect in that format, then a count summary.
3. Internal errors (exit 1) apologize, name the store as untouched/safe, and ask for a bug report with the (non-PII) diagnostic block — never a bare panic/stack trace by default (`--debug` opts into backtraces).

## Invariants

1. Error text never includes store free-text content beyond the minimal identifying excerpt (notes/rationales aren't echoed at length — DI-013 hygiene).
2. Remedies reference real commands/flags that exist (CI-checked against the command surface).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Error while stderr is not a TTY | Same format minus color/decoration |
| EC-002 | Panic in the pure core (engine bug) | Caught at boundary; rendered as exit-1 internal error with safe-state assurance |
| EC-003 | Very long path/element names | Wrapped, never truncated to ambiguity |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| `answer 9.Q full` | `error[E-VAL-004]` + suggestion help line | happy-path (of the error path) |
| Catalog with 3 defects | 3 formatted defects + summary | happy-path |
| Injected engine panic | exit 1; formatted internal-error block; no raw backtrace | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-082 | Every taxonomy code renders with a non-empty help line | CI exhaustive render test |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-007 |
| L2 Domain Invariants | DI-013 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-6.06.001 — composes with (exit classes)
