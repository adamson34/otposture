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
subsystem: "SS-01"
capability: "CAP-001"
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

# Behavioral Contract BC-1.01.004: Bundled CPG 2.0 Catalog Content Correctness

## Description

The catalog file shipped with the binary faithfully encodes CISA CPG 2.0 (December 2025, TLP:CLEAR — verified distributable, ASM-001 validated 2026-06-05). Content fidelity is a contract because the catalog is exec-facing data: a wrong goal title or rating misleads real security decisions.

## Preconditions

1. The bundled catalog is the build's default when `--catalog` is not supplied.

## Postconditions

1. The bundled catalog covers exactly the **34 official goals** in six functions with counts: GOVERN 5 (1.A–1.E), IDENTIFY 5 (2.A–2.E), PROTECT 19 (3.A–3.S), DETECT 2 (4.A–4.B), RESPOND 2 (5.A–5.B), RECOVER 1 (6.A) — per ASM-002 validation against the official PDF. Documented sub-item splits (EC-002) roll up to their official parent ID and do not change this coverage assertion.
2. Each goal carries: official goal ID, title, outcome text (verbatim from the TLP:CLEAR source), CISA Cost/Impact/Ease ratings, `default_tier` with a written rationale, and `weight`.
3. `licensing_provenance` cites the source document, its TLP:CLEAR marking, and retrieval date.
4. `catalog_id = "cisa-cpg"`, `version` tracks the CPG edition (e.g., `2.0.0`).

## Invariants

1. No content from restricted frameworks (IEC 62443, SANS whitepapers) appears in the bundled catalog (DI-009); other frameworks referenced by ID only.
2. Tier-default rationales are part of the catalog (publicly reviewable — R-007 mitigation).

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | CISA publishes CPG 2.0 errata | New catalog version (e.g., 2.0.1) with changelog; old version remains available for pinned history (DI-001) |
| EC-002 | A compound goal proves un-assessable as one ordinal answer during authoring | Goal split into sub-items under the official ID (e.g., 3.Q.a/3.Q.b) with the split documented in the catalog changelog |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Count official goal IDs covered per function (sub-items roll up to parents) | 5/5/19/2/2/1, total 34 | happy-path |
| Grep bundled catalog for IEC 62443 requirement text | No matches (ID references at most) | edge-case |
| Validate bundled catalog with BC-1.01.002 rules | Zero defects | happy-path |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-007 | Bundled catalog passes full validation on every build (CI golden test) | manual/CI |
| VP-008 | Every goal ID is an official CPG 2.0 ID or a changelog-documented sub-item of one (`<official-id>.<letter>`), and all 34 official IDs are covered | manual/CI golden |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-001 |
| L2 Domain Invariants | DI-008, DI-009 |
| L2 Assumptions | ASM-001 (validated), ASM-002 (validated), ASM-005 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-1.01.001 / BC-1.01.002 — depends on (loads and validates through them)
- BC-4.04.008 — composes with (ratings feed next-best-actions)
