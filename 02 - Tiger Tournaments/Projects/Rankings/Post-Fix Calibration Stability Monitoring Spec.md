---
title: Post-Fix Calibration Stability Monitoring Spec
type: monitoring-spec
author: ELO
date: 2026-04-25
status: active — begins April 29
related: "[[Post-April-28 Rating Shift Analysis Spec]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]", "[[April 29 Post-Fix Decision Tree]]", "[[Boys Calibration Status Summary — April 2026]]"
tags: [calibration, monitoring, ga-aspire, stability, dss, evo-draw]
---

# Post-Fix Calibration Stability Monitoring Spec

> **Purpose:** Systematic tracking of calibration stability from April 29 through DSS (May 16). The April 29 analysis (`Post-April-28 Rating Shift Analysis Spec.md`) confirms the immediate post-fix state. This spec covers the next 17 days — ensuring the rating system stabilizes before the demo and that any instability is caught early enough to investigate.

---

## Section 1: Stability Definition

"Calibration stable" means the Girls GA ASPIRE rating correction has been absorbed by the system and new games are being processed accurately at cal=100. ELO defines stability by three criteria, all of which must hold:

**Criterion A — Rate of change:** Average Girls GA ASPIRE rating change per daily pipeline run is **< 3 points** across affected age groups (U13G–U17G), measured for **3 consecutive runs**. A 3-point per-day average means the correction has settled. Larger daily swings indicate the system is still adjusting (expected in the first 2–3 days post-fix).

**Criterion B — Distribution convergence:** The Girls GA ASPIRE average rating (by age group) is within **± 8 points** of the April 29 post-fix baseline for **5 consecutive days**. This confirms the initial correction isn't drifting further in either direction as new games are ingested.

**Criterion C — Outlier count:** The number of individual teams with a day-over-day rating change > **15 points** is fewer than **25 teams per pipeline run** (across all affected age groups combined). Spikes above 25 teams/run signal that new GA ASPIRE games are processing with unexpected weight, or that a non-ASPIRE source of volatility has been introduced.

**What "stable" means:** Ratings are moving predictably (small, expected increments as new games are ingested), not oscillating or drifting structurally. Stable does NOT mean ratings have stopped moving — in an active season, ratings always move. Stable means the movement is normal Glicko convergence, not artifact-driven correction.

**Threshold rationale:** The Girls GA ASPIRE overcalibration distortion was 40+ points. A system with 40-pt instability is not stable. A system with 2-pt day-over-day variation is fully stable. The thresholds above (3-pt avg change, 8-pt convergence window, 25-team outlier cap) are set at roughly 10% of the distortion magnitude — small enough to catch real instability, large enough not to fire on normal Glicko variation.

---

## Section 2: Monitoring Query Pack

Three queries. Run after each pipeline execution from April 29 through May 9. Save results dated by run date. Query runtime: < 2 minutes total.

### Query 1 — Girls GA ASPIRE Average Rating Trend

Run after each pipeline execution. Detects ongoing drift in the corrected cohort.

```sql
-- Save output to: reports/ga-aspire-stability-[YYYY-MM-DD].txt
SELECT
  r.age_group,
  r.gender,
  ROUND(AVG(r.rating)::numeric, 1) AS avg_rating,
  ROUND(STDDEV(r.rating)::numeric, 1) AS stddev_rating,
  COUNT(*) AS team_count,
  NOW()::DATE AS run_date
FROM rankings r
WHERE r.team_id IN (
  SELECT DISTINCT g.home_team_id FROM games g
  JOIN events e ON e.id = g.event_id
  WHERE e.event_tier = 'ga_aspire'
  UNION
  SELECT DISTINCT g.away_team_id FROM games g
  JOIN events e ON e.id = g.event_id
  WHERE e.event_tier = 'ga_aspire'
)
AND r.gender = 'F'
AND r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
GROUP BY r.age_group, r.gender
ORDER BY r.age_group;
```

**How to use:** Compare today's `avg_rating` per age group to yesterday's output. The day-over-day delta should be < 3 pts per cohort after April 30 (April 29 itself will show the largest shift). Log each delta in the daily briefing as a one-liner: "GA ASPIRE stability: U13G +0.8, U14G +1.2, U15G +0.4, U16G +0.1, U17G −0.3 → PASS all < 3."

**FAIL signal:** Any single cohort delta > 8 pts two days in a row.

---

### Query 2 — Daily Rating Volatility Check

Counts teams with day-over-day rating changes above threshold. High count is the earliest signal of instability.

```sql
-- Requires: yesterday's ratings in a snapshot table OR manual comparison.
-- If pipeline does not maintain a historical ratings table, save yesterday's
-- Query 1 output and compare manually. The count below is the target format.

-- Alternative: use updated_at timestamps to find recently-changed teams
SELECT
  r.age_group,
  r.gender,
  COUNT(*) AS team_count,
  SUM(CASE WHEN ABS(r.rating - r.prior_rating) > 15 THEN 1 ELSE 0 END) AS large_movers,
  MAX(ABS(r.rating - r.prior_rating)) AS max_single_move,
  NOW()::DATE AS run_date
FROM rankings r
WHERE r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
AND r.gender = 'F'
GROUP BY r.age_group, r.gender
ORDER BY r.age_group;

-- If rankings table does not have prior_rating column, use this alternative:
-- Compare the max/min rating within the last 2 pipeline runs as a proxy
SELECT
  r.age_group,
  COUNT(*) FILTER (WHERE ABS(r.rating - 1200) > 250) AS extreme_ratings,
  MAX(r.rating) AS max_rating,
  MIN(r.rating) AS min_rating,
  COUNT(*) AS team_count
FROM rankings r
WHERE r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
AND r.gender = 'F'
GROUP BY r.age_group;
```

**PASS:** `large_movers` count < 25 across all age groups combined.
**FAIL:** `large_movers` > 25 combined — flag in briefing as "Volatility spike: [N] teams moved > 15 pts." ELO investigates source before next run.

**Max single move threshold:** Any single team moving > 80 pts in a single pipeline run is a data artifact (new game batch ingested incorrectly, merge applied mid-run, or stale team re-entering). Investigate immediately.

---

### Query 3 — Cross-Gender Convergence Check

After the fix, Girls GA ASPIRE ratings should move **toward** Girls ECNL average, not away. This query tracks the gap.

```sql
-- Run weekly (May 5, May 9, May 14) or whenever instability is flagged.
SELECT
  sub.age_group,
  ROUND(AVG(CASE WHEN sub.source = 'ga_aspire' THEN sub.avg_rating END)::numeric, 1) AS ga_aspire_avg,
  ROUND(AVG(CASE WHEN sub.source = 'ecnl' THEN sub.avg_rating END)::numeric, 1) AS ecnl_avg,
  ROUND(
    AVG(CASE WHEN sub.source = 'ecnl' THEN sub.avg_rating END) -
    AVG(CASE WHEN sub.source = 'ga_aspire' THEN sub.avg_rating END),
    1
  ) AS delta_ecnl_minus_ga_aspire,
  NOW()::DATE AS run_date
FROM (
  -- GA ASPIRE teams
  SELECT 'ga_aspire' AS source, r.age_group, r.gender, AVG(r.rating) AS avg_rating
  FROM rankings r
  WHERE r.team_id IN (
    SELECT DISTINCT home_team_id FROM games g
    JOIN events e ON e.id = g.event_id WHERE e.event_tier = 'ga_aspire'
    UNION
    SELECT DISTINCT away_team_id FROM games g
    JOIN events e ON e.id = g.event_id WHERE e.event_tier = 'ga_aspire'
  )
  AND r.gender = 'F' AND r.age_group IN ('U13','U14','U15','U16','U17')
  GROUP BY r.age_group, r.gender

  UNION ALL

  -- ECNL Girls teams
  SELECT 'ecnl' AS source, r.age_group, r.gender, AVG(r.rating) AS avg_rating
  FROM rankings r
  WHERE r.team_id IN (
    SELECT DISTINCT home_team_id FROM games g
    JOIN events e ON e.id = g.event_id WHERE e.event_tier = 'ecnl'
    UNION
    SELECT DISTINCT away_team_id FROM games g
    JOIN events e ON e.id = g.event_id WHERE e.event_tier = 'ecnl'
  )
  AND r.gender = 'F' AND r.age_group IN ('U13','U14','U15','U16','U17')
  GROUP BY r.age_group, r.gender
) sub
GROUP BY sub.age_group
ORDER BY sub.age_group;
```

**How to use:** Track `delta_ecnl_minus_ga_aspire` across runs. After April 29:
- This delta should be **positive** (ECNL average is above GA ASPIRE average — ECNL is Tier 1A)
- The delta should be **narrowing or stable** as GA ASPIRE teams correct upward
- If the delta is **widening** (GA ASPIRE falling further below ECNL), that is a convergence failure

**Expected baseline (April 29, post-fix):** ECNL Girls avg − GA ASPIRE Girls avg ≈ 15–40 pts per age group.
**Stability threshold:** Delta should not widen by more than 10 pts from the April 29 baseline over the monitoring period.

---

## Section 3: Monitoring Schedule

| Date | Action | Pass Condition | Decision |
|------|--------|---------------|----------|
| **April 29** | Run full post-fix analysis (Post-April-28 Rating Shift Analysis Spec). File baseline in briefing. Establish Query 1 + Query 3 baseline values. | G1–G4 all pass (Decision Tree). | GREEN → continue to Event Strength Phase 1 and Rank Bands. Any RED → hold downstream per Decision Tree. |
| **April 30** | Run Query 1 + Query 2 after pipeline. File deltas in briefing. | Day-over-day avg change < 8 pts (first day — allowance for post-fix adjustment). Large movers count < 50. | Expected some drift today. Flag to SENTINEL only if avg change > 15 pts per cohort. |
| **May 1** | Run Query 1 + Query 2 after pipeline. | Avg change < 5 pts per cohort. Large movers < 35. | If > 5 pts avg → "Persistent adjustment — monitoring closely." No hold yet. |
| **May 2** | Run Query 1 + Query 2 after pipeline. | Avg change < 3 pts per cohort. Large movers < 25. | If Criterion A is met for the first time today, start the 3-run streak counter. |
| **May 5** | Run full 3-query pack. File stability status in briefing. | All three stability criteria met (Criteria A, B, C). | Decision: "Stable" → Event Strength Phase 1 cleared from calibration stability dependency. "Not stable" → ELO investigation before authorizing Event Strength Phase 1. |
| **May 7–8** | Run Query 1 + Query 2 after each pipeline run. | Avg change < 3 pts. Large movers < 25. | Confirm stability holds through Club Rankings implementation window (May 2–3 session is complete by now). |
| **May 9** | Run full 3-query pack. File stability status in briefing. Final stability check before DSS readiness assessment. | All three criteria met; convergence check (Query 3) shows stable delta. | Must be stable to authorize DSS readiness go/no-go. |
| **May 14** | Pre-demo spot check. Run Query 1. | Avg change < 3 pts for all cohorts. No large movers spike. | Any instability flagged to SENTINEL by 11 AM. Do not wait for EOD — there are only 2 days before DSS. |

**Minimum frequency:** Query 1 and Query 2 after every pipeline run. If pipeline runs every night, that is daily. If pipeline runs less frequently, run the queries when Presten is in the DB regardless of pipeline timing.

---

## Section 4: Instability Response Protocol

**Level 1 — Mild instability (single-day spike, resolves next run):**
- Definition: One cohort exceeds the avg change threshold for a single run, then returns to normal.
- ELO action: Log in briefing as "Spike observed [date], cohort [age group], delta [N] pts. Resolved next run." No hold on downstream tasks.
- Example: U14G avg moves 6 pts one day, returns to 1.5 pts the next. This is expected as a batch of new ASPIRE-tagged games gets ingested.

**Level 2 — Persistent instability (2+ consecutive runs above threshold):**
- Definition: Same cohort exceeds threshold for 2 or more consecutive runs.
- ELO action: Flag in briefing: "Calibration instability detected — [specific signal]. Investigating source."
- ELO identifies whether the instability is sourced from:
  - New GA ASPIRE games being ingested (normal — will self-correct as game volume accumulates, ratings converge)
  - A computation error (miscalibration in the updated cal map, merge artifact, wrong tier tag for a new event)
- SENTINEL holds Event Strength Phase 1 authorization until ELO files a clearance determination.
- Resolution: ELO posts a one-line determination in the next briefing: "Level 2 spike source identified as [new game batch / computation error]. [Action taken]. Stability cleared."

**Level 3 — Structural instability (distribution diverging, wrong direction):**
- Definition: Query 3 shows the ECNL–GA ASPIRE delta is WIDENING by > 10 pts from the April 29 baseline; OR GA ASPIRE average is moving in the wrong direction (downward from the post-fix baseline) over 3+ consecutive runs.
- ELO action: Escalate directly to SENTINEL in the same briefing. File a Queue item: `Queue/pending-[date]-ga-aspire-calibration-regression.md` with category: alert, priority: urgent.
- SENTINEL action: Hold Event Strength Phase 1 and Rank Bands implementation. Alert Presten.
- April 28 fix may need revision — the calibration value or the UPDATE scope may be incorrect.
- DSS demo risk elevated: ELO flags Girls GA ASPIRE-dependent features (Club Rankings Girls, Rank Bands for GA ASPIRE teams) as uncertain in the DSS Feature Readiness Tracker.

---

## Section 5: Stability Clearance Protocol

When ELO determines calibration is stable (all three criteria met for 3 consecutive runs):

**In the next briefing, include this block:**

> **GA ASPIRE Calibration Stability Confirmed**
>
> Date: [date]
> Stability criteria met for 3 consecutive runs (April [X], [X+1], [X+2]).
>
> - Criterion A (rate of change < 3 pts/run): ✅ — max observed: [N] pts [age group]
> - Criterion B (distribution convergence within ±8 pts of Apr 29 baseline): ✅ — max drift: [N] pts [age group]
> - Criterion C (outlier count < 25 teams/run): ✅ — max observed: [N] teams
>
> Event Strength Phase 1 cleared from calibration stability dependency.
> Club Rankings implementation cleared from calibration stability dependency.

**SENTINEL records clearance** in the next SENTINEL briefing and removes the "monitoring GA ASPIRE stability" flag from the active monitoring section.

---

## References

- `Rankings/Post-April-28 Rating Shift Analysis Spec.md` — April 29 analysis that establishes the post-fix baseline this spec monitors against
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — predicted values this spec compares against
- `Rankings/April 29 Post-Fix Decision Tree.md` — G2 gate that feeds into this spec's baseline
- `Rankings/Boys Calibration Status Summary — April 2026.md` — confirms Boys should not be affected
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — calibration flags this spec maintains

*ELO — 2026-04-25*
