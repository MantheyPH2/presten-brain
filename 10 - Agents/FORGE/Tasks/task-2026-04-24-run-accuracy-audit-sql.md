---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: blocked
priority: high
due: 2026-04-24
blocked_reason: Requires psql access to youth_soccer DB at localhost:5432. All SQL queries are written and ready (in USA Rank Comparison 2026-04-23.md). Presten must execute or grant FORGE direct DB access.
---

# Task: Execute ELO's USA Rank Accuracy Audit SQL Against the Database

## Context

ELO has written complete SQL queries to cross-check Evo Draw's rankings against 16 USA Rank reference teams (U13G and U14G), but ELO does not have direct database access. The comparison tables in the following documents are **empty** — they contain the USA Rank data but the Evo Draw columns are all `???`:

- `02 - Tiger Tournaments/Projects/Rankings/USA Rank Comparison 2026-04-23.md` — 16 teams, comparison table + gap analysis
- `02 - Tiger Tournaments/Projects/Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` — same teams, methodology divergence analysis

FORGE has DB access. This is a 30-minute task that unblocks calibration validation, confirms or refutes the Football Academy NJ merge concern, and gives Presten an empirical answer to "are our rankings close to USA Rank?"

This is **due today** because the comparison is needed before the calibration approval decision, and the calibration decision is needed before May 1.

## What to Do

### Step 1: Run the Lookup Queries

Open a psql session against `youth_soccer` at localhost:5432 (user `p`). Run these lookups for each of the 16 reference teams. Use the SQL from `Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` — ELO wrote fuzzy match queries there. Adapt as needed for name variations.

Reference team list with USA Rank positions:

**U13G (age_group = 'U13', gender = 'F'):**
1. The Football Academy NJ 2012 Girls Black — USA Rank #4
2. Wall Soccer Club Wall Elite Porto — USA Rank #12
3. FA 12G Black — USA Rank #38
4. FC Stars G2012 ECNL White — USA Rank #121
5. NE Surf G2012 State Navy — USA Rank #198
6. Scorpions SC ECNL G12 — USA Rank #317
7. NYCFC Girls 2012 North — USA Rank #491
8. Dix Hills EST Angel City G2012 — USA Rank #562
9. BVB International Academy Pittsburgh 12G Prem — USA Rank #613
10. Old Line FC Girls 12 Black — USA Rank #741
11. SYC 2012G GA II — USA Rank #826

**U14G (age_group = 'U14', gender = 'F'):**
12. Maryland Rush Azzurri 2011 — USA Rank #144
13. NJ Premier FC G2011 — USA Rank #810
14. Marlton Chiefs Premier 2011 — USA Rank #926
15. Greater Severna Park 11G Green — USA Rank #988
16. Seacoast United MA 2011G MA Elite Navy — USA Rank #1995

For each team:
- Find the team in `teams` (fuzzy match on name, `ILIKE '%keyword%'`)
- Join to `rankings` to get `rating` and `rank`
- Note the exact team name as it appears in our DB

Fuzzy match template:
```sql
SELECT t.id, t.name, r.rating, r.rank, r.age_group, r.gender
FROM teams t
JOIN rankings r ON r.team_id = t.id
WHERE t.name ILIKE '%Football Academy%'
  AND r.age_group = 'U13'
  AND r.gender = 'F'
ORDER BY r.rank ASC
LIMIT 5;
```

If a team is NOT found: try a broader search (`ILIKE '%Surf%'`, `ILIKE '%Scorpions%'`) and note whether the team has any games in the `games` table even without a ranking:
```sql
SELECT COUNT(*) FROM games 
WHERE home_team_name ILIKE '%NE Surf%' OR away_team_name ILIKE '%NE Surf%';
```

### Step 2: Flag the Football Academy NJ / FA 12G Black Identity Question

This is the most important lookup in the set. USA Rank shows:
- "The Football Academy NJ 2012 Girls Black" at #4
- "FA 12G Black" at #38

These may be the same club under two naming conventions. In our DB, they may be:
1. Two separate team records (correct if they are legitimately different)
2. Merged into one canonical record (correct if they are the same team)
3. Present as two records that should be merged but aren't

Run:
```sql
-- Search broadly for Football Academy / FA variations
SELECT t.id, t.name, r.rating, r.rank
FROM teams t
LEFT JOIN rankings r ON r.team_id = t.id AND r.age_group = 'U13' AND r.gender = 'F'
WHERE t.name ILIKE '%Football Academy%' OR t.name ILIKE '%FA 12G%'
ORDER BY r.rank ASC NULLS LAST;
```

Also check `team_merges` — are either of these team IDs listed as a canonical or merged entry?
```sql
SELECT tm.*, t1.name as canonical_name, t2.name as merged_name
FROM team_merges tm
JOIN teams t1 ON t1.id = tm.canonical_team_id
JOIN teams t2 ON t2.id = tm.merged_team_id
WHERE t1.name ILIKE '%Football Academy%' OR t2.name ILIKE '%Football Academy%'
   OR t1.name ILIKE '%FA 12G%' OR t2.name ILIKE '%FA 12G%';
```

Report: Are they one team or two? If two, are they competing in the same events on the same dates (which would be a merge problem)? Pull their game histories for 2024–2025 to check for overlap.

### Step 3: Populate the Comparison Table

Fill in the comparison table from `Rankings/USA Rank Comparison 2026-04-23.md`:

| Team | USA Rank | Evo Draw Rank | Evo Draw Rating | Delta | Notes |
|------|----------|--------------|-----------------|-------|-------|

For each of the 16 teams: fill in our rank and rating, or mark "NOT FOUND" with notes on whether games exist.

### Step 4: Write Findings

Write a brief results document to:
`Reports/usa-rank-accuracy-audit-results-2026-04-24.md`

Include:
1. The completed comparison table (copy from the template, fill in Evo Draw columns)
2. Summary: how many teams found, how many not found, average absolute rank delta for found teams
3. Football Academy NJ / FA 12G Black identity finding — one team or two? What is the recommended action?
4. Any other notable divergences (teams where our rank differs by >200 positions from USA Rank — flag those for ELO to investigate)

Also update the `Rankings/USA Rank Comparison 2026-04-23.md` document directly — add the Evo Draw data to the comparison table so ELO's document is complete.

## Deliverable

1. `Reports/usa-rank-accuracy-audit-results-2026-04-24.md` — results report with filled comparison table and Football Academy NJ merge finding
2. Updated `Rankings/USA Rank Comparison 2026-04-23.md` with Evo Draw columns populated
3. Summary in your 2026-04-24 briefing: how many teams found, average rank delta, Football Academy NJ answer

## Definition of Done

- Comparison table has Evo Draw data for every team found (not `???`)
- Football Academy NJ / FA 12G Black identity is resolved with evidence
- Any team with a >200-rank delta vs USA Rank is flagged for ELO investigation
- ELO's document is updated (not just a separate FORGE report)

## Time Estimate

30–45 minutes of DB queries. This is not an investigation — it is executing already-written SQL and recording the results.

## References

- `02 - Tiger Tournaments/Projects/Rankings/USA Rank Comparison 2026-04-23.md` — ELO's comparison document (update this directly)
- `02 - Tiger Tournaments/Projects/Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` — ELO's audit document with all SQL queries pre-written
- PostgreSQL `youth_soccer` at localhost:5432, user `p`
