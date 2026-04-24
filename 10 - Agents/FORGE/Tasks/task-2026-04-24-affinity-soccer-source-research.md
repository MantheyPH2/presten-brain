---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: medium
due: 2026-04-28
---

# Task: Affinity Soccer Source Research

## Context

The GotSport org-ID expansion task (2026-04-24) explicitly skipped Illinois because IL uses **Affinity Soccer** instead of GotSport. Affinity Soccer is also used by several other USYS state associations and regional leagues — making it a distinct source family that Evo Draw currently has zero coverage from.

USA Rank references Affinity Soccer leagues in their rankings. SENTINEL has not yet audited what coverage gap this represents. Before DSS, we need to know: is Affinity Soccer a small supplementary source, or is it a meaningful gap (like SnapSoccer turned out to be)?

This is vault-only research. No DB access required.

## What You Need to Do

### Step 1: Identify Which States and Leagues Use Affinity Soccer

Based on your knowledge of the US youth soccer ecosystem and the USYS State Cup Source Research doc:

- Which USYS state associations use Affinity Soccer instead of GotSport? (Confirmed: Illinois. Likely others.)
- Are any ECNL or MLS NEXT affiliated leagues on Affinity Soccer?
- Do any national-level US Club Soccer events run on Affinity Soccer?

Document what you find in a table:
| State/League | Platform | Notes |
|---|---|---|
| Illinois Soccer (IYSA) | Affinity Soccer | USYS state |
| ... | ... | ... |

### Step 2: Investigate Affinity Soccer's Public Data Exposure

Affinity Soccer is a competing platform to GotSport. Research (from memory/knowledge of the platform):

- Does Affinity Soccer have a public-facing results or standings page?
- Is there a known API or structured data endpoint?
- Are game results (scores, dates, teams) publicly accessible without authentication?
- What is the data format (HTML tables, JSON API, CSV export)?

Document for each:
- **Public results pages:** Yes/No, URL pattern if known
- **API:** Exists / Unknown / Not accessible
- **Authentication required:** Yes/No/Unknown
- **Data quality:** Complete scores/dates/teams, or partial?

If you don't have specific knowledge of Affinity Soccer's technical interface, document what is known about their product (it's a sports management platform similar to GotSport) and note that URL/API research requires Presten to investigate in a browser.

### Step 3: Estimate Coverage Gap

Based on the states and leagues identified in Step 1:

- How many USYS state association teams are likely on Affinity Soccer?
- What is the estimated game volume per year from Affinity Soccer sources (relative to GotSport's volume)?
- Is the gap concentrated in specific age groups or regions?

Use the following reasoning framework:
- Illinois alone has ~50,000 registered youth soccer players. If IL state league results are on Affinity Soccer, that could represent 5,000–15,000 games/year.
- Compare to SnapSoccer's coverage estimate from your SnapSoccer Source Research doc for calibration.

State whether Affinity Soccer represents:
- **Major gap** (>10,000 games/year, multiple top-10 states): warrants a dedicated ingestion effort
- **Moderate gap** (5,000–10,000 games/year): worth adding after SnapSoccer
- **Minor gap** (<5,000 games/year): deprioritize until post-DSS

### Step 4: Compare to USA Rank's Coverage

Does USA Rank appear to incorporate Affinity Soccer data? Evidence either way:
- Are Illinois state league teams ranked by USA Rank?
- If yes, and we don't have them, this is a direct competitive differentiator argument.
- If USA Rank also lacks them, the gap is industry-wide.

### Step 5: Effort Estimate and Recommendation

Given what you've found:
1. How difficult would it be to ingest Affinity Soccer data relative to GotSport?
   - GotSport: known API, established scraper
   - SnapSoccer: new scraper, being designed now
   - Affinity Soccer: [your assessment]
2. Is this worth pursuing before DSS (May 16)?
3. If yes: what is the MVP approach (scraper vs API vs data request to Affinity Soccer)?

Write a clear recommendation: pursue now, pursue post-DSS, or deprioritize with reasoning.

## Deliverable

Write the research document to:
`02 - Tiger Tournaments/Projects/Data Sources/Affinity Soccer Source Research.md`

Structure it the same way as `SnapSoccer Source Research.md` — platform overview, data exposure, coverage estimate, effort assessment, recommendation.

## Definition of Done

- States and leagues using Affinity Soccer are documented (at minimum 3 confirmed)
- Data exposure is assessed (public results or not)
- Coverage gap estimate is specific (game count range, not "unknown")
- Recommendation is stated: pursue before DSS, after DSS, or deprioritize
- Summary in your next briefing: estimated coverage gap and recommendation

## References

- `02 - Tiger Tournaments/Projects/Data Sources/SnapSoccer Source Research.md` — format template + comparative coverage data
- `02 - Tiger Tournaments/Projects/Data Sources/USYS State Cup Source Research.md` — IL skipped note
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — does USA Rank show IL state league teams?
