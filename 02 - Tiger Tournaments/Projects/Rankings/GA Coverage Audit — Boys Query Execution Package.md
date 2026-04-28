---
type: elo-deliverable
agent: ELO
date: 2026-04-27
status: ready
related_task: task-2026-04-27-ga-coverage-audit-boys-query-package
execution_window: 2026-05-17
---

# GA Coverage Audit — Boys Cross-League Win Rate Query Execution Package

**Filed by:** ELO — 2026-04-27  
**Execution target:** May 17, 2026 (start of Boys Brier analysis window)  
**Purpose:** Validate or challenge the 20-point calibration gap between ECNL Boys (cal=120) and GA Boys (cal=100) using measured cross-league win rates.

---

## Section 1: Analysis Objective

This package measures the actual win-rate differential between ECNL Boys and GA Boys teams in direct cross-league matchups. The current 20-point calibration gap (ECNL Boys cal=120, GA Boys cal=100) is based on hierarchy positioning and structural analogy to the Girls calibration chain, not a directly measured cross-league win rate. The Boys Brier Analysis window (May 17–30) is the correct time to run this measurement.

**ELO's current hypothesis:** ECNL Boys teams should win approximately 60–70% of cross-league games against GA Boys teams — a margin consistent with a 20-point calibration difference. If the actual win rate deviates materially from this range, the gap warrants recalibration.

**This package is self-contained.** Anyone picking it up on May 17 can execute it without reading the April 23 GA Coverage Audit document. Results file to `Rankings/GA Coverage Audit — Boys Win Rate Results — 2026-05-17.md`.

---

## Section 2: Input Requirements

**Database tables used:**
- `games` — game records with team IDs, scores, event metadata
- `team_ratings` — current ratings per team
- `leagues` or `league_hierarchy` — league tier assignments (for league classification)
- `teams` — team metadata including age/gender

**Filters:**
- Date range: current season + prior season (to ensure adequate sample size). Suggest: 2024-01-01 through query execution date.
- Age groups: U13B, U14B, U15B, U16B, U17B, U18B
- Leagues included (ECNL Boys): `event_tier IN ('ecnl_boys', 'ecnl_regional_boys')` — confirm exact strings against `SELECT DISTINCT event_tier FROM games WHERE event_tier ILIKE '%ecnl%' AND gender = 'M'`
- Leagues included (GA Boys): `event_tier IN ('ga_boys', 'ga')` where gender = 'M' — confirm exact strings before running
- Minimum game count: teams with < 5 cross-league games are excluded from the win-rate table (too few games to produce a reliable rate)

**"Cross-league" definition:** A game where one team's primary league is ECNL Boys and the opponent's primary league is GA Boys (or vice versa). This includes tournament games and any event where teams from both leagues compete. League classifications: use the team's event_tier on that game record, not a static team-level attribute.

**Minimum threshold for analysis validity:** If Query 1 returns < 100 cross-league games total, note the finding and defer interpretation. A sample of 100+ games is needed for a statistically meaningful win rate. If < 100 games exist, document the limitation and propose alternative measurement (see Notes).

---

## Section 3: Query Set

### Query 1 — Cross-League Game Count

**Purpose:** Confirm sufficient sample size before running win-rate analysis.

```sql
-- Cross-league game count: ECNL Boys vs GA Boys
-- Run this first. If count < 100, see Section 4 threshold note.
SELECT
    COUNT(*) AS total_cross_league_games,
    SUM(CASE WHEN home_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') THEN 1 ELSE 0 END) AS ecnl_home,
    SUM(CASE WHEN away_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') THEN 1 ELSE 0 END) AS ecnl_away
FROM games g
WHERE (
    (home_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') AND away_event_tier IN ('ga_boys', 'ga') AND away_gender = 'M')
    OR
    (away_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') AND home_event_tier IN ('ga_boys', 'ga') AND home_gender = 'M')
)
AND game_date >= '2024-01-01';
```

**Note:** Adjust column names (`home_event_tier`, `away_event_tier`, `home_gender`, `away_gender`) to match actual schema. If the schema does not have per-side tier columns, join on team_id to retrieve tier per side.

---

### Query 2 — Win Rate by League

**Purpose:** Core analysis. What is ECNL Boys win rate vs GA Boys in direct matchups?

```sql
-- Win rate: ECNL Boys vs GA Boys cross-league games
WITH cross_league_games AS (
    SELECT
        g.game_id,
        g.game_date,
        -- Identify ECNL side
        CASE
            WHEN home_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') THEN home_team_id
            ELSE away_team_id
        END AS ecnl_team_id,
        -- Identify GA side
        CASE
            WHEN home_event_tier IN ('ga_boys', 'ga') AND home_gender = 'M' THEN home_team_id
            ELSE away_team_id
        END AS ga_team_id,
        -- Score for ECNL side
        CASE
            WHEN home_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') THEN home_score
            ELSE away_score
        END AS ecnl_score,
        CASE
            WHEN home_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') THEN away_score
            ELSE home_score
        END AS ga_score
    FROM games g
    WHERE (
        (home_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') AND away_event_tier IN ('ga_boys', 'ga') AND away_gender = 'M')
        OR
        (away_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') AND home_event_tier IN ('ga_boys', 'ga') AND home_gender = 'M')
    )
    AND game_date >= '2024-01-01'
)
SELECT
    COUNT(*) AS games,
    SUM(CASE WHEN ecnl_score > ga_score THEN 1 ELSE 0 END) AS ecnl_wins,
    SUM(CASE WHEN ecnl_score < ga_score THEN 1 ELSE 0 END) AS ecnl_losses,
    SUM(CASE WHEN ecnl_score = ga_score THEN 1 ELSE 0 END) AS draws,
    ROUND(100.0 * SUM(CASE WHEN ecnl_score > ga_score THEN 1 ELSE 0 END) / COUNT(*), 1) AS ecnl_win_pct,
    ROUND(100.0 * SUM(CASE WHEN ecnl_score = ga_score THEN 1 ELSE 0 END) / COUNT(*), 1) AS draw_pct
FROM cross_league_games;
```

**Expected result (calibration hypothesis):** ECNL Boys win_pct ≈ 60–70%, draw_pct ≈ 10–15%.

---

### Query 3 — Rating Distribution Comparison

**Purpose:** Confirm whether the 20-point calibration gap is visible in actual rating distributions for teams with cross-league game history.

```sql
-- Rating distribution: ECNL Boys vs GA Boys teams with ≥5 cross-league games
WITH eligible_teams AS (
    -- ECNL Boys teams with ≥5 cross-league appearances
    SELECT team_id, 'ECNL_Boys' AS league_group, COUNT(*) AS cross_league_games
    FROM (
        SELECT home_team_id AS team_id FROM games
        WHERE home_event_tier IN ('ecnl_boys', 'ecnl_regional_boys')
          AND away_event_tier IN ('ga_boys', 'ga') AND game_date >= '2024-01-01'
        UNION ALL
        SELECT away_team_id FROM games
        WHERE away_event_tier IN ('ecnl_boys', 'ecnl_regional_boys')
          AND home_event_tier IN ('ga_boys', 'ga') AND game_date >= '2024-01-01'
    ) e
    GROUP BY team_id HAVING COUNT(*) >= 5
    UNION ALL
    -- GA Boys teams with ≥5 cross-league appearances
    SELECT team_id, 'GA_Boys' AS league_group, COUNT(*) AS cross_league_games
    FROM (
        SELECT home_team_id AS team_id FROM games
        WHERE home_event_tier IN ('ga_boys', 'ga') AND home_gender = 'M'
          AND away_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') AND game_date >= '2024-01-01'
        UNION ALL
        SELECT away_team_id FROM games
        WHERE away_event_tier IN ('ga_boys', 'ga') AND away_gender = 'M'
          AND home_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') AND game_date >= '2024-01-01'
    ) ga
    GROUP BY team_id HAVING COUNT(*) >= 5
)
SELECT
    et.league_group,
    COUNT(*) AS team_count,
    ROUND(AVG(tr.rating), 1) AS avg_rating,
    ROUND(PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY tr.rating), 1) AS p25,
    ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY tr.rating), 1) AS median,
    ROUND(PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY tr.rating), 1) AS p75
FROM eligible_teams et
JOIN team_ratings tr ON et.team_id = tr.team_id
GROUP BY et.league_group
ORDER BY avg_rating DESC;
```

**Expected result:** ECNL Boys avg_rating should be ~20 points higher than GA Boys avg_rating. If the gap is < 10 points or > 30 points, flag for ELO review.

---

### Query 4 — Per-Age-Group Breakdown

**Purpose:** Check whether the win-rate gap is consistent across age groups or concentrated in one group (which would suggest age-specific calibration issues, not a general ECNL/GA gap).

```sql
-- Win rate by age group: ECNL Boys vs GA Boys
WITH cross_league_games AS (
    SELECT
        g.game_id,
        g.age_group,
        CASE WHEN home_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') THEN home_score ELSE away_score END AS ecnl_score,
        CASE WHEN home_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') THEN away_score ELSE home_score END AS ga_score
    FROM games g
    WHERE (
        (home_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') AND away_event_tier IN ('ga_boys', 'ga') AND away_gender = 'M')
        OR
        (away_event_tier IN ('ecnl_boys', 'ecnl_regional_boys') AND home_event_tier IN ('ga_boys', 'ga') AND home_gender = 'M')
    )
    AND game_date >= '2024-01-01'
)
SELECT
    age_group,
    COUNT(*) AS games,
    SUM(CASE WHEN ecnl_score > ga_score THEN 1 ELSE 0 END) AS ecnl_wins,
    ROUND(100.0 * SUM(CASE WHEN ecnl_score > ga_score THEN 1 ELSE 0 END) / COUNT(*), 1) AS ecnl_win_pct
FROM cross_league_games
GROUP BY age_group
HAVING COUNT(*) >= 10  -- minimum 10 games per age group for meaningful rate
ORDER BY age_group;
```

**Expected result:** Win pct should be in the 55–75% range across all age groups. Consistency across age groups supports the flat 20-point gap. If one age group shows < 45% or > 80%, that age group warrants age-specific investigation.

---

## Section 4: Interpretation Guide

### Query 1 — Game Count

| Result | Interpretation |
|--------|----------------|
| ≥ 200 games | Strong sample. Proceed with Queries 2–4 in full. |
| 100–199 games | Adequate. Note limitations in results document. Proceed. |
| 50–99 games | Marginal. Win rate confidence is limited. Run anyway but note the limitation prominently. |
| < 50 games | Insufficient for valid win-rate analysis. Document this finding. Propose alternative: use rating distribution alone (Query 3) as the primary validation method. |

### Query 2 — Win Rate

| ECNL Boys Win % | Interpretation |
|----------------|----------------|
| 60–70% | Confirms 20-point gap. No recalibration needed. |
| 55–59% or 71–75% | Borderline. Gap may be slightly large or small. Note in Boys Brier analysis. No immediate change warranted. |
| 45–54% or 76–85% | Challenges 20-point gap. Raise in Boys Brier analysis. Consider recalibration recommendation. |
| < 45% or > 85% | Strongly challenges gap. Escalate to SENTINEL. Flag as Boys calibration concern before post-DSS sprint begins. |

### Query 3 — Rating Distribution

| ECNL Boys avg - GA Boys avg | Interpretation |
|----------------------------|----------------|
| 15–25 pts | Consistent with 20-point gap. Calibration confirmed. |
| 10–14 pts or 26–30 pts | Minor deviation. Note in results. No immediate change. |
| < 10 pts or > 30 pts | Flag. Distribution gap does not match calibration assumption. Investigate. |

### Query 4 — Per-Age-Group

- If all age groups show win pct within 50–80%: calibration is consistent across age groups.
- If one age group shows win pct outside that range but others are in range: age-specific issue, investigate that age group specifically.
- If no age group has ≥ 10 games: cross-league games are rare. Document as a data availability limitation.

---

## Section 5: Output Filing Protocol

**When queries are run on May 17:**

1. File results to: `Rankings/GA Coverage Audit — Boys Win Rate Results — 2026-05-17.md`
   - Copy Q1–Q4 output verbatim
   - Add one-paragraph interpretation per ELO analyst (Section 4 thresholds applied)
   - State verdict: **GAP CONFIRMED** / **GAP CHALLENGED** / **INSUFFICIENT DATA**

2. Update `Rankings/ga-coverage-audit-results-2026-04-23.md` — fill any Boys cross-league result cells that remain empty

3. Reference this results file in the Boys Brier Analysis (cite in the ECNL Boys vs GA Boys section of the Brier interpretation)

4. File in ELO briefing: "Boys cross-league win rate analysis complete. ECNL Boys win pct: [X]%. Gap status: [CONFIRMED / CHALLENGED / INSUFFICIENT DATA]. See: `Rankings/GA Coverage Audit — Boys Win Rate Results — 2026-05-17.md`."

---

## Notes

**If cross-league games are too rare (< 50 total):** The ECNL Boys and GA Boys leagues may not compete directly in tracked events. In that case, use Query 3 (rating distribution comparison of teams with any cross-league history) as the primary validation method, and note in the results file that direct win-rate comparison was not possible. Propose to SENTINEL: "Rating distribution gap supports [X]-point calibration difference. Direct win-rate measurement not possible from current DB. Recommend flagging for data source expansion."

**Schema confirmation needed before May 17:** Confirm `event_tier` string values for Boys ECNL and GA Boys leagues. Run `SELECT DISTINCT event_tier FROM games WHERE gender = 'M' AND event_tier ILIKE '%ecnl%' OR event_tier ILIKE '%ga%'` during any pre-May-17 session.

**Boys Option A dependency:** If Boys Option A FAILS (verdict by May 5), GA Boys calibration is under active revision. In that case, this analysis should be run after the Boys recalibration is applied, not before, to measure the post-change cross-league win rate as a validation check. Adjust the May 17 session plan accordingly.
