---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-23
status: completed
completed: 2026-04-23
deliverables: "Reports/unverified-merges-analysis-2026-04-23.md, Reports/high-risk-merges-2026-04-23.md"
note: "CSV (unverified-merges-top-2000) requires DB execution by FORGE/Presten"
priority: medium
---

# Task: Team Merges Verification Campaign — Top 2,000 Unverified Merges

## Context

Strategic Insights Insight 2 identifies the `team_merges` table as Evo Draw's primary competitive moat: "the team_merges table IS the moat." 27,820 total merge records link the same team across GotSport, TGS, Sinc Sports, and other sources. But only 18,684 of those are verified. The remaining **9,136 unverified merges** could contain errors that silently corrupt rankings for any team that touches a bad merge.

Strategic Insights Action Item 6 calls for a verification campaign targeting the top 2,000 unverified merges by game volume. This task executes that campaign.

This is assigned to ELO because merge errors corrupt ratings — you are directly affected when a team's game history includes games from a misidentified duplicate. Understanding which merges are suspect informs both the ranking quality picture and the calibration work.

## What to Do

### Step 1: Identify the Top 2,000 Unverified Merges by Impact

Query the `team_merges` table for unverified records:

```sql
-- Find unverified merges with highest game volume (most impactful if wrong)
SELECT 
  tm.merge_id,
  tm.canonical_team_id,
  tm.merged_team_id,
  tm.merge_method,  -- fuzzy_match, coach_name, manual, etc.
  tm.confidence_score,  -- if this field exists
  t1.team_name as canonical_name,
  t2.team_name as merged_name,
  COUNT(g.id) as total_games_affected
FROM team_merges tm
JOIN teams t1 ON tm.canonical_team_id = t1.team_id
JOIN teams t2 ON tm.merged_team_id = t2.team_id
JOIN games g ON (g.home_team_id = tm.merged_team_id OR g.away_team_id = tm.merged_team_id)
WHERE tm.verified = false  -- or equivalent unverified flag
  AND g.is_hidden = false
GROUP BY tm.merge_id, tm.canonical_team_id, tm.merged_team_id, tm.merge_method, 
         tm.confidence_score, t1.team_name, t2.team_name
ORDER BY total_games_affected DESC
LIMIT 2000;
```

(Adjust column names to match the actual `team_merges` schema documented in [[Database Schema]].)

Write the full list to: `Reports/unverified-merges-top-2000-2026-04-23.csv`

### Step 2: Classify Merge Method Risk

Group the 2,000 records by how the merge was generated and assess the error rate risk for each method:

| Merge Method | Count in Top 2K | Estimated Error Rate | Rationale |
|--------------|-----------------|----------------------|-----------|
| exact_name_match | ? | Low (~2-5%) | Same name = likely same team |
| fuzzy_name_match | ? | Medium (~10-20%) | Fuzzy matching produces false positives |
| coach_name | ? | Medium-Low (~5-10%) | Coach names can transfer between teams |
| cross_source_id | ? | Low (~3-7%) | If GotSport ID matches TGS record, high confidence |
| manual | ? | Very Low (<2%) | Human reviewed |
| Other/Unknown | ? | Unknown | Flag for investigation |

Report the distribution and total games affected by each method category.

### Step 3: Auto-Verify High-Confidence Records

For records where auto-verification is safe, mark them verified without manual review:

**Auto-verify criteria (ALL must be true):**
1. Merge method = `exact_name_match` AND
2. Both teams have the same age group AND
3. Both teams have games in overlapping date ranges (confirming they are the same team over time, not two different teams with the same name) AND
4. At least one of the team IDs is a GotSport API ID (most reliable source)

Write the SQL to mark these verified (do not execute without Presten's approval — stage the update and report the count):

```sql
-- STAGED (do not execute without approval)
UPDATE team_merges
SET verified = true, verified_by = 'ELO_AUTO_2026-04-23', verified_at = NOW()
WHERE merge_id IN (
  -- [auto-verify candidate IDs from the Step 3 criteria above]
);
```

Report: how many of the top 2,000 would be auto-verified under these criteria.

### Step 4: Flag High-Risk Unverified Merges

Identify the records in the top 2,000 that are most likely to be errors:

**High-risk merge indicators:**
- Fuzzy name match + different age groups (a U15 team merged with a U16 team = likely wrong)
- Fuzzy name match + teams from different geographic regions (if geographic data available)
- Coach name match + teams with non-overlapping date ranges (coach moved; teams are not the same)
- Any merge where the canonical team's current Elo is more than 200 points away from the merged team's Elo at the time of merge (suggests very different performance levels = possible wrong merge)

For each high-risk merge: output a record showing `canonical_name`, `merged_name`, `merge_method`, `games_affected`, `risk_reason`.

Write to: `Reports/high-risk-merges-2026-04-23.md` — include the top 50 highest-risk merges with the most games affected. These are the ones most worth Presten's time to manually review.

### Step 5: Quantify the Rating Impact of Potential Bad Merges

For the top 20 highest-risk merges by game volume:
- If the merge is wrong, how many games would be incorrectly attributed to the canonical team?
- What is the estimated Elo rating impact of removing those games? (Use the current Glicko-2 machinery to estimate: remove the merged team's games from the canonical team's history and recalculate)
- Report: "If merge ID X is wrong, the canonical team's rating shifts by approximately Y points"

This is the severity analysis. It tells us which bad merges matter most to fix.

## What to Read

- `/Users/p/Documents/Obsidian Vault/02 - Tiger Tournaments/Projects/Strategic Insights.md` — Insight 2 (the moat), Action Item 6
- `[[Dedup Strategy]]` vault note — `team_merges` table structure, merge methods, verification logic
- `[[Database Schema]]` vault note — exact column names in `team_merges` table
- `[[Ranking Engine]]` — Phase 1 (how team merges feed into rankings computation)

## What to Produce

1. `Reports/unverified-merges-top-2000-2026-04-23.csv` — ranked list of top 2,000 unverified merges by game volume
2. `Reports/unverified-merges-analysis-2026-04-23.md` — summary analysis with:
   - Merge method distribution and risk classification
   - Count of auto-verifiable records + staged SQL
   - Top 50 high-risk merges with risk reason
   - Severity analysis for top 20 highest-risk records (estimated Elo impact if wrong)
3. `Reports/high-risk-merges-2026-04-23.md` — the top 50 high-risk merge records formatted for Presten's review

## Definition of Done

- The top 2,000 unverified merges by game volume are identified and on record
- The merge method risk classification exists with game volume for each category
- The staged auto-verify SQL is ready (count of records it would affect is known)
- The top 50 high-risk merges are flagged with rating impact severity

## Why This Matters for ELO

Your Brier score analysis is measuring calibration against teams whose game histories may include cross-platform merge errors. If a significant fraction of the 9,136 unverified merges are wrong, the Brier scores you calculated are measuring the combined effect of calibration error AND merge error. Separating these would improve the precision of the calibration validation. Start with the highest game-volume merges first — those have the largest effect on rating computation.
