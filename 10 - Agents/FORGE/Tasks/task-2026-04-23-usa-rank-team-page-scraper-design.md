---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-23
status: pending
priority: high
---

# Task: Design a USA Rank Team Page Scraper for Competitive Benchmarking

## Context

USA Rank exposes ranked team data through a predictable URL structure:
- `usarank.com/rank/view` — ranking finder
- `usarank.com/teamlist/{id}/{year}/{agegroup}` — team pages with rank, rating, and game history
- `usarank.com/usaschedules/{id}/{year}/{agegroup}` — schedule/results pages

Our competitive intel file documents specific ranked teams we've already captured manually (e.g., "The Football Academy NJ 2012 Girls Black, #4" in U13G). A systematic scraper of these pages would give us:
1. A ranked team list to cross-check against our own rankings (where do we agree/disagree with USA Rank?)
2. Before/after rank movement data to validate our own ranking sensitivity
3. Event-level data (which events USA Rank is scoring) to identify source gaps
4. Opponent ratings and scores that USA Rank has indexed that we may not have in our DB

This is a benchmarking and gap-detection tool — not for production rankings, but for SENTINEL-level quality auditing.

Read before starting:
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — "Public Ranked Team Data" section, entry points, sample ranked teams captured
- `02 - Tiger Tournaments/Projects/Data Sources/Data Sources Overview.md` — current source architecture
- `02 - Tiger Tournaments/Projects/Rankings/Ranking Engine.md` — our Elo/Glicko methodology, for understanding what a ranking comparison would surface

## What You Need to Design

### 1. URL Structure Analysis
- Document the full URL pattern for USA Rank's public team and schedule pages
- What parameters are required: `id` (team ID?), `year`, `age_group`?
- How are age groups encoded in their URLs? (U13G, U13-Girls, 2012-girls, etc.)
- Is there a paginated team list endpoint that lets us enumerate ranked teams by age group without knowing team IDs in advance?
- Does `usarank.com/rank/view` expose a filterable list that could be crawled page-by-page to collect team IDs?

### 2. Data Fields Available
For each team page (`/teamlist/`), document what fields are present:
- Team name, rank, rating/score
- Before/after rank movement (week-over-week)
- Recent game list: opponent name, score, date, event name, opponent rank
- Schedule rating (average opponent strength)
- Band label (Gold/Silver/Bronze/etc.)

For each schedule page (`/usaschedules/`), document:
- Game results format
- Whether event names are present (to identify which of USA Rank's 400+ sources the game came from)
- Whether opponent team IDs or names link back to their own team pages

### 3. Technical Feasibility
- Is `usarank.com` server-rendered HTML or client-side JavaScript? (Curl test / inspect source)
- Is there Cloudflare or bot-detection protecting the pages?
- What is a safe crawl rate? (Respect robots.txt — document what it says)
- Estimate: how many requests to collect all ranked teams for one age group? For all age groups?
- Is a full crawl feasible for benchmarking purposes (weekly run, not real-time)?

### 4. Comparison Framework Design
Design the cross-reference logic:
- Match USA Rank teams to Evo Draw teams by name (fuzzy match — same problem as our team_merges)
- For matched teams: compute ranking delta (USA Rank rank − Evo Draw rank)
- Flag teams where our ranking diverges by more than ±50 positions
- Flag teams USA Rank has ranked that we have no record of (missing teams)
- Flag events USA Rank has scored that appear in game histories but not in our source list

### 5. Output Schema
Design a lightweight benchmarking output table:
```sql
CREATE TABLE usa_rank_benchmark (
  id SERIAL PRIMARY KEY,
  scraped_at TIMESTAMPTZ,
  age_group VARCHAR(10),
  gender CHAR(1),
  usa_rank_team_name VARCHAR(200),
  usa_rank_rank INT,
  usa_rank_band VARCHAR(20),
  evo_draw_team_id INT REFERENCES teams(id),
  evo_draw_rank INT,
  rank_delta INT,  -- usa_rank_rank - evo_draw_rank (positive = USA Rank rates them higher)
  matched BOOLEAN,
  match_confidence NUMERIC(3,2)
);
```
Review and revise.

## What Done Looks Like

Produce a design document at:
`02 - Tiger Tournaments/Projects/Data Sources/USA Rank Benchmark Scraper Design.md`

The document must include:
1. URL structure documentation — confirmed patterns with examples
2. Data field inventory — every field available on team and schedule pages
3. Technical feasibility assessment — HTML vs JS, bot protection, safe crawl rate
4. Comparison framework design — matching logic, delta thresholds, flag criteria
5. Output schema — revised SQL DDL for `usa_rank_benchmark` table
6. Crawl scope estimate — how many requests for a full age-group sweep
7. Implementation effort estimate — hours to build, can it be ready before DSS?
8. Ethical/legal note — are we scraping within fair-use limits? (Public data, no login required, respectful crawl rate — document the boundary)

## Success Criteria

- Design document written with all 8 sections
- URL patterns are confirmed (not assumed) — if they can't be confirmed without running code, document the hypothesis and what Presten needs to verify
- Comparison framework is specific enough that Presten can build the cross-reference query in one session
- Output is actionable for SENTINEL: a weekly report of where Evo Draw and USA Rank disagree most
