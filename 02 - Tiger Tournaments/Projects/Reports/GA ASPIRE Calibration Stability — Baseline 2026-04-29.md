---
title: GA ASPIRE Calibration Stability — Baseline 2026-04-29
type: monitoring-baseline
author: ELO
date: 2026-04-29
status: TEMPLATE — fill in on April 29 after G1 gate passes
related: "[[Post-Fix Calibration Stability Monitoring Spec]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]", "[[April 29 Post-Fix Decision Tree]]"
tags: [calibration, ga-aspire, stability, monitoring, baseline, evo-draw]
---

# GA ASPIRE Calibration Stability — Baseline 2026-04-29

> **TEMPLATE — Do not file until April 29 G1 gate passes (recompute confirmed complete).**
> Fill in all three query result sections during the April 29 session. File before closing the session. SENTINEL reads this alongside the April 29 Verdict Filing Document.

---

## Header

```
Baseline Type: Post-April-28 Fix — Initial Stability Baseline
Date: 2026-04-29
Session context: First read after GA ASPIRE event_tier fix (ga → ga_aspire) and full recompute
Pre-condition: G1 gate passed (April 28 Execution Log complete, recompute confirmed)
ELO notes: [any anomalies in session context that affect baseline interpretation]
```

---

## Query 1 Results: Girls GA ASPIRE Average Rating by Age Group

**Source:** Monitoring Spec Section 2, Query 1.

```sql
-- Copy-paste from Post-Fix Calibration Stability Monitoring Spec.md
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

**Results:**

| Age Group | Avg Rating (Post-Fix) | Std Dev | Team Count | Predicted Range (Pre-Run Model) | Within Range? |
|-----------|----------------------|---------|------------|---------------------------------|---------------|
| U13G      |                      |         |            | +5 to +15 pts above pre-fix avg | YES / NO      |
| U14G      |                      |         |            | +8 to +18 pts above pre-fix avg | YES / NO      |
| U15G      |                      |         |            | +6 to +15 pts above pre-fix avg | YES / NO      |
| U16G      |                      |         |            | +4 to +12 pts above pre-fix avg | YES / NO      |
| U17G      |                      |         |            | +2 to +8 pts above pre-fix avg  | YES / NO      |

**Baseline annotation:** Pre-fix averages from `Rankings/Pre-April-28 Baseline Snapshot`. If pre-fix baseline is not available in session, note "pre-fix comparison deferred to G2 analysis" and use this table as the April 29 absolute baseline only.

---

## Query 2 Results: Daily Rating Volatility Check

**Source:** Monitoring Spec Section 2, Query 2.

> **⚠️ Special interpretation for April 29:** This first Query 2 run captures the fix effect itself — the rating delta from the full recompute. The resulting large_movers count reflects the correction, not instability. Use April 30 as the first true stability datapoint for Criterion C. Document explicitly: "April 29 volatility is fix-delta, not signal."

```sql
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
```

**Results:**

```
Large movers count (teams with > 15 pt change since pre-recompute): ___
Maximum single-team rating change: ___ points
Team: ___ (if identifiable — large single moves > 80 pts should be noted)
Volatility status (April 29): EXPECTED FIX DELTA — not a stability metric
Stability comparison starts: April 30
```

**April 29 volatility ceiling:** Record large_movers count here as the reference ceiling. Subsequent runs are compared against this (expected: declining rapidly, < 25 by May 2).

---

## Query 3 Results: Cross-Gender Convergence Check

**Source:** Monitoring Spec Section 2, Query 3.

> Run on April 29 to establish convergence baseline. Expected: Girls GA ASPIRE avg has moved **toward** Girls ECNL avg (gap narrowed). If gap has widened, escalate to SENTINEL immediately — this is a G2 fail signal.

```sql
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

**Results:**

| Age Group | GA ASPIRE Avg | ECNL Avg | Delta (ECNL − GA ASPIRE) | Expected Delta | Direction OK? |
|-----------|--------------|----------|--------------------------|----------------|---------------|
| U13G      |              |          |                          | 15–40 pts      | YES / NO      |
| U14G      |              |          |                          | 15–40 pts      | YES / NO      |
| U15G      |              |          |                          | 15–40 pts      | YES / NO      |
| U16G      |              |          |                          | 15–40 pts      | YES / NO      |
| U17G      |              |          |                          | 15–40 pts      | YES / NO      |

**Convergence check:** Is delta_ecnl_minus_ga_aspire **smaller** than the pre-fix delta? If YES → convergence confirmed. If NO (delta widened) → escalate to SENTINEL immediately.

Pre-fix delta (from Pre-April-28 Baseline Snapshot, if available): ___

---

## Baseline Status Assessment

```
Baseline Filed: YES / NO
Date filed: 2026-04-29
All 3 queries executed: YES / NO (if NO: reason ___)
G1 gate confirmed before filing: YES / NO

Query 1 — GA ASPIRE avg ratings direction matches prediction: YES / NO
Query 2 — Fix-delta volatility count (for reference ceiling): ___
Query 3 — Convergence direction correct (delta narrowed): YES / NO

Anomalies: [none / description]

ELO stability assessment at baseline (one sentence): ___________

Next monitoring run: April 30
April 30 runs: Query 1 + Query 2 only
April 30 thresholds: avg change < 8 pts per cohort (day 1 allowance), large movers < 50
```

---

## Stability Monitoring Log

Use this section to append results from subsequent runs. One row per pipeline run.

| Date | Q1 Max Avg Change | Q2 Large Movers | Q2 Max Single Move | Criterion A | Criterion B | Criterion C | Status |
|------|-------------------|-----------------|-------------------|-------------|-------------|-------------|--------|
| Apr 29 | ___ (fix delta) | ___ (fix delta) | ___ | — | — | — | BASELINE |
| Apr 30 | | | | | | | |
| May 1  | | | | | | | |
| May 2  | | | | | | | |
| May 5  | | | | | | | FULL CHECK |
| May 7  | | | | | | | |
| May 8  | | | | | | | |
| May 9  | | | | | | | FULL CHECK |
| May 14 | | | | | | | PRE-DEMO |

**Criteria reminders:**
- Criterion A: avg change per cohort < 3 pts for 3 consecutive runs
- Criterion B: all age groups within ±8 pts of April 29 baseline for 5 consecutive days
- Criterion C: large_movers count < 25 per run across all cohorts combined

**Stability clearance:** Confirm when all 3 criteria met for 3 consecutive runs. File clearance note in next ELO briefing using the format in Monitoring Spec Section 5.

---

*ELO — template prepared 2026-04-26, filled 2026-04-29*
