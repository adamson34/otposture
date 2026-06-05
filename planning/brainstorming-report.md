# Brainstorming Report: OT Security Posture Tracker

## Session Summary

- **Date:** 2026-06-05
- **Participant:** Luke Adamson (OT security practitioner; author of `otsniff`, a pcap triage tool)
- **Facilitator:** Claude (vsdd-factory brainstorming skill)
- **Techniques used:** Brain dump (pain-point elicitation), SCAMPER (seeded on otsniff), guided synthesis
- **Context:** Greenfield. Solo developer, nights & weekends, no OT lab/hardware access. Target scale: small utility / open-source.

## Problem Discovery (Brain Dump)

The core pain, in the participant's own framing:

> People struggle to track and improve security posture accurately and timely.

Decomposed into four sub-pains:

1. **Overwhelm** — practitioners don't know where to start; "there is so much."
2. **No quantifiable progress** — no way to say "we've improved our security posture by X%."
3. **Staleness** — posture is measured via paid point-in-time assessments. A plant can be assessed as "really bad," install an IDS two weeks later, and the assessment is already wrong. The cadence of measurement doesn't match the cadence of change.
4. **Reporting burden** — the pain lands on the "unicorn workers" who keep the plant alive AND are responsible for updating the C-suite. They have no time to build dashboards or slide decks weekly for a CISO/COO.

**Key insight from synthesis:** The assessment industry sells *snapshots*; the practitioner needs a *ticker* — a number that moves, with evidence behind it, that generates its own exec update.

**Persona:** The overloaded plant OT engineer/security lead ("unicorn worker") — dual-hatted operations + security, reports upward, time-poor, often budget-poor.

## Ideas Generated

### SCAMPER provocations (seeded on otsniff)

| Lens | Idea | Disposition |
|------|------|-------------|
| Substitute | Triage other artifacts: PLC project files, historian exports, firewall configs, EWS images | Not selected; folded into Concept C as future evidence sources |
| Combine | otsniff output + asset inventory → network baseline + diff | Folded into Concept A as an evidence layer |
| Adapt | "jq for Modbus/DNP3", grep-able OT protocol decoder | Parked — reference material |
| Modify/Magnify | Auto-generate assessment-report deliverables from artifacts | Became Concept B |
| Put to other use | Same parsing for IR timelines, NERC CIP evidence, engineer troubleshooting | Parked |
| Eliminate | Remove what every OT tool requires: agents, appliances, live network access, cloud upload | Adopted as core design constraints for Concept A |
| Reverse | Generate realistic OT traffic for detection testing without a lab | Parked — interesting but different audience |

### Synthesized concepts

**Concept A — Posture-as-code tracker (SELECTED)**
Local-first CLI/TUI. Posture stored as versioned, git-friendly data. Scored against a lightweight framework (candidate: CISA Cross-Sector CPGs — free, simple, designed for exactly this audience, unlike the sprawling IEC 62443). Re-assess in minutes, diff over time ("▲ +8% since April"), attach evidence artifacts (including otsniff output), auto-render a one-page exec summary. Suggests "next best actions" ranked by impact vs. effort to address the "where do I start" overwhelm.

**Concept B — Exec report generator only**
Narrower: skip frameworks/scoring. Ingest whatever the engineer already tracks and auto-generate the weekly/monthly C-suite update. Rejected as the primary direction (the report needs a credible score underneath it to be more than formatting), but retained as a flagship *feature* of Concept A (`report --exec`).

**Concept C — Evidence-first scanner suite**
Family of artifact analyzers (pcaps, firewall configs, asset exports) emitting posture findings; scoring layered on later. Rejected as the entry point (delays the "number that moves" that defines the pain) but retained as Concept A's evidence roadmap — otsniff becomes the first evidence plugin.

## Selected Direction

- **Problem:** OT plant security leads can't measure, track, or communicate security posture at the cadence it actually changes; point-in-time assessments are expensive and stale within weeks, and upward reporting is unpaid overhead.
- **Solution:** A local-first, offline posture tracker — versioned posture data scored against a lightweight framework, diffable over time, with attached evidence and auto-generated executive reporting.
- **Audience:** Dual-hatted OT engineers / plant security leads at small-to-mid industrial sites (water, manufacturing, energy) who report to a CISO/COO; secondarily, OT consultants who reassess clients repeatedly.
- **Differentiator:** *Tracking, not assessing.* Free/open-source, offline/no-agents/no-appliance (OT data sovereignty), evidence-linked rather than purely questionnaire-based, and the exec one-pager generates itself. Existing free tools (CISA CSET, DOE C2M2 tool) are heavyweight one-shot assessments; commercial platforms (Claroty, Dragos) require appliances and budget the target persona doesn't have.
- **Fit to constraints:** Pure parsing/scoring/rendering — no lab, no hardware, scopeable by one person; otsniff is a ready-made first evidence source.
- **Working name:** `otposture` (placeholder).

## Open Questions for Research

1. **Framework choice:** CISA CPGs vs. SANS ICS 5 Critical Controls vs. C2M2 vs. IEC 62443 subset — which gives credible scoring with minimal assessment burden? Can frameworks be pluggable?
2. **Scoring model:** How to compute a defensible "% posture" that survives C-suite scrutiny? Weighted controls? Maturity levels? How do other tools (CSET SAL, C2M2 MIL) do it?
3. **Competitive check:** Does anything already own "continuous lightweight OT posture tracking, offline, open-source"? Verify CSET/C2M2 tooling hasn't added tracking/diffing features.
4. **Evidence linkage:** What artifact types should the MVP accept as evidence vs. defer (pcap via otsniff first; firewall configs, asset CSVs later)?
5. **Staleness mechanics:** How should evidence/answers decay (e.g., "asset inventory stale 90 days ⚠")? Per-control freshness windows?
6. **Multi-site:** Is single-plant enough for v1, or is site-to-site comparison (CISO view) a near-term need?
7. **Data format:** Plain YAML/TOML in git vs. embedded DB — what do practitioners tolerate, and what makes diffs human-readable?
8. **Exec report format:** PDF vs. Markdown vs. PowerPoint — what does the C-suite update actually need to look like?

## Recommended Next Step

**Planning research** (`/vsdd-factory:planning-research` or market-intelligence-assessment) focused on open questions 1–3 (framework choice, scoring model, competitive landscape), then proceed to brief creation (`/vsdd-factory:create-brief`).
