---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-23
status: complete
completed: 2026-04-23
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Rank Bands Design.md"
priority: high
---

# Task: Design Gold/Silver/Bronze/Red/Blue/Green Rank Band System

## Context

USA Rank uses a 6-tier band system to label teams by rank range. We currently have no rank bands — teams just have a numeric Elo rating and ranking position. Adding bands gives users an immediate visual signal of team quality, makes rankings browsable without needing to know the exact numbers, and closes a visible UI gap vs our top competitor.

USA Rank's bands (for reference, do NOT copy blindly — design ours correctly for our data):
| Band | Rank Range |
|------|-----------|
| Gold | 1–100 |
| Silver | 101–250 |
| Bronze | 251–400 |
| Red | 401–700 |
| Blue | 701–1050 |
| Green | 1051+ |

Our system must reflect our actual Elo rating distribution, not USA Rank's rank-count-based thresholds.

Read before starting:
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — Ranking Bands table and How They Rank Teams section
- `02 - Tiger Tournaments/Projects/Rankings/Ranking Engine.md` — our Elo/Glicko v7 methodology, 9 phases
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — teams and rankings tables
- `02 - Tiger Tournaments/Projects/Infrastructure/Server and Frontend.md` — current frontend, API endpoints

## What You Need to Design

### 1. Rating Distribution Analysis
- Query the rankings table: what is the distribution of Elo ratings across all ranked teams?
- Key statistics needed: min, max, median, p10, p25, p75, p90, p95, p99
- How many teams are currently ranked per age group (U11–U19, boys and girls separately)?
- Are rating distributions consistent across age groups, or does U13G look different from U17B?
- Provide this as a SQL query Presten can run: `SELECT age_group, gender, MIN(rating), MAX(rating), PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating) as median FROM rankings GROUP BY age_group, gender`

### 2. Band Threshold Design

Two approaches to evaluate:

**Approach A: Rank-position bands** (like USA Rank — top N teams per age group get Gold, etc.)
- Pros: Easy to explain ("top 100"), scales automatically as more teams rank
- Cons: Percentile thresholds shift as team count grows; Gold means different things in big vs small age groups

**Approach B: Rating-value bands** (fixed Elo ranges, e.g., 1600+ = Gold)
- Pros: Objective, comparable across age groups, doesn't change as teams enter/exit
- Cons: Requires knowing the rating distribution; bands may be uneven in population size

For each approach:
- Define the 6 threshold values for each band (Gold, Silver, Bronze, Red, Blue, Green)
- Estimate how many teams currently fall in each band (per age group)
- Evaluate: which is more stable over time? Which is more intuitive to users?

**Recommended approach:** Design Approach B (rating-based) as primary, with a note on Approach A.

### 3. Band Display Design
- What color should each band display as? (Gold = #FFD700, Silver = #C0C0C0, Bronze = #CD7F32, Red = #DC2626, Blue = #2563EB, Green = #16A34A — confirm or revise)
- Should bands include a numeric rank within the band? ("Gold #47")
- Should inactive teams (7+ months no games) lose their band? What's our inactivity threshold?
- How should unranked teams (fewer than 6–10 results) display? ("Unranked" or hidden?)

### 4. Database Implementation
- Where do bands get stored? (Computed column on rankings table, or derived at query time?)
- Write the SQL function or CASE statement that maps a rating to a band label
- Should band be stored or always computed? Stored is faster for queries; computed is always fresh.
- Proposed: add a `rank_band` computed column or a SQL view `rankings_with_bands`

### 5. API Design
- Current rankings API returns: [team, rating, rank]. Add `band` and `band_color` fields.
- Endpoint change: `GET /api/rankings?age_group=U14G` → response includes `band: "Gold"`, `band_color: "#FFD700"`
- Should there be a filter endpoint? `GET /api/rankings?band=Gold&age_group=U14G`

## What Done Looks Like

Produce a design document at:
`02 - Tiger Tournaments/Projects/Rankings/Rank Bands Design.md`

The document must include:
1. Rating distribution table (min, max, median, percentiles) — with actual numbers from the DB or estimated from known data
2. Band threshold table — 6 bands × threshold values (Approach B, rating-based)
3. Estimated team count per band per age group
4. SQL CASE statement implementing the band mapping
5. SQL for `rankings_with_bands` view
6. API response schema change (before/after JSON example)
7. Color palette (hex codes for all 6 bands)
8. Inactivity handling rule
9. Implementation order: DB first → API second → Frontend third

## Success Criteria

- Design document written with all 9 sections
- SQL artifacts are production-ready (run against `youth_soccer` DB without modification)
- Band thresholds are grounded in actual rating distribution, not arbitrary numbers
- Frontend integration is scoped — Presten knows exactly what to add to the React components
- Feature is achievable before DSS (May 16, 2026) — implementation estimate included
