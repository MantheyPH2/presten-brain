---
title: Unverified Merges — Execution Results (Top 2,000)
tags: [team-merges, dedup, data-quality, verification, evo-draw, results]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: pending-db-execution
related_task: task-2026-04-23-execute-merge-audit-queries
related_spec: Reports/unverified-merges-analysis-2026-04-23.md
---

# Unverified Merges — Execution Results

> [!warning] DB Execution Required
> All SQL in this document is ready to run. Complete Section 2 (CSV export) and Section 4 (auto-verify count) in a single 30-minute psql session. Deadline: **May 10, 2026** — before the May 18–20 recalculation window.

---

## Section 1: Execution Order

1. Run **Query 1** → export CSV file (top-2000 by games)
2. Run **Query 2** → get merge method distribution (fill Section 3 table)
3. Run **Query 3** → get auto-verify candidate count (fill Section 4)
4. Review `Rankings/high-risk-merges-results-2026-04-23.md` → fill Section 5 after completing that doc
5. Once Presten approves, run the **staged auto-verify UPDATE** in Section 6

---

## Section 2: Query 1 — Export Top 2,000 Unverified Merges (CSV)

```sql
-- Export to CSV first, then analyze
\copy (
  SELECT 
    tm.id AS merge_id,
    tm.canonical_team_id,
    tm.merged_team_id,
    tm.merge_method,
    tm.confidence_score,
    t1.team_name AS canonical_name,
    t1.age_group AS canonical_age_group,
    t1.gender AS canonical_gender,
    t2.team_name AS merged_name,
    t2.age_group AS merged_age_group,
    t2.gender AS merged_gender,
    COUNT(g.id) AS total_games_affected,
    MIN(g.match_date) AS earliest_game,
    MAX(g.match_date) AS latest_game,
    COUNT(DISTINCT g.event_name) AS distinct_events
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
  LIMIT 2000
) TO '/tmp/unverified-merges-top-2000-2026-04-23.csv' WITH CSV HEADER;
```

---

## Section 3: Merge Method Distribution (Fill After Query 2)

Run this after the CSV export:

```sql
WITH top_merges AS (
  SELECT 
    tm.id AS merge_id,
    tm.merge_method,
    COUNT(g.id) AS total_games_affected
  FROM team_merges tm
  JOIN games g ON (g.home_team_id = tm.merged_team_id OR g.away_team_id = tm.merged_team_id)
  WHERE tm.verified = false
    AND g.is_hidden = false
  GROUP BY tm.id, tm.merge_method
  ORDER BY total_games_affected DESC
  LIMIT 2000
)
SELECT 
  COALESCE(merge_method, 'unknown/null') AS merge_method,
  COUNT(*) AS merge_count,
  SUM(total_games_affected) AS total_games,
  ROUND(AVG(total_games_affected), 1) AS avg_games_per_merge
FROM top_merges
GROUP BY COALESCE(merge_method, 'unknown/null')
ORDER BY merge_count DESC;
```

**Results (populate after running):**

| Merge Method | Count in Top 2K | Total Games | Avg Games/Merge | Est. Error Rate | Action |
|--------------|-----------------|-------------|-----------------|-----------------|--------|
| exact_name_match | | | | Low (2–5%) | Auto-verify candidates |
| fuzzy_name_match | | | | Medium (10–20%) | Manual review if age group mismatch |
| cross_source_id | | | | Low (3–7%) | Spot-check sample |
| coach_name | | | | Medium-Low (5–10%) | Check date overlap |
| manual | | | | Very Low (<2%) | Accept |
| unknown/null | | | | Unknown | Flag all for investigation |
| **TOTAL** | | | | | |

---

## Section 4: Auto-Verify Candidate Count (Fill After Query 3)

These are the ~420 merges that can be safely auto-verified without manual review.

```sql
WITH date_ranges AS (
  SELECT 
    team_id,
    MIN(match_date) AS first_game,
    MAX(match_date) AS last_game
  FROM (
    SELECT home_team_id AS team_id, match_date FROM games WHERE is_hidden = false
    UNION ALL
    SELECT away_team_id AS team_id, match_date FROM games WHERE is_hidden = false
  ) all_games
  GROUP BY team_id
)
SELECT 
  COUNT(*) AS auto_verify_candidates,
  SUM(game_count) AS total_games_covered
FROM (
  SELECT 
    tm.id AS merge_id,
    COUNT(g.id) AS game_count
  FROM team_merges tm
  JOIN teams t1 ON tm.canonical_team_id = t1.team_id
  JOIN teams t2 ON tm.merged_team_id = t2.team_id
  JOIN games g ON (g.home_team_id = tm.merged_team_id OR g.away_team_id = tm.merged_team_id)
  JOIN date_ranges dr1 ON dr1.team_id = tm.canonical_team_id
  JOIN date_ranges dr2 ON dr2.team_id = tm.merged_team_id
  WHERE tm.verified = false
    AND g.is_hidden = false
    AND tm.merge_method = 'exact_name_match'
    AND t1.age_group = t2.age_group
    -- Date overlap: merged team has games that overlap with canonical team's era
    AND dr2.last_game >= dr1.first_game - INTERVAL '180 days'
    AND (
      t1.source ILIKE '%gotsport%' 
      OR t2.source ILIKE '%gotsport%'
    )
  GROUP BY tm.id
  ORDER BY game_count DESC
  LIMIT 2000
) candidates;
```

**Result:** Auto-verify candidates found: _____ (expected ~420)  
**Total games that would be confirmed:** _____ (expected ~8,400)

---

## Section 5: High-Risk Merge Summary (Cross-Reference with high-risk-merges-results)

After completing `Rankings/high-risk-merges-results-2026-04-23.md`, fill in here:

| Risk Category | Count Identified | Total Games Affected | Impact Level |
|---------------|-----------------|---------------------|--------------|
| fuzzy_age_mismatch | | | |
| large_rating_gap | | | |
| unknown_method | | | |
| coach_name_date_gap | | | |
| review_fuzzy_match | | | |
| **Total high-risk** | | | |
| Confirmed wrong merges | | | Fix before May 18 |
| Confirmed correct merges | | | Add to auto-verify batch |

---

## Section 6: Staged Auto-Verify SQL (Run After Presten Approves)

```sql
-- STAGED — DO NOT EXECUTE WITHOUT APPROVAL
-- Replace [MERGE_IDS] with the comma-separated list of IDs from Query 3 output

-- Preview count first:
SELECT COUNT(*) FROM team_merges 
WHERE id IN ([MERGE_IDS_FROM_AUTO_VERIFY_QUERY]);

-- Then execute:
UPDATE team_merges
SET 
  verified = true,
  verified_by = 'ELO_AUTO_2026-04-23',
  verified_at = NOW()
WHERE id IN ([MERGE_IDS_FROM_AUTO_VERIFY_QUERY]);

-- Confirm:
SELECT COUNT(*) FROM team_merges WHERE verified_by = 'ELO_AUTO_2026-04-23';
```

**Status:** ☐ Pending approval | ☐ Approved | ☐ Executed (date: _____)

---

## Section 7: Final Summary

Complete after executing:

- Total unverified merges identified: _____ / 9,136 total
- Auto-verified via batch: _____
- Manually reviewed (top 50 high-risk): _____
- Confirmed wrong merges fixed: _____
- Games freed from bad merges: ~_____
- Estimated Brier score improvement for U13 from merge fixes: ~0.005–0.015 (per analysis)
- Remaining unverified after this campaign: _____

---

## Related

- `Reports/unverified-merges-analysis-2026-04-23.md` — full analysis with methodology
- `Rankings/high-risk-merges-results-2026-04-23.md` — top-50 high-risk review table
- [[Strategic Insights]] — Insight 2 (team_merges as the moat), Action Item 6
- [[Calibration Validation 2026-04-22]] — Brier scores potentially affected by merge errors
- [[Ranking Engine]] — Phase 1: team merges in ratings computation
