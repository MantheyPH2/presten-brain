---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-23
status: completed
completed: 2026-04-23
deliverable: "02 - Tiger Tournaments/Projects/Rankings/USA Rank Ranking Accuracy Audit — April 2026.md"
priority: high
---

# Task: Ranking Accuracy Audit — Cross-Check Evo Draw vs USA Rank for Known Teams

## Context

USA Rank's competitive intel file captured specific ranked teams with their USA Rank positions:
- U13G: The Football Academy NJ 2012 Girls Black (#4), Wall Soccer Club Wall Elite Porto (#12), FC Stars G2012 ECNL White (#121), NE Surf G2012 State Navy (#198)
- U14G: Maryland Rush Azzurri 2011 (#144), NJ Premier FC G2011 (#810)

These are data points we can act on today without a scraper. SENTINEL needs to know: where do Evo Draw's rankings put these same teams? If our #4 is USA Rank's #198, that is a quality problem. If our rankings broadly agree with USA Rank's for high-profile teams, that validates our methodology.

This audit also serves a second purpose: identifying whether our data gaps (source coverage) explain ranking divergence, or whether our methodology itself diverges from USA Rank's.

Read before starting:
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — "Public Ranked Team Data" section (the specific teams captured), "How They Rank Teams" section (methodology differences)
- `02 - Tiger Tournaments/Projects/Rankings/Ranking Engine.md` — our Glicko v7 methodology, 9 phases, how calibration values work
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — teams and rankings table structure

## What You Need to Produce

### 1. Team Lookup Query
Write a SQL query to look up each USA Rank team in our database:
```sql
SELECT t.id, t.name, r.rating, r.rank, r.age_group, r.gender
FROM teams t
JOIN rankings r ON r.team_id = t.id
WHERE t.name ILIKE '%Football Academy%'
  AND r.age_group = 'U13'
  AND r.gender = 'F'
ORDER BY r.rank ASC
LIMIT 10;
```
Adapt this pattern for each of the 6 known teams. Account for name variations — team names in our DB may not exactly match USA Rank's names (e.g., "The Football Academy NJ" vs "Football Academy NJ 2012 Girls Black").

Run these queries and document the results. If Presten hasn't run them, write the queries and explain exactly how to run and interpret them.

### 2. Comparison Table
Build a comparison table for all 6 known teams:

| Team | USA Rank Rank | Evo Draw Rank | Delta | Notes |
|------|--------------|--------------|-------|-------|
| Football Academy NJ U13G | #4 | ??? | ??? | |
| Wall SC Wall Elite Porto U13G | #12 | ??? | ??? | |
| FC Stars G2012 ECNL White U13G | #121 | ??? | ??? | |
| NE Surf G2012 State Navy U13G | #198 | ??? | ??? | |
| Maryland Rush Azzurri U14G | #144 | ??? | ??? | |
| NJ Premier FC G2011 U14G | #810 | ??? | ??? | |

For each team:
- If found in our DB: report rank, rating, rank delta vs USA Rank
- If NOT found in our DB: flag as "missing" — this is a data gap (either team isn't in our system, or name doesn't match)
- If found but ranked very differently: flag for methodology investigation

### 3. Delta Analysis
- What is the average absolute rank delta across teams we can find?
- Are we consistently ranking teams higher or lower than USA Rank? (Systematic bias?)
- For teams with large deltas (>100 rank positions): hypothesize why. Possible causes:
  a. Source gap — we don't have games from the events USA Rank has
  b. Methodology difference — USA Rank uses last 20 games / goal difference; we use Glicko v7 / full history
  c. League calibration difference — our hardcoded calibration values vs their emergent inter-league approach
  d. Team merge issue — we may have split or merged the team incorrectly

### 4. Missing Team Analysis
For teams NOT found in our DB:
- Are they in the teams table under a different name? (Run broader name search)
- Have they ever had games in our system? (`SELECT COUNT(*) FROM games WHERE home_team_name ILIKE '%Football Academy%'`)
- If they have games but no ranking, why? (Too few games? Inactivity? Team merge ate them?)
- If they have no games at all, this is a source gap — which of USA Rank's sources would have their games?

### 5. Methodology Divergence Assessment
Even for teams with small rank deltas, document what the methodology difference implies:
- USA Rank uses **last 20 games in last year** — does this favor teams with high recent form over our full-history approach?
- USA Rank uses **goal difference** as the rating signal — our Glicko uses win/loss/draw. For what type of team would this produce the biggest ranking divergence? (High-scoring dominant teams? Teams that win ugly?)
- USA Rank's **inter-league calibration is emergent** (from cross-league results). Our calibration is hardcoded. For the NJ/NE teams in this sample, which leagues are they in, and does our calibration for those leagues seem correct?

### 6. Recommendations
Based on the audit:
1. List any data gaps confirmed (source coverage issues driving ranking divergence)
2. List any methodology concerns flagged (cases where Glicko v7 produces rankings that seem clearly wrong vs a real-world-validated benchmark like USA Rank #4)
3. Recommended follow-up tasks for SENTINEL to assign (max 3 — be specific)

## What Done Looks Like

Produce an audit report at:
`02 - Tiger Tournaments/Projects/Rankings/USA Rank Ranking Accuracy Audit — April 2026.md`

The report must include:
1. The SQL lookup queries (ready to run)
2. Completed comparison table (with Evo Draw results filled in if DB access is available, or clearly marked as "pending DB run" if not)
3. Delta analysis with classification of divergence causes
4. Missing team investigation results
5. Methodology divergence assessment
6. Ranked list of recommendations (most impactful first)

## Success Criteria

- Audit report written with all 6 sections
- Comparison table is as complete as possible — do not leave it empty
- If DB access isn't available to ELO directly, the SQL queries are written precisely enough that Presten can run them and fill in the table in 15 minutes
- At least one concrete finding: either "our rankings broadly agree with USA Rank for these elite teams" OR "we have a systematic divergence that needs investigation" — not "it depends"
- Recommendations are specific and actionable (assign to FORGE or ELO with a clear deliverable)
