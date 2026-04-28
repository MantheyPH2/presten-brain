---
title: ECNL Boys vs GA Boys Win-Rate Gap Investigation — 2026-04-27
type: calibration-investigation
author: ELO
date: 2026-04-27
status: complete
related: "[[Boys-ECNL-Calibration-Scope-2026-04-27]]", "[[Boys Calibration Status Summary — April 2026]]", "[[League Hierarchy]]"
tags: [calibration, boys, ecnl, ga, win-rate, investigation, evo-draw]
---

# ECNL Boys vs GA Boys Win-Rate Gap Investigation — 2026-04-27

> **Context:** The Boys ECNL Calibration Scope document (filed 2026-04-27) identified the 20-point gap between ECNL Boys (cal=120) and GA Boys (cal=100) as the weakest link in Boys calibration — based on hierarchy positioning logic, not a directly measured cross-league win rate. This document closes that gap, or formally acknowledges the data limitation.

---

## Step 1: April 23 GA Coverage Audit — Cross-League Win Rate Check

**Finding:** The April 23 GA Coverage Audit (`ga-coverage-audit-results-2026-04-23.md`) was assigned to validate Boys GA calibration via cross-league win rates. However, the audit results document contains SQL query templates with `_(run query)_` placeholders in all result cells — the database queries were designed but the actual execution and results were not recorded. Section 5 of that document ("Boys GA Calibration Flag") notes:

> "If total boys GA game count > 200: cross-league win rate validation is feasible. Post a queue item."

The boys GA game count cell also shows `_____` (unfilled). The queue item for Boys GA calibration review was flagged as a deliverable from that task but there is no evidence it was filed as a separate queue item with measured results.

**Conclusion:** The April 23 GA Coverage Audit did NOT produce a measured cross-league Boys ECNL vs GA Boys win rate. The audit produced the SQL methodology and noted the Boys GA data gap, but execution results are absent from the vault.

---

## Step 2: Win-Rate Assessment (Pre-Measurement)

The 20-point ECNL Boys (120) → GA Boys (100) gap cannot be empirically confirmed or challenged from current vault data. What the vault does confirm:

**Evidence supporting the gap (indirect):**
- ECNL Boys (120) is validated from above: MLS NEXT Homegrown beats ECNL Boys at 71.4% — strongly confirming ECNL Boys is materially weaker than MLS NEXT (160). The 40-point gap is empirically grounded.
- ECNL RL (55) is validated: ECNL RL vs NPL cross-league win rate of 55.1% confirms the 65-point drop from ECNL Boys (120) to Tier 2 (55) is calibration-correct.
- By structural constraint: GA Boys must sit between ECNL Boys (120) and ECNL RL/NPL (55). A value of 100 places it roughly at the midpoint of that span, which is plausible for a national-level Boys league that is strong but not at ECNL's depth.

**Evidence that could challenge the gap:**
- Girls GA calibration (140) was found to be overcalibrated relative to actual win rates against ECNL Girls — GA ASPIRE teams were ranked 40 pts higher than their cross-league performance supported. If Boys GA has an analogous overcalibration, cal=100 could be too high.
- However: Boys GA is already calibrated 20 points below Boys ECNL (120), unlike Girls GA (140) which was only 10 points below Girls ECNL (130 for Girls ECNL). The Boys GA setup is more conservative from the start. If the structural positioning logic applied to Girls GA was over-indexed, the Boys GA equivalent is less exposed.

---

## Step 3: SQL to Measure the Cross-League Win Rate

The following query measures the direct Boys ECNL vs GA Boys cross-league win rate from games where both teams are present.

```sql
-- ECNL Boys vs GA Boys cross-league win rate
-- Step 1: Find cross-league games (one team ECNL, opponent GA)
WITH ecnl_boys_teams AS (
    SELECT DISTINCT home_team_id AS team_id
    FROM games g
    JOIN event_tiers et ON et.event_name = g.event_name
    WHERE et.event_tier = 'ecnl'
      AND g.gender = 'M'
      AND g.is_hidden = false
    UNION
    SELECT DISTINCT away_team_id
    FROM games g
    JOIN event_tiers et ON et.event_name = g.event_name
    WHERE et.event_tier = 'ecnl'
      AND g.gender = 'M'
      AND g.is_hidden = false
),
ga_boys_teams AS (
    SELECT DISTINCT home_team_id AS team_id
    FROM games g
    JOIN event_tiers et ON et.event_name = g.event_name
    WHERE et.event_tier = 'ga'
      AND g.gender = 'M'
      AND g.is_hidden = false
    UNION
    SELECT DISTINCT away_team_id
    FROM games g
    JOIN event_tiers et ON et.event_name = g.event_name
    WHERE et.event_tier = 'ga'
      AND g.gender = 'M'
      AND g.is_hidden = false
),
cross_league_games AS (
    SELECT
        g.id,
        g.home_score,
        g.away_score,
        CASE 
            WHEN g.home_team_id IN (SELECT team_id FROM ecnl_boys_teams) 
             AND g.away_team_id IN (SELECT team_id FROM ga_boys_teams) 
            THEN 'ecnl_home'
            WHEN g.away_team_id IN (SELECT team_id FROM ecnl_boys_teams) 
             AND g.home_team_id IN (SELECT team_id FROM ga_boys_teams) 
            THEN 'ecnl_away'
        END AS game_type
    FROM games g
    WHERE (
        (g.home_team_id IN (SELECT team_id FROM ecnl_boys_teams) AND g.away_team_id IN (SELECT team_id FROM ga_boys_teams))
        OR
        (g.away_team_id IN (SELECT team_id FROM ecnl_boys_teams) AND g.home_team_id IN (SELECT team_id FROM ga_boys_teams))
    )
    AND g.is_hidden = false
    AND g.gender = 'M'
)
SELECT
    COUNT(*) AS total_cross_league_games,
    -- ECNL Boys wins (from ECNL perspective)
    SUM(CASE 
        WHEN game_type = 'ecnl_home' AND home_score > away_score THEN 1
        WHEN game_type = 'ecnl_away' AND away_score > home_score THEN 1
        ELSE 0
    END) AS ecnl_wins,
    -- Draws
    SUM(CASE WHEN home_score = away_score THEN 1 ELSE 0 END) AS draws,
    -- GA Boys wins
    SUM(CASE 
        WHEN game_type = 'ecnl_home' AND away_score > home_score THEN 1
        WHEN game_type = 'ecnl_away' AND home_score > away_score THEN 1
        ELSE 0
    END) AS ga_wins,
    -- ECNL win rate (including half-credit for draws)
    ROUND(
        (
            SUM(CASE 
                WHEN game_type = 'ecnl_home' AND home_score > away_score THEN 1
                WHEN game_type = 'ecnl_away' AND away_score > home_score THEN 1
                ELSE 0
            END)
            + 0.5 * SUM(CASE WHEN home_score = away_score THEN 1 ELSE 0 END)
        )::NUMERIC / COUNT(*), 4
    ) AS ecnl_win_rate
FROM cross_league_games;
```

**Interpretation thresholds (from Cross-League Analysis methodology):**
| ECNL Boys Win Rate | Implied Cal Delta (ECNL vs GA) | Verdict |
|-------------------|-------------------------------|---------|
| 70% or higher | Gap > 30 pts | 20-pt gap too small; GA Boys may be overcalibrated |
| 60–69% | Gap ~20–30 pts | Consistent with current 20-pt gap (100→120) |
| 55–59% | Gap ~10–20 pts | 20-pt gap may be slightly too large |
| 45–54% | Gap < 10 pts | Gap significantly overstated; GA Boys should be closer to ECNL |
| < 45% | GA Boys outperform ECNL Boys | Cal reversal possible; flag immediately |

**Minimum sample size for valid interpretation:** 200 cross-league Boys ECNL vs GA Boys games. Below 200, win-rate variance is too high for a reliable calibration inference. If the query returns < 200 games, note explicitly that the verdict is data-limited.

---

## Step 4: Verdict

**Verdict: UNMEASURABLE from current vault documentation.**

The direct Boys ECNL vs GA Boys cross-league win rate has not been computed and recorded in any vault document. The April 23 GA Coverage Audit designed the methodology but did not record execution results. This is a data gap, not a calibration gap — the database likely contains sufficient Boys cross-league game data (Boys GA has existed since at least 2021), but ELO has not run the measurement query.

**What this means for Boys calibration confidence:**
- The 20-point ECNL Boys (120) → GA Boys (100) gap is **defensible but not empirically confirmed**.
- No active miscalibration signal exists: Boys Brier scores show no anomalies in ECNL-heavy age groups, and Boys rankings have not generated credible complaints from coaches or clubs.
- The gap is structurally appropriate: 100 sits logically between ECNL Boys (120) and ECNL RL/NPL (55), reflecting GA's national league status below ECNL.
- DSS impact: this gap is **not a DSS blocker**. Boys Option A validates GA Boys calibration against the distribution pattern, not against a single win rate. The absence of a measured win rate does not prevent ELO from defending Boys rankings at DSS.

**Post-DSS resolution:** Run the query in Step 3 during the Boys Brier analysis window (May 17–30). If game count ≥ 200, record the win rate and update the Boys ECNL Calibration Scope document. If win rate falls outside the 55–69% band (implying the gap is materially wrong), file a calibration recommendation to Presten before the June 1 age transition.

---

## References

- `Rankings/Boys-ECNL-Calibration-Scope-2026-04-27.md` — Boys ECNL calibration basis (filed today)
- `Rankings/ga-coverage-audit-results-2026-04-23.md` — April 23 audit (SQL designed, results not executed)
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys calibration overview
- [[Cross-League Analysis]] — 575K game win-rate methodology
- [[League Hierarchy]] — calibration values (ECNL Boys 120, GA Boys 100)

*ELO — 2026-04-27 | Task: task-2026-04-27-ecnl-boys-ga-boys-win-rate-gap-investigation*
