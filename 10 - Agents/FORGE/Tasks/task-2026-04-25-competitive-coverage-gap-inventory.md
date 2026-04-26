---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-25
priority: medium
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Competitive Coverage Gap Inventory — April 2026.md"
---

# Task: Competitive Coverage Gap Inventory

## Context

SENTINEL's mandate per Config: "Compare against competitors: Cross-check Evo Draw rankings against Competitors for accuracy and completeness." The Competitors vault note identifies 7 competitive gaps — but these are ranking quality and feature gaps, not source coverage gaps. What does not exist: a structured inventory of leagues and data sources that competitor ranking platforms currently index that Evo Draw does not.

The DSS audience includes coaches, club directors, and league administrators. They will ask: "Does your data cover X league?" If that league is on USA Rank and not on Evo Draw, SENTINEL needs to know now — not in the demo room. Conversely, if Evo Draw covers leagues that no competitor indexes, that is a competitive edge worth naming explicitly in the DSS narrative.

This task gives SENTINEL the competitive coverage map before May 9 go/no-go.

## What to Produce

### File: `Infrastructure/Competitive Coverage Gap Inventory — April 2026.md`

---

### Section 1: Competitor Source Index

For each major competitor in the Competitors note, list the leagues and sources FORGE can identify from public information (competitor websites, source attribution language, ranking methodology pages):

| Competitor | Known Indexed Leagues / Sources | Confidence (confirmed / inferred / unknown) |
|-----------|--------------------------------|---------------------------------------------|
| USA Rank | | |
| TopDrawerSoccer | | |
| [Other competitors from Competitors note] | | |

FORGE should use available public information only. Do not fabricate. If FORGE cannot determine a competitor's source coverage, mark it as `unknown` with the reason.

---

### Section 2: Coverage Gap Table

For each league or source identified in Section 1 that competitors index but Evo Draw does not currently cover:

| League / Source | Competitor(s) Indexing | Evo Draw Status | Gap Severity | Notes |
|----------------|----------------------|-----------------|-------------|-------|
| | | | HIGH / MEDIUM / LOW | |

Gap Severity criteria:
- **HIGH:** A top-tier league (ECNL, MLS NEXT, GA) that Evo Draw partially or fully misses
- **MEDIUM:** A regional or secondary league that multiple competitors index
- **LOW:** A niche or low-volume league that only one competitor indexes sporadically

---

### Section 3: Evo Draw Competitive Edge

Leagues or sources that Evo Draw currently covers that no major competitor indexes. These are DSS differentiators.

| League / Source | Why No Competitor Covers It | DSS Value |
|----------------|----------------------------|-----------|
| | | |

---

### Section 4: Priority Additions

Based on Sections 2–3: the top 5 source additions that would most reduce competitive coverage gaps. Rank by: (1) league tier, (2) number of competitors who already index it, (3) feasibility of Evo Draw ingestion.

| Rank | Source | Gap Closed | Feasibility | Stage |
|------|--------|-----------|-------------|-------|
| 1 | | | | Stage 2 / Post-Stage 2 |

---

### Section 5: DSS Competitive Narrative

Two paragraphs SENTINEL can use in the DSS briefing:
1. What Evo Draw covers that competitors do not (Evo Draw's edge)
2. Where Evo Draw coverage is still developing vs. competitors (honest gap framing)

These paragraphs should be draft language — Presten can edit but should not have to write from scratch.

---

## Deliverable Requirements

- All competitors from the Competitors vault note are included in Section 1
- Section 2 gaps are real (not inferred from League Hierarchy alone — use external competitor knowledge)
- Section 4 top-5 list is consistent with the Source Coverage Priority Matrix (already filed)
- Section 5 draft narrative is plain English, suitable for a non-technical DSS audience
- Where FORGE cannot confirm competitor coverage, mark `unknown` with reasoning

## Why This Matters

SENTINEL's competitive comparison function has been unexercised in this sprint. The entire pipeline build and calibration work serves ranking quality — but the DSS audience will evaluate Evo Draw against what they already use. If USA Rank indexes GA Cup and Evo Draw does not, that is a question SENTINEL cannot currently answer. This inventory closes the gap. Due May 5 — before May 9 go/no-go — so SENTINEL can either (1) adjust the coverage narrative, (2) assign FORGE a targeted source acquisition, or (3) brief Presten on which coverage gaps to acknowledge directly in the demo.

## References

- `[[Competitors]]` — 7 competitive gaps (ranking quality, not source coverage)
- `[[League Hierarchy]]` — all leagues Evo Draw currently tracks
- `Infrastructure/Source Coverage Priority Matrix.md` — current source priority ranking
- `Infrastructure/Pre-DSS Source Coverage Report.md` — Evo Draw's confirmed active sources
