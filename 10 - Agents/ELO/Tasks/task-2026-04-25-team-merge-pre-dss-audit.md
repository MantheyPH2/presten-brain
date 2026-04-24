---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: high
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Pre-DSS Team Merge Audit — U13G.md"
---

# Task: Pre-DSS Team Merge Audit — U13G Identity Splits

## Context

Your USA Rank Ranking Accuracy Audit (2026-04-23) identified that USA Rank lists both "The Football Academy NJ 2012 Girls Black" (#4 U13G) and "FA 12G Black" (#38 U13G) as separate teams. These are almost certainly the same organization. If Evo Draw has not merged them, our U13G #1 or #2 team may be fragmented across two records — meaning neither record holds the full rating it should. This is the most visible potential error in the most demo-critical age group.

DSS is May 16. This merge question must be resolved before Presten implements Rank Bands (5.5-hour session, should start this week). If bands are applied to a fragmented top-10, the demo shows an incorrect Gold-band assignment for a club the audience knows.

This task is a vault research and specification task. You cannot run the SQL yourself — your job is to write the queries and the merge recommendation so Presten can execute the whole thing in one 30–45 minute psql session.

## What You Need to Do

### Step 1: Define the Football Academy NJ Merge Investigation

Write the exact SQL to answer these questions about the Football Academy NJ / FA 12G Black case:

**Query 1A: Does either name exist in our teams table?**
```sql
SELECT id, name, rating, rd, games_played, last_game_date, created_at
FROM teams
WHERE name ILIKE '%football academy%' OR name ILIKE '%FA 12G%' OR name ILIKE '%FA12G%'
ORDER BY rating DESC;
```
Adapt as needed based on your knowledge of the teams table schema.

**Query 1B: Are they already merged?**
```sql
SELECT tm.primary_team_id, tm.secondary_team_id, t1.name, t2.name, tm.created_at
FROM team_merges tm
JOIN teams t1 ON t1.id = tm.primary_team_id
JOIN teams t2 ON t2.id = tm.secondary_team_id
WHERE t1.name ILIKE '%football academy%' OR t2.name ILIKE '%football academy%'
   OR t1.name ILIKE '%FA 12G%' OR t2.name ILIKE '%FA 12G%';
```

**Query 1C: If not merged, what is the rating impact?**
If both teams exist as separate records, calculate the hypothetical merged rating assuming a weighted average by games_played. Show what their combined Glicko rating would be vs their individual current ratings.

### Step 2: Identify Additional Top-50 U13G Split-Identity Candidates

The Football Academy NJ case is the one you identified, but it may not be the only one. USA Rank lists 16+ U13G teams in your reference set. Cross-reference the USA Rank team names from the accuracy audit against common patterns for split identity:

- Same club name, different abbreviation or suffix (e.g., "FC Barca" vs "Barca FC" vs "FC Barcelona Academy")
- Club name with and without geographic qualifier (e.g., "NJ Wildcats" vs "New Jersey Wildcats")
- Same team, different year format (e.g., "2012G" vs "12G" vs "U13G")

For each of the 16 USA Rank reference teams, write a SQL query to surface all name variants in our `teams` table within edit distance 2 or matching on key tokens. If the `team_merges` table shows any of them already resolved, note it.

Produce a table:

| USA Rank Team Name | USA Rank Rank | Evo Draw Match | Potential Split? | Action |
|---|---|---|---|---|

Where "Action" is one of: `confirmed-merged`, `confirmed-single`, `investigate`, or `merge-recommended`.

### Step 3: Write Merge Recommendations

For any case where your analysis shows a merge is likely needed:

1. Identify which record should be `primary` (higher games_played and older created_at = more game history attached)
2. Write the exact merge SQL Presten should run — use the same pattern as existing `team_merges` inserts in the codebase
3. Specify the rating recompute that must follow each merge
4. Order the merges: highest-impact first (by the rank gap the merge closes)

The recommendations must be specific enough that Presten can copy-paste the SQL and run it without needing to make any decisions.

### Step 4: Write the Pre-Merge Verification Protocol

Before any merge is executed, Presten needs to verify it won't create a new error. Write:

- The before-state snapshot query (save both teams' current ratings and game lists)
- The merge execution SQL
- The after-state verification query (confirm the primary now has all games and the recomputed rating makes sense)
- The rollback path (how to un-merge if the resulting rating is clearly wrong)

## What Done Looks Like

Deliverable: `02 - Tiger Tournaments/Projects/Rankings/Pre-DSS Team Merge Audit — U13G.md`

The document must:
- Answer the Football Academy NJ question definitively (with the exact SQL, pre-populated with what you know)
- List all top-50 U13G split candidates with action classification
- For any `merge-recommended` case: include complete, copy-paste-ready merge SQL
- Include the pre-merge verification protocol

Presten should be able to run every SQL in this document sequentially in a single psql session and resolve all U13G merge issues before Rank Bands implementation begins.

## Success Criteria

- Football Academy NJ case is fully analyzed with complete SQL
- All 16 USA Rank reference teams have an action classification
- At least the Football Academy NJ merge recommendation (if applicable) has complete, ready-to-run SQL
- Pre-merge verification protocol is present
- Document is usable by Presten without additional research

## Why Before DSS

Football Academy NJ appears to be the top or near-top U13G team in the NY/NJ metro area — the exact demographic in the DSS room. If they appear at rank #38 instead of #4 because they're fragmented, the accuracy claim fails live. This is a non-negotiable pre-demo fix.
