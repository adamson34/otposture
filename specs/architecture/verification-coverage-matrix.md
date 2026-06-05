---
document_type: architecture-section
level: L3
section: "verification-coverage-matrix"
version: "1.0"
status: draft
producer: architect
timestamp: 2026-06-05T14:58:33
phase: 1b
inputs: [domain-spec/L2-INDEX.md, prd.md]
input-hash: "1ccddf836bc64b23fe870d3dfd9df37d"
traces_to: ARCH-INDEX.md
---

# Verification Coverage Matrix

> VP definitions live in their owning BCs (behavioral-contracts/) — this matrix maps
> them to modules and methods. The 10 formal (Kani) targets additionally have
> standalone VP files in `specs/verification-properties/` with proof harness skeletons.

| Module | Kani (formal) | proptest | fuzz | fault-injection | CI gates | Coverage notes |
|--------|--------------|----------|------|-----------------|----------|----------------|
| op-score | VP-040, 041, 044, 046, 047, 048, 049, 050, 053, 056 | VP-042, 043, 045, 054, 055, 057*, 058, 059 | — | — | VP-051 (purity lint), VP-052 (golden matrix) | *VP-057 (guard is unique delta entry) is architecture-review + lint, not runtime |
| op-catalog | VP-004 (validation soundness) | VP-001, 003, 005, 006 | VP-002 (parser) | — | VP-007, 008 (bundled goldens) | digest canonicalization shares the codec determinism approach |
| op-store | — | VP-014, 015, 017, 019, 020, 021, 022, 023, 024, 025, 027 | VP-016 (store parser) | VP-010, 011, 012, 013†, 018, 026 | — | †VP-013 (no truncate-before-durable) is code-audit + lint |
| op-assess | — | VP-030, 031, 032 (kill-injection), 033, 034, 035 | — | — | VP-036 (disclosure snapshots, shared with render) | session purity makes VP-032 deterministic |
| op-render | — | VP-072 (render-parse round trip) | VP-075 (escape) | VP-076 (output safety, with op-cli) | VP-070, 071, 073, 074 (snapshot/golden/audit) | askama autoescape is the backstop, tests are the proof |
| op-cli | — | — | — | — | VP-080, 081, 082, 083, 084, 085, 086 | thin shell — everything here is an integration/CI gate |

## Coverage Accounting

- **Total VPs:** 70 defined across 33 BCs (ranges: 001–008, 010–027, 030–036, 040–059, 070–076, 080–086).
- **Every VP has:** owning BC (definition), module (this matrix), method (this matrix). The consistency-validator should fail any future VP added to a BC without a row here.
- **CRITICAL modules** (op-score, op-store): 10 formal proofs + 19 property suites + fault injection — matches module-criticality.md ≥95% kill-rate intent.
- **Deliberately thin:** op-cli has zero unit-level VPs; its correctness is the sum of taxonomy mapping (VP-080) and shared-source goldens (VP-086). Adding logic to op-cli that would deserve its own VP is an architecture smell — push it down into a pure crate.

## Gap Register (known, accepted)

| Gap | Why accepted | Revisit when |
|-----|-------------|--------------|
| Windows syscall-trace for VP-083 is a documented manual gate | no clean strace equivalent in CI runners | a Windows CI tracing tool is adopted |
| fsync durability beyond OS promise | unverifiable from userspace | never (documented limitation, R-006 backup guidance) |
| CAP-012/013 (evidence, otsniff import) | post-MVP; schema reserves space but no VPs yet | feature-mode delta when built |
