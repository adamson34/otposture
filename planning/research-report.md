# Planning Research Report: `otposture`

- **Date:** 2026-06-05
- **Scope:** Open questions 1–3 from `.factory/planning/brainstorming-report.md` — framework choice, scoring model, competitive landscape
- **Method:** Live web research (WebSearch/WebFetch) from the main session. Three research subagents were spawned per skill process but were blocked (WebSearch/WebFetch permission-denied at subagent level; Perplexity/Context7 MCP servers not loaded). One subagent returned a training-data synthesis explicitly flagged UNVERIFIED; its load-bearing claims were subsequently **verified live** in the main session and are cited below. Claims that remain unverified are flagged inline.
- **Citations:** All sources in `research-sources.md` with access date 2026-06-05.

---

## Q1: Framework choice

### Finding 1.1 — CISA CPG 2.0 released December 2025 (HIGH confidence, post-cutoff, verified across 3+ sources)

CISA released **Cybersecurity Performance Goals 2.0** on 2025-12-11 [S1, S2, S3]. Key facts:

- Aligned to **NIST CSF 2.0's six functions** (Govern, Identify, Protect, Detect, Respond, Recover) — adds the GOVERN function vs. CPG 1.0.1 [S1, S4].
- **OT-only goals from 1.0.1 were folded into "universal goals"** addressing IT and OT holistically, explicitly "to enable small- and medium-sized entities to apply one framework" [S5] — i.e., CISA redesigned CPGs *for exactly otposture's target persona*.
- New goals cover third-party/MSP access risk and zero-trust lateral-movement mitigation [S5]. Sources conflict on counts ("four new goals" vs. "six new, three removed") — ⚠ UNVERIFIED exact goal count; confirm against the CPG 2.0 PDF [S2] before embedding.
- Published artifacts: PDF report [S2], a **"Complete CPGs Matrix/Spreadsheet"** described as the master source document [S6], and a **CPG 2.0 Checklist + CSET assessment module due Q1 2026** [S1]. No JSON/OSCAL publication found.
- Caveat found in coverage: CPG cost/impact/ease guidance "primarily applies to IT infrastructure" and doesn't necessarily extend to OT systems [S7] — the tool's OT framing must add value on top.
- Licensing: US federal government works are public domain (17 U.S.C. §105), and the CPGs are distributed freely; ⚠ UNVERIFIED — confirm no contractor copyright notice in the CPG 2.0 PDF before embedding text verbatim.

### Finding 1.2 — C2M2 stable at v2.1 (June 2022) (HIGH confidence)

DOE C2M2 v2.1 remains current [S8, S9]. Self-evaluation via HTML tool (c2m2.doe.gov) or PDF tool; all data stays local; a self-evaluation takes "as little as one day" [S9, S10]. Structure: 10 domains, practices answered Fully/Largely/Partially/Not Implemented, MILs 0–3 achieved per domain by gating (see Q2). Free to use; energy-sector credibility. Assessment burden (~1 day, ~350 practices) is heavy for "re-assess in minutes."

### Finding 1.3 — IEC 62443 is legally unusable for embedding (HIGH confidence)

IEC/ISA copyright prohibits reproduction in any form without written permission; ISA additionally **prohibits entering its IP into AI tools and creating derivatives** [S11, S12]. Embedding 62443 requirement text in an open-source tool is a non-starter. Mapping *by reference* (control IDs only, no text) remains possible. Also note 62443-2-1 Edition 2.0 was published 2024-08 [S13].

### Finding 1.4 — SANS Five ICS Critical Controls: credible lens, not a scoring rubric (MEDIUM-HIGH confidence)

The Lee/Conway whitepaper's five controls (ICS incident response, defensible architecture, network visibility/monitoring, secure remote access, risk-based vulnerability management) are widely adopted as a strategic framing — heavily promoted by Dragos and used to structure assessments and exec communications [S14, S15, S16]. Only five controls → too coarse as the primary scoring rubric, but excellent as an **executive-report grouping layer** ("your posture by the 5 SANS controls"). ⚠ Licensing of the whitepaper text for embedding: UNVERIFIED — use the control names/concepts by reference.

### Finding 1.5 — OSCAL is the pluggability play (HIGH confidence)

NIST OSCAL provides machine-readable catalogs (XML/JSON/YAML); the **NIST CSF 2.0 catalog is published in OSCAL** [S17, S18]. CIS maintains a CIS Controls OSCAL repository [S19]. NIST released CSWP 53 ("Charting the Course for NIST OSCAL") for public comment in December 2025, and RegScale open-sourced an "OSCAL Hub" the same month [S20, S21] — the compliance-as-code ecosystem is consolidating on OSCAL. **No OSCAL edition of the CPGs was found** — publishing a community OSCAL catalog of CPG 2.0 could be otposture's framework-pluggability mechanism *and* a community contribution that drives adoption.

### Recommendation (Q1)

1. **Default framework for v1: CISA CPG 2.0.** Free, designed for small/medium IT+OT entities, CSF 2.0-aligned, exec-credible, ships with a prioritization matrix, and fresh as of Dec 2025. Model the tool's internal catalog as OSCAL-shaped from day one.
2. **Pluggable later:** C2M2 v2.1 (energy sector), NIST CSF 2.0 via NIST's own OSCAL catalog, CIS Controls via CIS's OSCAL repo.
3. **SANS 5 ICS Controls as a reporting overlay**, not a rubric — group CPG results under the five controls in the exec one-pager.
4. **Never embed IEC 62443 text.** Reference IDs only, or omit in v1.
5. **Strongest argument against the CPG pick:** CISA's 2025 budget/staffing turmoil (staffing fell from ~3,700 to ~2,200–2,600; vulnerability assessments cut $30.8M) [S22, S23] creates uncertainty about CPG/CSET maintenance cadence — and CSET's own CPG 2.0 module (Q1 2026) means CISA covers CPG *assessment* itself, so otposture must win on tracking/diff/evidence/reporting, not assessment.

---

## Q2: Scoring model

### Finding 2.1 — C2M2 deliberately refuses a single score (HIGH confidence, verified)

C2M2 answers each practice on a 4-point scale (Not/Partially/Largely/Fully Implemented). A domain achieves a MIL only when **all practices at that MIL and below are Largely or Fully Implemented** — a cumulative, weakest-link gating model, not an average [S10, S24]. The design intentionally avoids a weighted percentage; output is per-domain MIL attainment. This is the canonical precedent that **averaging masks critical gaps**.

### Finding 2.2 — CSET produces compliance percentages and already does trend charts (HIGH confidence, verified)

CSET compares answers to a chosen standard and computes overall compliance percent; its **aggregation feature ("Trend and Compare") charts how scores change over time across multiple assessments of the same facility**, including overall compliance % ranged over the period, areas to improve, and commonly missed questions [S25, S26]. ⚠ Exact aggregation arithmetic (treatment of Alternate/NA/Unanswered) UNVERIFIED — readable from the open-source repo (MIT/Apache licensed [S27]) if we want to imitate or diverge deliberately.

### Finding 2.3 — CIS CSAT scores four dimensions per safeguard (HIGH confidence, verified)

CIS-hosted CSAT scores each safeguard on **Policy / Implementation / Automation / Reporting**, each 1–5; CSAT Pro offers a simple single 1–5 per safeguard; overall score is 0–100 with industry-average comparison [S28, S29]. The multi-dimension model distinguishes "we have a policy" from "it's operating and verified" — directly transferable to otposture's evidence-linking differentiator.

### Finding 2.4 — Known failure modes of single-number scores (MEDIUM confidence — principles verified by framework precedent; vendor-specific criticisms ⚠ UNVERIFIED training data)

Synthesis (from the scoring subagent, cross-checked where possible): gaming via cheap wins; false precision; incomparability over time when the rubric changes; averaging away critical gaps; stale evidence counted as current. Mitigations with precedent: criticality weighting (CIS Implementation Groups), gating/floor rules (C2M2 MILs), bands instead of raw % (NIST CSF tiers/profiles), per-domain sub-scores (C2M2/CSET), framework version pinning.

### Recommended scoring design (Q2)

| Element | Design | Precedent |
|---|---|---|
| Answer scale | 4-point ordinal: Not (0) / Partially (.33) / Largely (.67) / Fully (1.0) + N/A (out of denominator) + Unknown (=0, flagged) | C2M2 [S10] |
| Evidence dimension | Each answer is "claimed" or "evidence-attached"; claimed-only credit is visibly discounted | CIS CSAT dimensions [S28] |
| Weighting | Controls tagged Critical/High/Standard, weights explicit and inspectable in the data file | CIS IGs; anti-opacity |
| Floor rule | Headline band is capped while any Critical control is Not Implemented — no "85%" with a fatal gap | C2M2 MIL gating [S24] |
| Headline | **Maturity band** (e.g., Foundational/Developing/Managed/Resilient) is the hero; the % and its delta are labeled sub-metrics | NIST CSF tiers; C2M2 |
| Staleness | Per-control freshness windows; stepwise decay of credit past the window with a ⚠ stale badge | Novel — addresses pain #3 |
| Diff integrity | Pin every assessment to a framework version; on version change, back-cast history under the new rubric or show an explicit series break; never silently rescore | Avoids the rubric-shift trap |
| Delta attribution | "▲ +8%: +5% IDS deployment, +3% inventory refresh" — every delta names its drivers with evidence links | Survives CISO scrutiny |

This gives the "ticker" the brainstorm wants while inheriting the defensibility of the gating/band models the authorities use.

---

## Q3: Competitive landscape

### Finding 3.1 — CSET is the nearest competitor, and it's closer than assumed (HIGH confidence, verified)

CSET is **actively maintained** (v12.4.0.4, 2025-07-18; 15k+ commits; MIT/Apache licensing) [S27] and its aggregation feature already does **trend (% over time) and comparison across repeated assessments** [S25, S26]. The CPG 2.0 CSET module lands Q1 2026 [S1]. However: it remains a heavyweight desktop/enterprise web application built around *assessment events* (the trend feature assumes ~annual reassessments [S26]); it has no git-native versioned data, no evidence-artifact linking, no staleness decay, no auto-generated exec one-pager, and no minutes-scale re-assess loop. **Differentiation is real but narrower than the brainstorm assumed — it's cadence + evidence + reporting ergonomics, not the existence of trending.**

### Finding 3.2 — C2M2 tooling is point-in-time (HIGH confidence)

The HTML/PDF self-evaluation tools keep data local and can pre-populate from *prior model versions* (for version migration), but no over-time trend tracking feature was found [S9, S10].

### Finding 3.3 — Open-source niche is effectively empty (HIGH confidence, verified)

- **CyberOT** (gloomyleo): React/Flask OT maturity-assessment platform covering IEC 62443 + NIST CSF with longitudinal tracking ambitions — **1 star, 1 fork, 7 commits, no releases; a prototype** [S30].
- **ICS Ninja Scanner** (MottaSec): active scanner with `diff`/`trend` commands — but **requires live network access to ICS devices** (different shape; violates otposture's offline/artifact constraint), is **PolyForm Noncommercial-licensed** (not OSI open source), and has 2 commits/no releases [S31].
- OSCAL/compliance-as-code ecosystem (compliance-trestle, Lula, RegScale OSCAL Hub) is IT/GRC-oriented, not OT-framed [S17–S21] — adjacent infrastructure, not competition; otposture could become "the OT face of OSCAL."

### Finding 3.4 — Commercial platforms don't serve the persona (MEDIUM confidence; ⚠ specifics UNVERIFIED training data)

Claroty, Dragos, Nozomi, Tenable OT, Armis, Microsoft Defender for IoT et al. market posture/risk scores but require sensors/appliances/cloud and enterprise budgets; methodologies are proprietary. Directionally supports the offline/free/transparent gap; per-vendor pricing and 2026 positioning were not individually verified.

### Finding 3.5 — Demand signals validate the persona and the timing (HIGH confidence, verified)

- **SANS/OPSWAT 2025 ICS-OT survey (330 respondents):** only **9% of professionals dedicate 100% of their time to ICS/OT security**; only 27% of orgs place OT budget under a CISO/CSO; 55% report budget growth [S32, S33]. The dual-hatted "unicorn" persona is the statistical norm, not an edge case.
- **NIS2:** ENISA's June 2025 technical guidance maps controls to NIST CSF 2.0/ISO 27001, treated by national regulators as primary compliance evidence; continuous posture monitoring with evidence trails is the emerging expectation for covered industrial entities (50+ employees / €10M+ turnover) [S34, S35].
- **CISA retreat:** 2025 staffing fell from ~3,700 to ~2,200–2,600; vulnerability assessments cut by $30.8M; free assessments for water utilities/rural hospitals threatened [S22, S23]. The free-government-assessment safety net is shrinking exactly as regulatory demand for continuous evidence grows — a widening gap for a free, self-service tool.

### Gap verdict (Q3)

**"Continuous, lightweight, offline, open-source, evidence-linked OT posture tracking with automated exec reporting" is genuinely unowned.** Ranked nearest competitors by overlap:

1. **CISA CSET** — free, trending, CPG module incoming; loses on cadence, evidence linking, ergonomics, exec reporting.
2. **ICS Ninja Scanner** — has diff/trend verbs; loses on live-scan requirement, noncommercial license, maturity.
3. **CyberOT** — same problem framing; prototype-stage.
4. **Commercial OT platforms** — posture scores; lose on footprint/price for small-mid plants.

**Biggest threat to differentiation:** CSET's Q1 2026 CPG 2.0 module making "free CPG assessment with trend charts" a CISA-official answer. Counter-positioning: otposture is the *between-assessments* tool — minutes-scale updates, git-diffable history, evidence decay, and the self-writing exec one-pager — and can even import/export alongside CSET rather than fight it.

---

## Implications for the product brief

1. Default framework: **CPG 2.0**; internal catalog format: **OSCAL-shaped**; frameworks pluggable; SANS 5 as exec-report overlay; no 62443 text embedding.
2. Headline = **maturity band with floor rule**; % + attributed delta as sub-metric; per-control staleness decay; framework-version-pinned history.
3. Positioning: **"the ticker between assessments"** — explicitly complementary to CSET (import its findings) rather than a CSET replacement.
4. Tailwind narrative for README/launch: shrinking free federal assessments + NIS2 continuous-evidence pressure + 91% dual-hatted practitioners.

## Unverified items / follow-ups

| Item | Why it matters | How to close |
|---|---|---|
| Exact CPG 2.0 goal count & implementation-scale structure | Sizes the v1 catalog and re-assess time | Read the CPG 2.0 PDF [S2] + Matrix spreadsheet [S6] |
| CPG 2.0 public-domain status (no contractor copyright) | Legal basis for embedding text | Check copyright page of [S2] |
| CSET aggregation arithmetic (Alternate/NA/Unanswered) | Decide imitate-vs-diverge on scoring | Read scoring code in github.com/cisagov/cset |
| Commercial vendor pricing/positioning 2026 | Sharpens brief's competitive section | Vendor-by-vendor check if needed for launch copy |
| Bitsight/SecurityScorecard methodology criticisms | Useful citations for "why bands not bare %" | 2 independent security-press sources |
