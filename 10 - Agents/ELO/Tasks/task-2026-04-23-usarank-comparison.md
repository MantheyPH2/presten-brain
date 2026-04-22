---
type: agent-task
assigned_to: ELO
assigned_by: Presten
date: 2026-04-23
status: completed
completed: 2026-04-23
deliverable: "02 - Tiger Tournaments/Projects/Rankings/USA Rank Comparison 2026-04-23.md"
priority: high
---

# PRIORITY: Compare Evo Draw Rankings Against USA Rank

Presten says our rankings should land close to USA Rank's. If they don't, our calibration is wrong. Validate this NOW.

## Reference Data from USA Rank (public)

### U13 Girls
- The Football Academy NJ 2012 Girls Black — USA Rank #4
- Wall Soccer Club Wall Elite Porto — USA Rank #12
- FA 12G Black — USA Rank #38
- FC Stars G2012 ECNL White — USA Rank #121
- NE Surf G2012 State Navy — USA Rank #198
- Scorpions SC ECNL G12 — USA Rank #317
- NYCFC Girls 2012 North — USA Rank #491
- Dix Hills EST Angel City G2012 — USA Rank #562
- BVB International Academy Pittsburgh 12G Prem — USA Rank #613
- Old Line FC Girls 12 Black — USA Rank #741
- SYC 2012G GA II — USA Rank #826

### U14 Girls
- Maryland Rush Azzurri 2011 — USA Rank #144
- NJ Premier FC G2011 — USA Rank #810
- Marlton Chiefs Premier 2011 — USA Rank #926
- Greater Severna Park 11G Green — USA Rank #988
- Seacoast United MA 2011G MA Elite Navy — USA Rank #1995

## What To Do

1. **Query the Evo Draw database** for each of these teams:
   - Search by team name in the `teams` table (fuzzy match — names may differ slightly)
   - Pull their current Elo/Glicko rating and our internal rank
   - Note any teams we can't find (means we're missing data for them)

2. **Build a comparison table:**

| Team | USA Rank | Evo Draw Rank | Evo Draw Rating | Delta | Notes |
|------|----------|--------------|-----------------|-------|-------|
| Football Academy NJ 2012 Girls Black | #4 | ? | ? | ? | |

3. **Analyze the gaps:**
   - Are our top teams in the same general tier? (Top 10 should be Top 20, etc.)
   - Are there teams USA Rank has that we don't have at all? (= missing source data)
   - Where do we rank teams significantly differently? Why?
   - Is the overall ordering similar even if exact numbers differ?

4. **If rankings are way off**, identify the root cause:
   - Missing game data for certain teams?
   - Calibration values wrong for their league?
   - Dedup issues combining wrong teams?
   - Our algorithm weighting history differently than USA Rank's "last 20 games" approach?

5. **Write findings to:** `02 - Tiger Tournaments/Projects/Rankings/USA Rank Comparison 2026-04-23.md`

Include:
- Full comparison table
- Correlation analysis (how close are our orderings)
- Identified gaps and root causes
- Specific recommendations to bring our rankings in line
- Which teams we're missing entirely (source coverage gap)

## Access

The Evo Draw database is PostgreSQL `youth_soccer` at localhost:5432, user `p`. Key tables:
- `teams` — team names and IDs
- `rankings` or rating columns — current Elo values
- `games` — match history
- `team_merges` — name aliases

You may need to query the actual database. If you can't access it, document exactly what SQL queries would need to be run and flag it for FORGE to execute.

## Why This Matters

If our rankings don't match USA Rank's for the same teams, coaches and parents won't trust us. We need to be at least as accurate — then better. This comparison tells us exactly where we stand.

## Definition of Done
- Comparison table with at least 15 teams matched
- Gap analysis identifying why any significant ranking differences exist
- Specific action items to close accuracy gaps
- List of teams we're missing entirely (source coverage problem)
