---
document_type: product-brief
level: L1
version: "1.0"
status: draft
producer: "claude-opus-4-8 (create-brief skill, facilitated with Luke Adamson)"
timestamp: 2026-06-05T11:56:03
phase: 1a
inputs:
  - .factory/planning/brainstorming-report.md
  - .factory/planning/research-report.md
input-hash: "aa15d8c4a44404e8454547970bc97fb3"
traces_to: ""
---

# Product Brief: otposture

## What Is This?

`otposture` is a free, open-source, offline-first CLI that lets industrial plants track their OT security posture continuously instead of measuring it once a year. Posture lives as versioned, git-friendly data scored against CISA's Cybersecurity Performance Goals 2.0; every re-assessment takes minutes, every change is diffable ("▲ since April, driven by the IDS deployment"), and a one-page executive report renders itself. The assessment industry sells snapshots; otposture is the ticker between them.

## Who Is It For?

| Persona | Pain Point | Current Workaround |
|---------|-----------|-------------------|
| **Dual-hatted plant OT engineer / security lead** (primary) — keeps the plant alive AND owns security reporting at a small-to-mid industrial site (water, manufacturing, energy). Per the SANS/OPSWAT 2025 survey, only 9% of ICS/OT security professionals are fully dedicated to it — this persona is the norm. | Can't say "we improved X%"; paid point-in-time assessments are stale within weeks; weekly/monthly C-suite updates are unpaid overhead; doesn't know where to start. | Spreadsheet + memory + hand-built slide decks; or a $50k annual assessment that's wrong by the next maintenance window. |
| **OT security consultant** (secondary) — reassesses the same clients repeatedly. | Re-collects baseline posture from scratch each engagement; no continuity between visits. | Re-running full assessments; copy-pasting from last year's report. |

Technical sophistication assumption (validated with the author, himself the primary persona): CLI-comfortable. The deliverable for *non*-technical stakeholders is the generated HTML/PDF report, not the tool itself.

## Scope

### In Scope (MVP)

- **Guided assessment** against the CISA CPG 2.0 goal set via CLI prompts — ordinal answers (Not / Partially / Largely / Fully Implemented, N/A, Unknown), completable incrementally in minutes-scale sessions.
- **Defensible scoring** — maturity band as the headline (floor rule: band is capped while any Critical-tier control is Not Implemented or Unknown), percentage and signed delta as labeled sub-metrics, per-domain sub-scores. Weights explicit and inspectable in the data.
- **Versioned posture history & diff** — posture state stored as human-readable, git-friendly files pinned to a framework version; `diff`/`status` show what moved and why, with every delta attributed to its driver controls.
- **Executive one-pager generator** — `report --exec` renders a styled static HTML (print-to-PDF) summary: band, trend, biggest movers, top open risks, suggested "next best actions" ranked by impact vs. effort.
- **CPG 2.0 catalog as data** — the framework encoded in a machine-readable (OSCAL-shaped) catalog file, separate from engine code, so additional frameworks can plug in later.

### Out of Scope

- **Any network access or scanning** — no agents, no appliances, no live connections to OT equipment, no cloud upload. Strictly artifact/answer-based and offline.
- **Evidence attachment & staleness decay** — designed-for but post-MVP (fast-follow #1; the data format must reserve room for it).
- **otsniff integration** (pcap findings auto-filling network controls) — post-MVP fast-follow #2.
- **IEC 62443 content** — ISA/IEC copyright prohibits embedding standard text or derivatives; at most, future mapping by control ID reference. Not in v1.
- **Multi-site / fleet roll-up (CISO view)** — single site per data store in v1.
- **GRC-platform features** — no ticketing, workflow, user management, or audit-trail compliance claims.
- **Web UI / TUI** — CLI + static report only in v1.

## Success Criteria

Primary goal (chosen by author): **credibility builder** — the project establishes professional standing in the OT security community.

| Outcome | Metric | Target |
|---------|--------|--------|
| Dogfood proof | Author's own monthly exec update produced via `otposture report --exec` | In routine use within 1 month of v1; update prep time < 15 minutes |
| Community credibility | Conference talk/CFP acceptance, podcast/newsletter mention, or citation in professional work referencing otposture | ≥ 1 within 12 months of v1 release |
| Real-world adoption signal | Distinct external users confirming use (issues, discussions, direct reports) | ≥ 10 within 12 months of v1 release |
| Re-assessment cadence (the thesis) | Time for a returning user to update posture and regenerate the report | < 15 minutes per session |

## Constraints & Integration Points

- **Solo developer, nights & weekends; no deadline** — ship when ready. Scope discipline substitutes for schedule pressure: anything not in MVP scope is deferred, not squeezed in.
- **No OT lab or hardware access** — the tool must be buildable and testable with no industrial equipment (pure parsing/scoring/rendering; fixtures over hardware).
- **Rust** — single static binary, cross-platform (incl. air-gapped Windows engineering workstations), no runtime dependencies. Aligns with formal-verification tooling (Kani, cargo-mutants) used by this factory.
- **Offline-first is a hard product boundary** — OT data sovereignty is a differentiator, not an implementation detail. No telemetry, no phone-home, no cloud requirement, ever.
- **Framework content licensing** — embed only public-domain / permissively-usable framework content (CPG 2.0 expected public domain as US government work — verify the PDF's copyright page before release). Never embed ISA/IEC 62443 or SANS whitepaper text; reference by ID/name only.
- **Framework-version pinning** — every stored assessment records its catalog version; history is never silently rescored (back-cast under new rubric or show an explicit series break).
- **CSET coexistence** — position as complementary ("the ticker between assessments"); avoid feature-mirroring CSET's assessment depth. CSET's CPG 2.0 module (Q1 2026) makes this positioning load-bearing.

## Prior Art & References

- **otsniff** (author's prior tool) — pcap triage for OT; proven wedge for offline artifact analysis; planned post-MVP evidence source.
- **CISA CSET** (github.com/cisagov/cset, MIT/Apache, active) — nearest free competitor; already has Trend & Compare across repeated assessments; heavyweight, assessment-event-centric. Its open-source scoring code is reference material for imitate-vs-diverge decisions.
- **DOE C2M2 v2.1** — precedent for ordinal answers + MIL gating (the "no single score" design rationale) and for local-only data handling.
- **CIS CSAT** — precedent for multi-dimension per-control scoring (policy vs. implemented vs. verified).
- **NIST OSCAL** — machine-readable catalog format; NIST CSF 2.0 already published in OSCAL; no public CPG 2.0 OSCAL catalog exists (community-contribution opportunity).
- **CyberOT, ICS Ninja Scanner** — open-source prototypes adjacent to the space (1–9 stars; the latter requires live scanning and is noncommercial-licensed); confirm the niche is unowned.
- Full citations: `.factory/planning/research-sources.md` (35 sources, accessed 2026-06-05).

## Open Questions

1. **Exact CPG 2.0 goal count and implementation-scale structure** — read the CPG 2.0 PDF + Complete CPGs Matrix spreadsheet to size the catalog and the guided-assessment flow. (Sources conflicted: "four new goals" vs. "six new, three removed.")
2. **CPG 2.0 public-domain confirmation** — check the report's copyright page for contractor notices before embedding goal text.
3. **Critical-tier control designation** — CPG 2.0 doesn't tag goals Critical/High/Standard; otposture's floor rule needs a defensible tiering (own judgment + cited rationale, or derive from CPG's cost/impact ratings?).
4. **Maturity band names and thresholds** — e.g., Foundational / Developing / Managed / Resilient; thresholds need definition and rationale.
5. **Posture data file format** — YAML vs. TOML vs. JSON for the human-readable, git-diffable store; one file vs. file-per-domain.
6. **CSET scoring arithmetic** — read cisagov/cset scoring code to decide where to imitate and where to deliberately diverge (treatment of Unknown/N/A).
7. **"Next best actions" ranking inputs** — where do impact/effort estimates come from in v1 (static per-goal metadata vs. user-supplied)?

## Overflow Context (Optional — Reference Only)

- **Market timing narrative** (for README/launch): CISA's 2025 staffing fell ~3,700 → ~2,200–2,600 and vulnerability assessments were cut $30.8M (free federal assessments shrinking) while NIS2/ENISA guidance pushes continuous, evidence-backed posture for industrial entities — the gap otposture fills is widening from both sides.
- **SANS Five ICS Critical Controls** — planned as an executive-report *grouping overlay* (present CPG results under the five controls) since execs increasingly recognize that framing; concept-level use only, no whitepaper text embedding.
- **Differentiation summary vs. CSET** (from research): minutes-scale re-assess loop vs. assessment events; git-native versioned history; delta driver-attribution; staleness decay (post-MVP); self-writing exec one-pager; single static binary vs. desktop/enterprise web install.
- **Monetization** — none planned; success is credibility. If demand signals emerge, a multi-site/CISO roll-up tier was identified as the plausible commercial seam (explicitly out of scope now).
- **Naming** — `otposture` continues the `ot*` family begun with otsniff, building an identifiable toolchain brand.
