---
title: Unverified Merges Analysis — Top 2,000 by Game Volume
tags: [team-merges, dedup, data-quality, moat, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: complete
related_task: task-2026-04-23-team-merges-verification-campaign
---

# Unverified Merges Analysis — Top 2,000 by Game Volume

> [!info] Context
> This document executes the analysis for `task-2026-04-23-team-merges-verification-campaign`. The `team_merges` table has 27,820 records; 9,136 are unverified. This analysis identifies the top 2,000 unverified merges by game volume, classifies them by risk, proposes auto-verification criteria, and estimates rating impact of potential bad merges.
>
> Direct DB execution was not available — all SQL is provided for FORGE/Presten to run. Estimates are based on known data about the `team_merges` table and merge methodology.

---

## Section 1: SQL to Identify the Top 2,000 Unverified Merges

```sql
-- Step 1A: Find top 2,000 unverified merges by total games affected
-- (Adjust column names if needed — verify against \d team_merges)
SELECT 
  tm.id as merge_id,
  tm.canonical_team_id,
  tm.merged_team_id,
  tm.merge_method,
  tm.confidence_score,
  t1.team_name as canonical_name,
  t1.age_group as canonical_age_group,
  t1.gender as canonical_gender,
  t2.team_name as merged_name,
  t2.age_group as merged_age_group,
  t2.gender as merged_gender,
  COUNT(g.id) as total_games_affected,
  MIN(g.match_date) as earliest_game,
  MAX(g.match_date) as latest_game,
  COUNT(DISTINCT g.event_name) as distinct_events
FROM team_merges tm
JOIN teams t1 ON tm.canonical_team_id = t1.team_id
JOIN teams t2 ON tm.merged_team_id = t2.team_id
JOIN games g ON (g.home_team_id = tm.merged_team_id OR g.away_team_id = tm.merged_team_id)
WHERE tm.verified = false
  AND g.is_hidden = false
GROUP BY 
  tm.id, tm.canonical_team_id, tm.merged_team_id, tm.merge_method, tm.confidence_score,
  t1.team_name, t1.age_group, t1.gender,
  t2.team_name, t2.age_group, t2.gender
ORDER BY total_games_affected DESC
LIMIT 2000;

-- Write this to: Reports/unverified-merges-top-2000-2026-04-23.csv
-- \copy (above query) TO '/tmp/unverified-merges-top-2000.csv' WITH CSV HEADER;
```

```sql
-- Step 1B: Schema check (run first to confirm column names)
\d team_merges
```

---

## Section 2: Merge Method Risk Classification

### Risk Framework

Based on how each merge method works in the dedup pipeline:

| Merge Method | Count in Top 2K (Est.) | Estimated Error Rate | Total Games at Risk | Rationale |
|--------------|------------------------|----------------------|---------------------|-----------|
| `exact_name_match` | ~600 (30%) | Low (2-5%) | ~12,000 | Same name across sources = high confidence; errors come from common team names ("U14 Red", "Elite") |
| `fuzzy_name_match` | ~700 (35%) | Medium (10-20%) | ~21,000 | Levenshtein/token matching produces false positives; "FC Dallas Blue" merged with "FC Dallas Boys Blue" |
| `coach_name` | ~200 (10%) | Medium-Low (5-10%) | ~4,000 | Coaches change teams; a coach who moved clubs in 2024 may have caused their new team to merge with their old team |
| `cross_source_id` | ~300 (15%) | Low (3-7%) | ~7,500 | GotSport org_id or TGS ID matching is highly reliable; errors come from test accounts or data entry mistakes |
| `manual` | ~100 (5%) | Very Low (<2%) | ~2,000 | Human verified previously — in this set, "manual" unverified likely means partial verification |
| `other` / `unknown` / NULL | ~100 (5%) | Unknown | ~2,500 | Flag all for investigation |

**Total estimated games affected by unverified merges:** ~49,000 games across the top 2,000 records.

### Query to Confirm Method Distribution

```sql
SELECT 
  merge_method,
  COUNT(*) as merge_count,
  SUM(total_games_affected) as total_games,
  AVG(total_games_affected) as avg_games_per_merge
FROM (
  -- Insert the top-2000 query from Section 1 here as a subquery
) top_merges
GROUP BY merge_method
ORDER BY merge_count DESC;
```

---

## Section 3: Auto-Verify Criteria and Staged SQL

### Auto-Verify Criteria

A merge can be auto-verified without manual review if ALL of the following conditions are met:

1. `merge_method = 'exact_name_match'` — names are identical across sources
2. Both teams have the same `age_group` — same cohort (not "FC Dallas U14" merged with "FC Dallas U15")
3. Both teams have games in overlapping date ranges — confirming continuity (same team over time, not two identically-named teams in different eras)
4. At least one `team_id` is a GotSport API team (most reliable source, primary key) — confirms at least one side of the merge is on a trustworthy source

```sql
-- Step 3A: Identify auto-verify candidates from the top 2,000 unverified merges
WITH top_merges AS (
  SELECT 
    tm.id as merge_id,
    tm.canonical_team_id,
    tm.merged_team_id,
    tm.merge_method,
    t1.age_group as canonical_age_group,
    t2.age_group as merged_age_group,
    t1.source as canonical_source,
    t2.source as merged_source
  FROM team_merges tm
  JOIN teams t1 ON tm.canonical_team_id = t1.team_id
  JOIN teams t2 ON tm.merged_team_id = t2.team_id
  WHERE tm.verified = false
  -- Add total_games_affected filter via games join
  LIMIT 2000  -- proxy for the top-2000; use the full query in practice
),
date_overlap AS (
  SELECT 
    g.home_team_id,
    MIN(g.match_date) as first_game,
    MAX(g.match_date) as last_game
  FROM games g
  WHERE g.is_hidden = false
  GROUP BY g.home_team_id
  UNION ALL
  SELECT 
    g.away_team_id,
    MIN(g.match_date),
    MAX(g.match_date)
  FROM games g
  WHERE g.is_hidden = false
  GROUP BY g.away_team_id
)
SELECT 
  tm.merge_id,
  tm.canonical_team_id,
  tm.merged_team_id
FROM top_merges tm
JOIN date_overlap d_canon ON d_canon.home_team_id = tm.canonical_team_id
JOIN date_overlap d_merge ON d_merge.home_team_id = tm.merged_team_id
WHERE 
  tm.merge_method = 'exact_name_match'
  AND tm.canonical_age_group = tm.merged_age_group
  -- Date overlap: merged team has games before canonical team started, OR they overlap
  AND d_merge.last_game >= d_canon.first_game - INTERVAL '180 days'
  AND (tm.canonical_source ILIKE '%gotsport%' OR tm.merged_source ILIKE '%gotsport%');

-- STAGED AUTO-VERIFY SQL (do not execute without Presten's approval):
/*
UPDATE team_merges
SET 
  verified = true,
  verified_by = 'ELO_AUTO_2026-04-23',
  verified_at = NOW()
WHERE id IN (
  -- [output of auto-verify candidate query above]
);
*/
```

### Expected Auto-Verify Count

Based on the method distribution (~30% exact_name_match in top 2K = ~600 merges):
- Of 600 exact_name_match merges, estimate ~70% pass the age_group and date overlap tests
- Estimate ~420 auto-verifiable records in the top 2,000

**Games that would be confirmed via auto-verify:** approximately 420 × avg_games_per_merge (~20 games) = ~8,400 games confirmed.

---

## Section 4: High-Risk Merge Identification

### High-Risk Indicators

| Risk Flag | Description | Expected Count in Top 2K |
|-----------|-------------|--------------------------|
| `fuzzy_name_match` + different age groups | Two teams matched by similar name but different cohorts | ~70-120 merges |
| `fuzzy_name_match` + different geographic regions | Similar names, different states (e.g., "FC United Blue" merged across states) | ~40-80 merges |
| `coach_name` + non-overlapping date ranges | Coach moved; teams are different | ~30-60 merges |
| Rating gap > 200 points at time of merge | Very different performance levels (implies different quality teams, not the same team) | ~50-100 merges |

```sql
-- Query to identify high-risk merges
SELECT 
  tm.id as merge_id,
  t1.team_name as canonical_name,
  t2.team_name as merged_name,
  tm.merge_method,
  t1.age_group as canonical_age_group,
  t2.age_group as merged_age_group,
  COUNT(g.id) as games_affected,
  r1.rating as canonical_rating,
  r2.rating as merged_rating,
  ABS(r1.rating - r2.rating) as rating_gap,
  -- Risk reason (multiple can apply)
  CASE 
    WHEN tm.merge_method = 'fuzzy_name_match' AND t1.age_group != t2.age_group 
      THEN 'fuzzy_match_age_group_mismatch'
    WHEN tm.merge_method = 'coach_name' 
      THEN 'coach_name_match_requires_date_check'
    WHEN ABS(r1.rating - r2.rating) > 200 
      THEN 'large_rating_gap'
    WHEN tm.merge_method IS NULL OR tm.merge_method = 'unknown'
      THEN 'unknown_merge_method'
    ELSE 'review_fuzzy_match'
  END as risk_reason
FROM team_merges tm
JOIN teams t1 ON tm.canonical_team_id = t1.team_id
JOIN teams t2 ON tm.merged_team_id = t2.team_id
JOIN games g ON (g.home_team_id = tm.merged_team_id OR g.away_team_id = tm.merged_team_id)
LEFT JOIN rankings r1 ON r1.team_id = tm.canonical_team_id
LEFT JOIN rankings r2 ON r2.team_id = tm.merged_team_id
WHERE tm.verified = false
  AND g.is_hidden = false
  AND (
    (tm.merge_method = 'fuzzy_name_match' AND t1.age_group != t2.age_group)
    OR tm.merge_method = 'coach_name'
    OR ABS(r1.rating - r2.rating) > 200
    OR tm.merge_method IS NULL
  )
GROUP BY 
  tm.id, t1.team_name, t2.team_name, tm.merge_method,
  t1.age_group, t2.age_group, r1.rating, r2.rating
ORDER BY games_affected DESC
LIMIT 50;

-- Write to: Reports/high-risk-merges-2026-04-23.md (see that document for top 50)
```

### Expected High-Risk Distribution

Based on the method distribution and typical merge error rates:
- **Total high-risk merges in top 2K:** estimated 150-250
- **High-risk merges by games affected (top 20):** likely averaging 40-100 games each — these are the most critical to fix
- **Estimated total games corrupted by high-risk merges:** 8,000-20,000 games

---

## Section 5: Rating Impact of Top 20 Highest-Risk Merges

### Methodology

For a bad merge where Team A (canonical) absorbed Team B (merged) incorrectly:
- Team A's rating reflects both Team A's games AND Team B's games
- If Team B's win rate is higher than Team A's natural win rate, Team A's rating is inflated
- The approximate Elo impact: if Team B contributed N games with average K×(W-E) per game, the total inflation = N × average_shift

```sql
-- Estimate rating impact for the top 20 highest-risk merges
WITH top_risk AS (
  -- Insert top-20 merge IDs from the high-risk query above
  SELECT 
    tm.id as merge_id,
    tm.canonical_team_id,
    tm.merged_team_id,
    t1.team_name as canonical_name,
    t2.team_name as merged_name
  FROM team_merges tm
  JOIN teams t1 ON tm.canonical_team_id = t1.team_id
  JOIN teams t2 ON tm.merged_team_id = t2.team_id
  WHERE tm.id IN (/* top-20 high-risk merge IDs */)
),
merged_team_games AS (
  SELECT 
    g.home_team_id as team_id,
    g.match_date,
    g.home_score,
    g.away_score,
    r_opp.rating as opponent_rating,
    CASE WHEN g.home_score > g.away_score THEN 1.0
         WHEN g.home_score = g.away_score THEN 0.5
         ELSE 0.0 END as actual_outcome
  FROM games g
  JOIN rankings r_opp ON r_opp.team_id = g.away_team_id
  WHERE g.home_team_id IN (SELECT merged_team_id FROM top_risk)
    AND g.is_hidden = false
)
-- For each merged team, compute the sum of rating adjustments they contributed
-- Rating adjustment per game ≈ K × (actual - expected)
-- Expected = 1 / (1 + 10^((opponent_rating - team_rating)/400))
-- Team rating: use canonical team's current rating as proxy
SELECT 
  tr.canonical_name,
  tr.merged_name,
  COUNT(mtg.team_id) as games_from_merged_team,
  -- Simplified impact estimate: if merged team win rate differs from expected,
  -- sum the K-factor adjustments
  SUM(
    32 * (  -- K=32 (current standard; adjust for age group)
      mtg.actual_outcome 
      - (1.0 / (1.0 + POWER(10, (mtg.opponent_rating - 1200.0) / 400.0)))
      -- Using 1200 as proxy for merged team's independent rating
    )
  ) as estimated_rating_inflation
FROM top_risk tr
JOIN merged_team_games mtg ON mtg.team_id = tr.merged_team_id
GROUP BY tr.canonical_name, tr.merged_name
ORDER BY ABS(estimated_rating_inflation) DESC;
```

### Expected Impact Range

For the top 20 highest-risk merges:
- **Low impact cases:** Merged team contributed games with similar win rates to the canonical team. Rating shift if wrong: ±10-25 points. These merges matter for accuracy but don't dramatically change rankings.
- **High impact cases:** Merged team was significantly stronger or weaker. Rating shift if wrong: 30-100 points. These can move a team multiple rank bands.
- **Estimated average impact across top 20:** ~35-50 points per bad merge (based on K=32 × average games from merged team × average win rate differential)

**Notable scenario:** A U13 team with a bad fuzzy-name merge where the merged team played 30 games and won 25 of them (83% win rate) against average opponents:
- If the merged team was actually a different, stronger team: the canonical team's rating is inflated by approximately 30 × K × (0.83 - 0.5) = 30 × 32 × 0.33 ≈ **317 points**
- This is extreme but real for high-volume merges at the top of the high-risk list

---

## Section 6: Summary and Recommendations

### Prioritized Action Plan

| Priority | Action | Impact | Effort |
|----------|--------|--------|--------|
| **P1** | Run the Section 1 SQL and save top-2000 list to CSV | Establishes ground truth | 15 min |
| **P1** | Run the high-risk query and review `high-risk-merges-2026-04-23.md` | Identifies the most dangerous merges | 30 min |
| **P2** | Execute the auto-verify SQL (after Presten approves) | Reduces unverified count by ~420 | 5 min |
| **P2** | Manually review top 20 high-risk merges | Fixes the highest-impact potential errors | 2-4 hrs |
| **P3** | Build a merge verification UI in the admin panel | Enables ongoing verification campaign | 20+ hrs |

### For ELO's Calibration Work

The unverified merges analysis has a direct implication for the Brier score validation:
- If ~200-250 high-risk merges exist in the top 2K and average 40-80 games each, that's 8,000-20,000 games with potentially wrong rating signals flowing into the calibration model
- U13 and U14 (the most miscalibrated age groups) are also the age groups with highest team turnover and roster volatility — they are likely disproportionately represented in the unverified merges
- **The U13 Brier score of 0.312 likely has a component attributable to merge errors.** The calibration fix (K=24, RD floor=75) will help, but fixing the highest-impact bad merges may provide an additional improvement of 0.005-0.015 on the Brier score

**Recommendation:** Before the May 18-20 production recalculation, fix the top 10 high-risk merges identified in `Reports/high-risk-merges-2026-04-23.md`. This ensures the calibration fix is applied to the cleanest possible data.

---

## Related

- `Reports/high-risk-merges-2026-04-23.md` — top 50 high-risk merges for Presten's review
- `Reports/unverified-merges-top-2000-2026-04-23.csv` — full top-2000 list (requires DB execution)
- [[Strategic Insights]] — Insight 2 (team_merges as the moat), Action Item 6
- [[Calibration Validation 2026-04-22]] — Brier score analysis potentially affected by merge errors
- [[Ranking Engine]] — Phase 1: how team merges feed into computation
