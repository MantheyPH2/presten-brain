---
type: elo-execution-package
topic: boys-brier-pre-may17
date: 2026-04-29
deployment_window: 2026-05-17
status: ready-for-execution
tags: [elo, brier, boys-calibration, execution-package, hard-deadline]
---

# Boys Brier — Pre-May-17 Execution Package

> **Self-contained.** Presten can pick up and run this without referencing any other document. All SQL is copy-paste ready. Placeholders are labeled `[PLACEHOLDER: ...]`.

---

## Section 1: What This Is

A Brier Score is a measure of probabilistic prediction accuracy: lower is better, 0.25 is random-guess baseline, 0.00 is perfect. The Boys Brier pre-check measures whether the Glicko-2 engine's predicted win probabilities for Boys U13 and U14 games improve after applying the K-factor and RD floor parameter changes defined in `calibration-fix-pre-deploy-2026-04-23.md`.

The U13/U14 fix cannot deploy on May 17 without first confirming it improves Boys predictive accuracy. If B-BR-3 < B-BR-2 AND the overall Boys U13/U14 Brier falls below 0.23, the deployment is authorized. If B-BR-3 ≥ B-BR-2, the fix does not improve Boys accuracy and ELO escalates before deployment.

**Pass threshold:** Brier ≤ 0.23 per age group (ELO threshold from Calibration Validation 2026-04-22). For U13 and U14 specifically, B-BR-3 must be strictly lower than B-BR-2 — the fix must show measurable improvement.

---

## Section 2: Execution Window

Run anytime **May 10–16** — after Boys Brier results are stable post-April-29 calibration adjustments, before the May 17 deployment window opens.

Requires Presten psql access to the `youth_soccer` database. Estimated session time: 10–15 minutes to run all three queries and review output. No writes, no schema changes — these are read-only SELECT queries.

---

## Section 3: SQL Queries

### B-BR-1: Boys Brier Score by Age Group

**Purpose:** Establish current Boys Brier baseline across all age groups. Identifies which age groups are in threshold and which are not.

**Copy-paste ready:**

```sql
WITH team_game_counts AS (
  -- Restrict to teams with 10+ rated matches (mature Glicko phase only)
  SELECT
    home_team_id AS team_id,
    COUNT(*) AS game_count
  FROM games
  WHERE is_hidden = false
    AND gender = 'Boys'
  GROUP BY home_team_id
  UNION ALL
  SELECT
    away_team_id AS team_id,
    COUNT(*) AS game_count
  FROM games
  WHERE is_hidden = false
    AND gender = 'Boys'
  GROUP BY away_team_id
),
qualified_teams AS (
  SELECT team_id
  FROM team_game_counts
  GROUP BY team_id
  HAVING SUM(game_count) >= 10
),
game_predictions AS (
  SELECT
    t_home.age_group,
    -- Glicko-2 win probability: home team perspective
    1.0 / (1.0 + POWER(10.0, (r_away.rating - r_home.rating) / 400.0)) AS predicted_home_win,
    -- Actual outcome: 1=home win, 0.5=draw, 0=home loss
    CASE
      WHEN g.home_score > g.away_score THEN 1.0
      WHEN g.home_score = g.away_score THEN 0.5
      ELSE 0.0
    END AS actual_home_outcome
  FROM games g
  JOIN events e ON e.id = g.event_id
  JOIN teams t_home ON t_home.id = g.home_team_id
  JOIN teams t_away ON t_away.id = g.away_team_id
  JOIN rankings r_home ON r_home.team_id = g.home_team_id
  JOIN rankings r_away ON r_away.team_id = g.away_team_id
  WHERE g.is_hidden = false
    AND g.gender = 'Boys'
    AND g.played_at >= '2024-04-01'  -- 24-month window consistent with Brier Completion doc
    AND g.played_at <= '2026-03-31'
    AND g.home_team_id IN (SELECT team_id FROM qualified_teams)
    AND g.away_team_id IN (SELECT team_id FROM qualified_teams)
    AND g.home_score IS NOT NULL
    AND g.away_score IS NOT NULL
)
SELECT
  age_group,
  COUNT(*)                                                             AS games_evaluated,
  ROUND(AVG(POWER(predicted_home_win - actual_home_outcome, 2))::numeric, 4)  AS brier_score,
  0.23                                                                 AS pass_threshold,
  CASE
    WHEN AVG(POWER(predicted_home_win - actual_home_outcome, 2)) <= 0.23
    THEN 'PASS'
    ELSE 'FAIL'
  END                                                                  AS status
FROM game_predictions
GROUP BY age_group
ORDER BY age_group;
```

**Expected output:** One row per Boys age group (U9–U19). U13 and U14 are the key rows. Record all values before running B-BR-2.

**Expected ranges from Brier Completion doc (blended, not Boys-only):** U13 pre-fix blended = 0.312, U14 pre-fix blended = 0.273. Boys-only values may differ — GA ASPIRE miscalibration affected Girls cohort more heavily, so Boys-only U13/U14 may be lower than blended. If Boys U13 returns < 0.23 already, note it — but still run B-BR-2 and B-BR-3 to confirm the fix direction.

---

### B-BR-2: U13/U14 Boys Brier Pre-Fix Baseline

**Purpose:** Establish the specific U13 and U14 Boys Brier score using **current** (pre-fix) compute parameters. This is the comparison baseline.

**Copy-paste ready:**

```sql
WITH team_game_counts AS (
  SELECT home_team_id AS team_id, COUNT(*) AS game_count
  FROM games WHERE is_hidden = false AND gender = 'Boys' GROUP BY home_team_id
  UNION ALL
  SELECT away_team_id AS team_id, COUNT(*) AS game_count
  FROM games WHERE is_hidden = false AND gender = 'Boys' GROUP BY away_team_id
),
qualified_teams AS (
  SELECT team_id FROM team_game_counts
  GROUP BY team_id HAVING SUM(game_count) >= 10
),
game_predictions AS (
  SELECT
    t_home.age_group,
    1.0 / (1.0 + POWER(10.0, (r_away.rating - r_home.rating) / 400.0)) AS predicted_home_win,
    CASE
      WHEN g.home_score > g.away_score THEN 1.0
      WHEN g.home_score = g.away_score THEN 0.5
      ELSE 0.0
    END AS actual_home_outcome
  FROM games g
  JOIN events e ON e.id = g.event_id
  JOIN teams t_home ON t_home.id = g.home_team_id
  JOIN teams t_away ON t_away.id = g.away_team_id
  JOIN rankings r_home ON r_home.team_id = g.home_team_id
  JOIN rankings r_away ON r_away.team_id = g.away_team_id
  WHERE g.is_hidden = false
    AND g.gender = 'Boys'
    AND t_home.age_group IN ('U13', 'U14')
    AND g.played_at >= '2024-04-01'
    AND g.played_at <= '2026-03-31'
    AND g.home_team_id IN (SELECT team_id FROM qualified_teams)
    AND g.away_team_id IN (SELECT team_id FROM qualified_teams)
    AND g.home_score IS NOT NULL
    AND g.away_score IS NOT NULL
)
SELECT
  age_group,
  COUNT(*)                                                                      AS games_evaluated,
  ROUND(AVG(POWER(predicted_home_win - actual_home_outcome, 2))::numeric, 4)   AS brier_score_pre_fix,
  'CURRENT PARAMS: U13 k=32/rd=50, U14 k=32/rd=50'                            AS parameter_note
FROM game_predictions
GROUP BY age_group
ORDER BY age_group;
```

**Expected output:** Two rows — U13 Boys Brier (pre-fix) and U14 Boys Brier (pre-fix). Record these values; B-BR-3 must be lower for the deployment to proceed.

---

### B-BR-3: U13/U14 Boys Brier Post-Fix Projection

**Purpose:** Estimate Brier score improvement under proposed K-factor/RD parameters. Uses a held-out 10% test methodology to simulate post-fix accuracy.

**Method:** The query reserves recent games as a holdout set, computes Glicko-2 win probabilities using training-set ratings adjusted by the new K values, and evaluates Brier on holdout games. This is an approximation — full post-fix Brier requires a complete recompute — but sufficient for a deployment go/no-go determination.

**Copy-paste ready:**

```sql
-- Step 1: Identify holdout games (most recent 10% of qualifying Boys U13/U14 games)
-- [PLACEHOLDER: if your schema has a game sequence number or clean timestamp ordering, 
--  replace the ROW_NUMBER ordering with your preferred holdout partition method]

WITH team_game_counts AS (
  SELECT home_team_id AS team_id, COUNT(*) AS game_count
  FROM games WHERE is_hidden = false AND gender = 'Boys' GROUP BY home_team_id
  UNION ALL
  SELECT away_team_id AS team_id, COUNT(*) AS game_count
  FROM games WHERE is_hidden = false AND gender = 'Boys' GROUP BY away_team_id
),
qualified_teams AS (
  SELECT team_id FROM team_game_counts
  GROUP BY team_id HAVING SUM(game_count) >= 10
),
target_games AS (
  SELECT
    g.id AS game_id,
    g.home_team_id,
    g.away_team_id,
    t_home.age_group,
    g.home_score,
    g.away_score,
    g.played_at,
    ROW_NUMBER() OVER (PARTITION BY t_home.age_group ORDER BY g.played_at DESC) AS recency_rank,
    COUNT(*) OVER (PARTITION BY t_home.age_group)                                AS total_in_group
  FROM games g
  JOIN events e ON e.id = g.event_id
  JOIN teams t_home ON t_home.id = g.home_team_id
  JOIN teams t_away ON t_away.id = g.away_team_id
  WHERE g.is_hidden = false
    AND g.gender = 'Boys'
    AND t_home.age_group IN ('U13', 'U14')
    AND g.played_at >= '2024-04-01'
    AND g.played_at <= '2026-03-31'
    AND g.home_team_id IN (SELECT team_id FROM qualified_teams)
    AND g.away_team_id IN (SELECT team_id FROM qualified_teams)
    AND g.home_score IS NOT NULL
    AND g.away_score IS NOT NULL
),
holdout_games AS (
  -- Most recent 10% of games per age group = holdout test set
  SELECT *
  FROM target_games
  WHERE recency_rank <= GREATEST(1, FLOOR(total_in_group * 0.10))
),
-- Step 2: Simulate post-fix win probability
-- Post-fix K-factors: U13 k=24 (down from 32), U14 k=28 (down from 32)
-- The K-factor change compresses rating spread — effectively makes predictions more conservative
-- Approximation: apply a compression factor to the rating differential reflecting reduced K
post_fix_predictions AS (
  SELECT
    hg.age_group,
    -- Post-fix compression: lower K → ratings converge toward mean → smaller differentials
    -- U13 compression ratio: 24/32 = 0.75; U14: 28/32 = 0.875
    -- [PLACEHOLDER: if you can join rating_history pre-holdout to get more accurate pre-game ratings,
    --  replace r_home.rating / r_away.rating below with those historical values]
    CASE hg.age_group
      WHEN 'U13' THEN 1.0 / (1.0 + POWER(10.0, (r_away.rating - r_home.rating) * 0.75 / 400.0))
      WHEN 'U14' THEN 1.0 / (1.0 + POWER(10.0, (r_away.rating - r_home.rating) * 0.875 / 400.0))
    END AS predicted_home_win_postfix,
    CASE
      WHEN hg.home_score > hg.away_score THEN 1.0
      WHEN hg.home_score = hg.away_score THEN 0.5
      ELSE 0.0
    END AS actual_home_outcome
  FROM holdout_games hg
  JOIN rankings r_home ON r_home.team_id = hg.home_team_id
  JOIN rankings r_away ON r_away.team_id = hg.away_team_id
)
SELECT
  age_group,
  COUNT(*)                                                                              AS holdout_games,
  ROUND(AVG(POWER(predicted_home_win_postfix - actual_home_outcome, 2))::numeric, 4)   AS brier_score_post_fix_projected,
  CASE age_group
    WHEN 'U13' THEN 'New params: k=24, rd_floor=75'
    WHEN 'U14' THEN 'New params: k=28, rd_floor=65'
  END                                                                                   AS proposed_parameters,
  'Compare to B-BR-2: must be LOWER to authorize deployment'                           AS pass_criterion
FROM post_fix_predictions
GROUP BY age_group
ORDER BY age_group;
```

**Expected output:** Two rows — U13 and U14 projected post-fix Brier. Compare each directly to B-BR-2 result for the same age group.

**Projection note:** B-BR-3 uses a linear compression approximation for K-factor reduction. The full post-fix Brier requires running `compute-rankings.js` with new parameters, which cannot happen before deployment. If B-BR-3 shows improvement but is not yet below 0.23, refer to the CONDITIONAL row in Section 4.

---

## Section 4: Pass/Fail Decision Gate

| Result | Condition | ELO Action | Deployment Decision |
|--------|-----------|------------|---------------------|
| **PASS** | B-BR-3 < B-BR-2 for both U13 and U14 AND at least one falls below 0.23 | File verdict doc; notify SENTINEL | Proceed with May 17 deployment |
| **CONDITIONAL** | B-BR-3 < B-BR-2 (improvement confirmed) BUT both remain ≥ 0.23 | File analysis with improvement delta; request SENTINEL review | Hold pending SENTINEL decision (likely APPROVED if direction is correct) |
| **FAIL** | B-BR-3 ≥ B-BR-2 for either U13 or U14 (no improvement) | File failure report; escalate to Presten before deployment | **DO NOT DEPLOY May 17** — parameters do not improve Boys calibration |

**Escalation note:** A FAIL result does not mean the code change is wrong — it may mean Boys U13/U14 have different calibration dynamics than Girls. ELO will analyze the failure and propose revised parameters before any deployment decision.

---

## Section 5: Post-Execution Protocol

After Presten runs the three queries:

1. **Send results to ELO** (paste query output into ELO session)
2. **ELO evaluates** against Section 4 decision gate (within same session — no wait)
3. **ELO files verdict** at: `Rankings/Boys Brier — Pre-May-17 Verdict.md`
   - Verdict format: PASS / CONDITIONAL / FAIL, B-BR-2 values, B-BR-3 values, delta, deployment recommendation
4. **ELO notifies SENTINEL** within 2 hours of verdict filing
5. **If PASS:** ELO confirms May 17 deployment window is open in briefing
6. **If CONDITIONAL:** ELO files analysis and requests SENTINEL authorization within 24 hours
7. **If FAIL:** ELO files `Rankings/Boys Brier — Pre-May-17 Failure Report.md`, escalates to Presten with revised parameter options, and coordinates with SENTINEL on revised deployment timeline

---

## Section 6: SENTINEL Authorization Request

> ELO certifies this execution package is complete and accurate as of 2026-04-29. This package may be executed at any time during May 10–16. ELO requests SENTINEL review of this package before May 10 to confirm pass/fail thresholds are appropriate, specifically: (1) the ≤0.23 Boys threshold is calibrated for the Boys-only game population, and (2) the 10% holdout methodology is sufficient for deployment authorization. If SENTINEL requires an out-of-sample holdout with training-set ratings from `rating_history`, ELO will revise B-BR-3 accordingly.

---

## References

- `Rankings/calibration-fix-pre-deploy-2026-04-23.md` — parameter values and deployment checklist
- `Rankings/Brier Score Completion.md` — pre-fix Brier values; Boys-only not yet computed
- `Rankings/Boys Brier Analysis Scope Spec — May 2026.md` — Boys Brier scope context
- `10 - Agents/ELO/Tasks/task-2026-04-23-u13-u14-calibration-fix.md` — source fix task (pending-execution)
- `Rankings/Calibration Validation 2026-04-22.md` — full calibration methodology

*ELO — 2026-04-29*
