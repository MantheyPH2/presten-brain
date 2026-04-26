---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Top Sources — Ingestion Design.md"
---

# Task: Non-GotSport Top Sources — Ingestion Design

## Context

FORGE filed `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` today (April 26), completing the research phase on alternative game sources. The synthesis identifies which platforms host league games that Evo Draw currently misses.

Research is not enough. SENTINEL needs to know: can FORGE actually ingest from the top 2 non-GotSport sources identified in the synthesis? What would that pipeline look like? What are the blockers? This task converts the research into a concrete ingestion design for the top 2 candidates.

Without this step, the non-GotSport source work stalls at "we know these exist" with no path to execution. Stage 2 of crawl expansion targets the browser-session org-IDs (GotSport), but Stage 3 needs to reach beyond GotSport entirely — this document is the foundation for Stage 3 planning.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Non-GotSport Top Sources — Ingestion Design.md`

---

### Section 1: Top 2 Source Selection

From the research synthesis, name the 2 non-GotSport sources with the highest coverage value for Evo Draw (highest volume of in-scope games, most reliable data structure, most feasible access path).

For each source, state:
- Platform name
- Why it ranked in the top 2 (game volume estimate, leagues covered)
- Access method: public API / scrape / export / partner relationship

---

### Section 2: Ingestion Design — Source A

For the first source:

**Data model mapping:**
| Field (Source A) | Maps to Evo Draw Field | Transform Required? |
|-----------------|----------------------|---------------------|
| [game identifier] | game_id | |
| [date/time] | game_date | |
| [home team name] | home_team_name | |
| [away team name] | away_team_name | |
| [score] | home_score / away_score | |
| [league/event] | event_name / event_tier | |

If Source A uses a different ID scheme (not GotSport org_id), describe how FORGE would resolve team identity across sources.

**Pipeline integration:**
- Does this source fit the existing `run-daily.sh` architecture, or does it require a separate ingestion process?
- Estimated development effort (hours) to build the connector
- Rate limiting / scrape risk / ToS concerns (one sentence per)

**Blocker assessment:**
- What is the single largest blocker to activating Source A?
- Is the blocker in FORGE's control, or does it require Presten authorization?

---

### Section 3: Ingestion Design — Source B

Same structure as Section 2 for the second-ranked source.

---

### Section 4: Go/No-Go Recommendation

For each source, state a recommendation:

| Source | Recommendation | Primary Reason | Earliest Activation Window |
|--------|---------------|---------------|--------------------------|
| Source A | Activate / Hold / Defer | | |
| Source B | Activate / Hold / Defer | | |

"Activate" = FORGE recommends proceeding after May 1 pipeline is stable.
"Hold" = Technically feasible but a specific gate must pass first (state the gate).
"Defer" = Not worth activating before DSS; revisit post-May 16.

---

### Section 5: What SENTINEL Needs to Authorize

If any activation requires Presten approval (scraping a site, rate of requests, partnership outreach), state it explicitly:

> "Source A activation requires Presten to confirm: [specific authorization question]. FORGE cannot proceed until this is answered."

If no authorization is needed for the design phase, state that explicitly.

---

## Definition of Done

- File written at the deliverable path
- Top 2 sources named with justification
- Both sources have a complete data model mapping (no blank rows)
- Both sources have a go/no-go recommendation with a reason
- Any blockers requiring Presten authorization are stated explicitly
- Summary in briefing: "Non-GotSport ingestion design filed. Source A: [name], recommend [action]. Source B: [name], recommend [action]. Authorization needed: [yes/no + what]."

## Why This Matters

The current crawl expansion is constrained to GotSport because GotSport is the only source with a defined ingestion path. Every other platform hosting in-scope games is a coverage gap. The research synthesis maps the landscape. This design document identifies whether FORGE can close any of those gaps. If the top 2 sources are feasible, SENTINEL can authorize Stage 3 planning before DSS. If they are not feasible, SENTINEL knows the coverage ceiling and can set expectations for Presten accordingly.

## References

- `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` — the research this design builds on
- `Infrastructure/Source Coverage Gap Inventory — April 2026.md` — coverage gaps this work addresses
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — current ingestion baseline for comparison
- `task-2026-04-24-snapsoccer-ingestion-design.md` — prior SnapSoccer-specific ingestion work (may inform Source A/B designs)
