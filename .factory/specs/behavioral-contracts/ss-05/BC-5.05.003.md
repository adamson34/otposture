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
subsystem: "SS-05"
capability: "CAP-008"
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

# Behavioral Contract BC-5.05.003: Executive Report Generation

## Description

`report --exec` renders the one-page static HTML executive summary — the product's flagship deliverable. Self-contained single file (inline CSS, no external resources, no JS required for content), print-to-PDF clean, readable by a non-technical COO in 60 seconds.

## Preconditions

1. ≥1 snapshot exists (the report is snapshot-based, never live-Assessment-based — exec artifacts must correspond to committed history). Comparison snapshot defaults to: previous snapshot, or none (baseline report).

## Postconditions

1. Content sections, in order: (a) site name + report date + catalog version; (b) headline band with trend arrow and percent sub-metric; floor-rule cap banner with gaps when active; (c) "what changed" — top movers from the Delta, driver-attributed, regressions included; (d) top open risks — critical gaps and lowest-scoring functions; (e) next best actions with effort labels; (f) data-completeness note (unknown count) when > 0; (g) active tier overrides with rationales (DEC-005); (h) footer: generated-by tool version, snapshot seqs, catalog digest prefix — reproducibility trail.
2. Single `.html` file, no network fetches when opened (verifiable: zero external URLs in markup), renders legibly at A4/Letter print width.
3. Contains no personal data — roles/teams at most, never names (DI-013).

## Invariants

1. Pure projection: generation reads the store and writes only the output file (FM-004); a render failure leaves the store untouched.
2. Every number in the report reconciles with `status`/`diff` for the same snapshots (single scoring source, BC-4.04.005).
3. No marketing-style score inflation: the uncapped band is never shown without its cap when the floor is active.

## Edge Cases

| ID | Description | Expected Behavior |
|----|-------------|-------------------|
| EC-001 | Baseline report (single snapshot) | "Baseline" framing; no trend claims; next actions + completeness emphasized (DEC-001) |
| EC-002 | Heavily-Unknown store | Prominent completeness caveat box before any percentage (BC-3.03.003 EC-003) |
| EC-003 | Report spanning a bridged migration | Derivation note in "what changed"; series break (if unbridged in range) renders as explicit discontinuity statement |
| EC-004 | Site name with HTML-special characters | Escaped — no markup injection from store content (free-text fields are untrusted input) |

## Canonical Test Vectors

| Input | Expected Output | Category |
|-------|----------------|----------|
| Store with improving 3-snapshot history | One-page HTML, all 8 sections, reconciled numbers | happy-path |
| Floor-capped store | Cap banner present; uncapped band only alongside cap explanation | happy-path |
| Site named `<script>alert(1)</script> WTP` | Escaped text rendering; no script execution | error |

## Verification Properties

| VP | Property | Proof Method |
|----|----------|-------------|
| VP-073 | Report contains zero external resource references | CI HTML audit |
| VP-074 | Report numbers equal status/diff outputs for same snapshot pair | golden tests |
| VP-075 | All store-sourced strings are HTML-escaped | fuzz (store content → render → parse) |

## Traceability

| Field | Value |
|-------|-------|
| L2 Capability | CAP-008, CAP-009 |
| L2 Domain Invariants | DI-007 (no external fetches), DI-013 |
| L2 Failure Modes | FM-004 |
| L2 Edge Cases | DEC-001, DEC-005 |
| Architecture Module | (filled by architect) |
| Stories | (filled by story-writer) |

## Related BCs

- BC-4.04.004/006/008 — depends on (content inputs)
- BC-5.05.004 — composes with (output path safety)
