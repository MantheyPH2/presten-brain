---
title: High-Risk Merges — Results (Top 50)
tags: [team-merges, dedup, data-quality, high-risk, evo-draw, results]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: pending-db-execution
related_task: task-2026-04-23-execute-merge-audit-queries
related_spec: Reports/high-risk-merges-2026-04-23.md
---

# High-Risk Merges — Populated Results for Presten Review

> [!warning] DB Execution Required
> The SQL queries below are complete and ready to run in one psql session. Paste the output into the table in Section 3. This should take approximately 20–30 minutes. Deadline: **May 10, 2026** (before the May 18–20 recalculation).

---

## Section 1: How to Execute This Document

1. Open psql: `psql -U p -d youth_soccer`
2. Run Query A (top-50 high-risk merges) — paste results into Section 3
3. For any merge with `risk_reason = 'large_rating_gap'` AND `games_affected > 50`: run the impact query in Section 4
4. For confirmed wrong merges: run the un-merge SQL in Section 5
5. Update `review_verdict` in Section 3 for each record reviewed

---

## Section 2: Query A — Top-50 High-Risk Merges

Paste this directly into psql:

```sql
SELECT 
  tm.id AS merge_id,
  t1.team_name AS canonical_name,
  t1.age_group AS canonical_age_group,
  t2.team_name AS merged_name,
  t2.age_group AS merged_age_group,
  tm.merge_method,
  COUNT(g.id) AS games_affected,
  COALESCE(r1.rating, 0) AS canonical_rating,
  COALESCE(r2.rating, 0) AS merged_rating,
  ABS(COALESCE(r1.rating, 0) - COALESCE(r2.rating, 0)) AS rating_gap,
  CASE 
    WHEN tm.merge_method = 'fuzzy_name_match' AND t1.age_group != t2.age_group 
      THEN 'fuzzy_age_mismatch'
    WHEN tm.merge_method = 'coach_name' 
      THEN 'coach_name_date_gap'
    WHEN ABS(COALESCE(r1.rating, 0) - COALESCE(r2.rating, 0)) > 200 
      THEN 'large_rating_gap'
    WHEN tm.merge_method IS NULL OR LOWER(tm.merge_method) = 'unknown'
      THEN 'unknown_method'
    ELSE 'review_fuzzy_match'
  END AS risk_reason
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
    OR ABS(COALESCE(r1.rating, 0) - COALESCE(r2.rating, 0)) > 200
    OR tm.merge_method IS NULL
    OR LOWER(tm.merge_method) = 'unknown'
  )
GROUP BY 
  tm.id, t1.team_name, t1.age_group, t2.team_name, t2.age_group,
  tm.merge_method, r1.rating, r2.rating
ORDER BY games_affected DESC
LIMIT 50;
```

---

## Section 3: Review Table (Populate After Running Query A)

**Risk guide:** Anything with `games_affected > 50` AND `risk_reason IN ('fuzzy_age_mismatch', 'large_rating_gap')` is highest priority. Fix before May 18.

| # | Merge ID | Canonical Name | Age | Merged Name | Age | Method | Risk | Games | Rating Gap | Verdict | Notes |
|---|----------|---------------|-----|-------------|-----|--------|------|-------|------------|---------|-------|
| 1 | | | | | | | | | | ☐ Correct / ☐ Wrong / ☐ Unsure | |
| 2 | | | | | | | | | | ☐ Correct / ☐ Wrong / ☐ Unsure | |
| 3 | | | | | | | | | | ☐ Correct / ☐ Wrong / ☐ Unsure | |
| 4 | | | | | | | | | | ☐ Correct / ☐ Wrong / ☐ Unsure | |
| 5 | | | | | | | | | | ☐ Correct / ☐ Wrong / ☐ Unsure | |
| 6 | | | | | | | | | | ☐ Correct / ☐ Wrong / ☐ Unsure | |
| 7 | | | | | | | | | | ☐ Correct / ☐ Wrong / ☐ Unsure | |
| 8 | | | | | | | | | | ☐ Correct / ☐ Wrong / ☐ Unsure | |
| 9 | | | | | | | | | | ☐ Correct / ☐ Wrong / ☐ Unsure | |
| 10 | | | | | | | | | | ☐ Correct / ☐ Wrong / ☐ Unsure | |
| 11–50 | _run query and paste_ | | | | | | | | | | |

> **Decision rule:** If `rating_gap > 200` for a `fuzzy_name_match` merge and both teams played in the same recent season — this is almost certainly wrong. Mark as Wrong.

---

## Section 4: Rating Impact Query (Run for Any Top-10 "Wrong" Verdicts)

For each merge you identify as "Wrong," run this to estimate rating damage:

```sql
-- Replace [MERGED_TEAM_ID] and [CANONICAL_TEAM_ID] with actual IDs
-- Replace [CANONICAL_RATING] with the canonical team's current rating

SELECT 
  COUNT(*) AS games_from_merged_team,
  AVG(
    CASE 
      WHEN home_team_id = [MERGED_TEAM_ID] THEN home_score
      ELSE away_score 
    END
  ) AS avg_goals_scored,
  SUM(
    32.0 * (
      CASE 
        WHEN home_team_id = [MERGED_TEAM_ID] AND home_score > away_score THEN 1.0
        WHEN home_team_id = [MERGED_TEAM_ID] AND home_score = away_score THEN 0.5
        WHEN home_team_id = [MERGED_TEAM_ID] AND home_score < away_score THEN 0.0
        WHEN away_team_id = [MERGED_TEAM_ID] AND away_score > home_score THEN 1.0
        WHEN away_team_id = [MERGED_TEAM_ID] AND away_score = home_score THEN 0.5
        ELSE 0.0
      END
      -- Expected score (assume merged team was roughly average, rating 1200)
      - (1.0 / (1.0 + POWER(10.0, (
          CASE 
            WHEN home_team_id = [MERGED_TEAM_ID] 
              THEN (SELECT AVG(r.rating) FROM rankings r 
                    JOIN games g2 ON r.team_id = g2.away_team_id 
                    WHERE g2.home_team_id = [MERGED_TEAM_ID] AND g2.is_hidden = false) - 1200.0
            ELSE (SELECT AVG(r.rating) FROM rankings r 
                    JOIN games g2 ON r.team_id = g2.home_team_id 
                    WHERE g2.away_team_id = [MERGED_TEAM_ID] AND g2.is_hidden = false) - 1200.0
          END
        ) / 400.0)))
    )
  ) AS estimated_rating_inflation
FROM games
WHERE (home_team_id = [MERGED_TEAM_ID] OR away_team_id = [MERGED_TEAM_ID])
  AND is_hidden = false;
```

**Threshold:** If `estimated_rating_inflation > 75`, this is a HIGH-IMPACT bad merge. Fix before May 18.

---

## Section 5: Un-Merge SQL (One Template Per Wrong Verdict)

For each merge confirmed "Wrong," replace the placeholders and run in order:

```sql
-- STEP 1: Remove the bad merge record
-- Verify the merge_id first: SELECT * FROM team_merges WHERE id = [MERGE_ID];
DELETE FROM team_merges WHERE id = [MERGE_ID];

-- STEP 2: Restore games to the merged team (home games)
UPDATE games
SET home_team_id = [MERGED_TEAM_ID]
WHERE home_team_id = [CANONICAL_TEAM_ID]
  AND id IN (
    -- Games that were originally the merged team's (played before merge was created)
    SELECT g.id FROM games g
    WHERE g.home_team_id = [CANONICAL_TEAM_ID]
      AND g.match_date < (SELECT created_at FROM team_merges WHERE id = [MERGE_ID])
  );

-- STEP 2b: Restore games (away games)
UPDATE games
SET away_team_id = [MERGED_TEAM_ID]
WHERE away_team_id = [CANONICAL_TEAM_ID]
  AND id IN (
    SELECT g.id FROM games g
    WHERE g.away_team_id = [CANONICAL_TEAM_ID]
      AND g.match_date < (SELECT created_at FROM team_merges WHERE id = [MERGE_ID])
  );

-- STEP 3: Re-expose the merged team
UPDATE teams SET is_hidden = false WHERE team_id = [MERGED_TEAM_ID];

-- STEP 4: Note for rankings recompute
-- Run compute-rankings.js after all un-merges are complete.
-- Both [CANONICAL_TEAM_ID] and [MERGED_TEAM_ID] need recomputed ratings.
```

> [!warning] Run in dev/staging first. Confirm ratings look correct before committing to production. Do NOT run during an active pipeline execution.

---

## Section 6: Threshold Summary

After reviewing, populate this:

| Priority | Count | Games Affected | Action |
|----------|-------|---------------|--------|
| HIGH-IMPACT (rating shift >75pt, games >50) | | | Fix before May 18 |
| MEDIUM-IMPACT (rating shift 25–75pt) | | | Fix before June 1 |
| LOW-IMPACT (<25pt shift) | | | Fix when convenient |
| Total confirmed wrong merges | | | |
| Total confirmed correct merges | | | Auto-verify batch |

---

## Related

- `Reports/high-risk-merges-2026-04-23.md` — original spec with risk category definitions
- `Reports/unverified-merges-analysis-2026-04-23.md` — full analysis with all SQL
- `Rankings/unverified-merges-results-2026-04-23.md` — the auto-verify batch SQL and high-risk list
- [[Ranking Engine]] — Phase 1: how merges feed into ratings
- [[Calibration Validation 2026-04-22]] — note that merge errors may be contributing to U13/U14 Brier score issues
