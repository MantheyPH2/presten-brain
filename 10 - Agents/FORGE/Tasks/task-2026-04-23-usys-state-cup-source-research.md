---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-23
status: pending
priority: high
---

# Task: Research USYS State Cup and State League Results as a Data Source

## Context

USA Rank ingests USYS State Cup results, State League games, and Presidents Cup results — these are confirmed in their event list (our intel file explicitly names "state cups, Presidents Cup, state leagues, NPL" as high-confidence USA Rank sources). USYS-affiliated state associations publish results on their own websites and through the USYS national platform. This is a significant coverage gap: elite teams from these competitions are heavily represented in USA Rank's rankings but may be missing or underrepresented in Evo Draw.

Read before starting:
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — "Confirmed Source Families" table, USYS/State Associations row
- `02 - Tiger Tournaments/Projects/Data Sources/Data Sources Overview.md` — current source families and priority system
- `02 - Tiger Tournaments/Projects/Pipeline/Data Pipeline.md` — Step 0 scraper requirements and source_id format conventions

## What You Need to Research

### 1. USYS Platform Footprint
- Where does USYS publish game results nationally? Is there a central `usys.org` results portal, or does each state association run its own site?
- Identify the top 5–10 state associations by team volume (TX, CA, FL, NY, NJ are likely largest). What platform does each use to publish results — GotSport, SincSports, AffInity, or their own custom site?
- Are USYS National Championships and Presidents Cup results published in a machine-readable format anywhere?
- Is there a `usys.org/events` or similar results hub accessible without credentials?

### 2. State League Coverage Audit
- USA Rank lists NPL (National Premier Leagues) and state leagues as confirmed sources. Which state leagues are the most active? (Elite Clubs National League state cups, US Youth Soccer state leagues, etc.)
- For the top 3 state associations (recommend starting with TX, CA, FL), identify:
  - The platform they use to publish results
  - Whether those platforms are already in our source list (e.g., if TX uses GotSport, we may already have these games)
  - The gap — games we're definitely missing

### 3. Scraper Feasibility per Platform
- If results are on GotSport: we likely already capture them if the event org_id is in our crawl scope. Identify whether state cup events are under GotSport org IDs we don't yet crawl.
- If results are on SincSports: same infrastructure as SnapSoccer (already assessed). Is there a USYS-specific SincSports tournament ID range?
- If results are on a custom state site (e.g., `texassoccer.org`): assess HTML structure, auth requirements, and crawl feasibility.
- Rate limit risk and auth requirements for each platform found.

### 4. Dedup Assessment
- If state cup games were already played under GotSport events, are they already in our DB?
- Design a check: `SELECT COUNT(*) FROM games WHERE event_name ILIKE '%state cup%' OR event_name ILIKE '%presidents cup%' OR event_name ILIKE '%NPL%'` — estimate what we already have vs. what's missing.
- Dedup risk if we add a new USYS source vs. existing GotSport coverage.

## What Done Looks Like

Produce a research memo at:
`02 - Tiger Tournaments/Projects/Data Sources/USYS State Cup Source Research.md`

The memo must include:
1. Platform map — which state associations use which platform for publishing results
2. Coverage gap estimate — games in state cup/NPL/Presidents Cup events that are NOT currently in our DB
3. Top 3 sources to add (ranked by game volume impact)
4. Scraper feasibility for each: Go / No-Go, effort estimate, auth required
5. Dedup risk per source (Low / Medium / High overlap with existing GotSport coverage)
6. Recommended source priority numbers (1–8 scale)
7. Build order recommendation: which to tackle first given DSS is May 16, 2026

## Success Criteria

- Research memo written with all 7 sections
- At least one concrete Go recommendation with scraper skeleton or implementation notes
- Coverage gap is quantified — not just "we might be missing games" but a specific estimate
- If all state cup games are already captured via GotSport, document that finding clearly so we stop investigating this path
