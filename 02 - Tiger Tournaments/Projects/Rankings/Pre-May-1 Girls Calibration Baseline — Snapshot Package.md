---
title: Pre-May-1 Girls Calibration Baseline — Snapshot Package
type: calibration-snapshot
author: ELO
date: 2026-04-25
status: ready-to-run
due: 2026-04-30
related_task: task-2026-04-25-pre-may1-girls-calibration-baseline
related: "[[Post-Fix Calibration Stability Monitoring Spec]]", "[[April 29 Post-Fix Decision Tree]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]"
tags: [calibration, baseline, girls, ga-aspire, monitoring, evo-draw]
---

# Pre-May-1 Girls Calibration Baseline — Snapshot Package

> **Purpose:** Capture the stable post-fix, pre-production Girls calibration state immediately after the April 29 analysis confirms GREEN. This becomes the comparison baseline for all stability monitoring through May 14 — not the April 29 Decision Tree baseline, which reflects the DB session state rather than the production-pipeline-ready state.
>
> **Critical timing:** Run **after** April 29 Decision Tree verdict is GREEN (or INVESTIGATE-not-HOLD). Run **before** the May 1 pipeline executes. If April 29 verdict is RED: do not run — the baseline would capture an unstable state that invalidates all downstream monitoring.
>
> **Time required:** ~10 minutes in psql. Save output to `Rankings/Pre-May-1 Baseline Results — [date].md`.

---

## When to Run

**Trigger:** April 29 Decision Tree gate G1 returns GREEN or INVESTIGATE (not HOLD, not RED).

**Window:** April 29 post-session through midnight April 30. If this window closes (May 1 pipeline runs before baseline is captured), the baseline is invalid — flag to SENTINEL. ELO will use the May 1 post-pipeline snapshot as a secondary reference, but the analytical gap should be noted in the next briefing.

**Why this specific timing:** The pre-May-1 baseline must capture the post-fix, pre-production-pipeline state. Once the May 1 pipeline executes, new games are ingested and ratings begin moving again. The monitoring spec compares each May pipeline run against *this* baseline, not against April 29's initial recompute output. These are different: April 29 is the "recompute result snapshot"; pre-May-1 is the "stable baseline that production monitors against."

---

## Snapshot Query 1 — Girls GA ASPIRE Average Rating by Age Group

Primary calibration health signal. Captures the post-fix GA ASPIRE cohort average that all subsequent stability monitoring compares against.

```sql
-- Pre-May-1 Baseline: Girls GA ASPIRE average by age group
-- Run after April 29 Decision Tree verdict = GREEN, before May 1 pipeline
SELECT
  r.age_group,
  ROUND(AVG(r.rating), 1) AS avg_rating,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY r.rating)::int AS median_rating,
  ROUND(STDDEV(r.rating), 1) AS stddev_rating,
  COUNT(*) AS team_count
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.gender = 'F'
  AND r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND r.rank IS NOT NULL
  AND t.id IN (
    -- Teams with at least 1 GA ASPIRE game
    SELECT DISTINCT g.home_team_id FROM games g
    JOIN event_tiers et ON g.event_name = et.event_name
    WHERE et.event_tier = 'ga_aspire' AND g.is_hidden = false
    UNION
    SELECT DISTINCT g.away_team_id FROM games g
    JOIN event_tiers et ON g.event_name = et.event_name
    WHERE et.event_tier = 'ga_aspire' AND g.is_hidden = false
  )
GROUP BY r.age_group
ORDER BY r.age_group;
```

**Save result to:** `Rankings/Pre-May-1 Baseline Results — [date].md`

**Expected output shape:** 5 rows (U13–U17), avg_rating values reflecting the post-fix increase relative to pre-April-28. From the Pre-Run Model: U14G avg should be highest among Girls GA ASPIRE cohorts; U17G lowest. Exact values confirmed from this query.

---

## Snapshot Query 2 — Girls Cross-Tier Rating Distribution

Captures the full Girls rating landscape across all tier groups at the post-fix state. Used as the reference distribution for detecting cross-tier drift during May pipeline runs.

```sql
-- Pre-May-1 Baseline: Girls cross-tier distribution
-- Groups by calibration tier, age group
SELECT
  COALESCE(et.event_tier, 'unclassified') AS tier_group,
  r.age_group,
  ROUND(AVG(r.rating), 1) AS avg_rating,
  COUNT(DISTINCT r.team_id) AS team_count
FROM rankings r
JOIN teams t ON t.id = r.team_id
LEFT JOIN (
  -- Get the primary event tier for each team (by game count)
  SELECT
    COALESCE(g.home_team_id, g.away_team_id) AS team_id,
    et.event_tier,
    COUNT(*) AS game_count,
    ROW_NUMBER() OVER (
      PARTITION BY COALESCE(g.home_team_id, g.away_team_id)
      ORDER BY COUNT(*) DESC
    ) AS rn
  FROM games g
  JOIN event_tiers et ON g.event_name = et.event_name
  WHERE g.is_hidden = false
  GROUP BY COALESCE(g.home_team_id, g.away_team_id), et.event_tier
) et ON et.team_id = r.team_id AND et.rn = 1
WHERE r.gender = 'F'
  AND r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND r.rank IS NOT NULL
GROUP BY et.event_tier, r.age_group
ORDER BY r.age_group, avg_rating DESC;
```

**Save result to:** Same results file as Query 1.

**Key rows to watch:** `ga_aspire` avg should be within 40 pts of `ecnl` avg per age group (convergence effect of the fix). If `ga_aspire` avg is still more than 80 pts below `ecnl` avg for any age group, flag to SENTINEL — partial fix or underpowered correction.

---

## Snapshot Query 3 — Girls Rating Distribution Percentiles

Anchors the distribution shape for drift detection. During May monitoring, if the p50 or p90 for a Girls age group moves more than 10 pts from this baseline, the stability monitoring spec's Level 2 threshold is triggered.

```sql
-- Pre-May-1 Baseline: Girls rating percentiles by age group
SELECT
  r.age_group,
  PERCENTILE_CONT(0.10) WITHIN GROUP (ORDER BY r.rating)::int AS p10,
  PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY r.rating)::int AS p25,
  PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY r.rating)::int AS p50_median,
  PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY r.rating)::int AS p75,
  PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY r.rating)::int AS p90,
  PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY r.rating)::int AS p99,
  COUNT(*) AS total_ranked
FROM rankings r
WHERE r.gender = 'F'
  AND r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND r.rank IS NOT NULL
GROUP BY r.age_group
ORDER BY r.age_group;
```

**Save result to:** Same results file as Queries 1 and 2.

**Design reference:** The Rank Bands threshold design assumes p50 ≈ 1050 (Red floor). If post-fix U13G median is significantly different, note it — it may affect whether the April 28 Rank Bands validation thresholds need adjustment.

---

## Baseline Reference Table

Fill this after queries run. This is the standing lookup for all stability monitoring through May 14.

| Age Group | GA ASPIRE Avg Rating | ECNL Avg Rating | MLS NEXT Avg Rating | p50 (Cohort Median) | Date Captured |
|-----------|---------------------|-----------------|---------------------|---------------------|---------------|
| U13G | | | | | |
| U14G | | | | | |
| U15G | | | | | |
| U16G | | | | | |
| U17G | | | | | |

**How to use this table:**
- Stability Monitoring Query 1 (`Post-Fix Calibration Stability Monitoring Spec.md`) compares each pipeline run's Girls GA ASPIRE avg against the "GA ASPIRE Avg Rating" column here.
- If any age group's GA ASPIRE avg deviates > 15 pts from this baseline on a single run → Level 2 flag.
- If deviation > 30 pts or holds for 2+ consecutive runs → Level 3 escalation.

---

## Integration with Monitoring Spec

This document is referenced by `Rankings/Post-Fix Calibration Stability Monitoring Spec.md`. The Monitoring Spec's Queries 1–3 compare against this pre-May-1 baseline, not against the April 29 Decision Tree recompute output.

**Note for Monitoring Spec users:** If this baseline was not captured before May 1 pipeline launched (window closed), note in the briefing. Use the May 1 post-pipeline output as a secondary baseline — but the first 24 hours of May pipeline comparison data will be unavailable and must be acknowledged as a monitoring gap.

---

## Related

- [[Post-Fix Calibration Stability Monitoring Spec]] — the monitoring spec this baseline supports
- [[April 29 Post-Fix Decision Tree]] — GREEN verdict required before running this baseline
- [[Post-April-28 Expected Rating Shift — Pre-Run Model]] — predicted values to compare against when filling the Baseline Reference Table

*ELO — 2026-04-25*
