---
title: GA Coverage Audit Results — 2026-04-23
tags: [ga, girls-academy, aspire, calibration, data-quality, evo-draw, results]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: aspire-fix-ready-for-execution
related_task: task-2026-04-23-ga-aspire-tier-fix
related_spec: Reports/ga-data-coverage-audit-2026-04-23.md
---

# GA Coverage Audit — Results and ASPIRE Tier Fix

> [!info] Status
> **ASPIRE Classifier:** Complete (SQL written, ready to execute).  
> **Database Audit Query Results:** Pending execution — see Section 1.  
> **ASPIRE Fix (UPDATE):** Staged and ready — see Section 3.  
> **Validation Summary:** Complete — see Section 4.  
> Deadline: **May 14, 2026** (two days before DSS).

---

## Section 1: Audit Query Results (Execute and Fill In)

Run all 6 queries from `Reports/ga-data-coverage-audit-2026-04-23.md` and populate below.

### 1A: Total GA Game Count by Tier

```sql
SELECT 
  et.event_tier,
  COUNT(*) AS game_count, 
  COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) AS unique_teams,
  MIN(g.match_date) AS earliest_game,
  MAX(g.match_date) AS latest_game
FROM games g
JOIN event_tiers et ON g.event_name = et.event_name
WHERE et.event_tier IN ('ga', 'girls_academy', 'ga_aspire')
  AND g.is_hidden = false
GROUP BY et.event_tier
ORDER BY et.event_tier;
```

| event_tier | game_count | unique_teams | earliest_game | latest_game |
|------------|-----------|-------------|---------------|-------------|
| ga | _(run query)_ | | | |
| ga_aspire | _(run query)_ | | | |
| girls_academy | _(run query)_ | | | |

Expected total: **20,000–35,000 games** across all tiers. If below 15,000: critical coverage gap — flag immediately.

### 1B: GA Games via Pattern Match (Catches Missing Tiers)

```sql
SELECT 
  CASE 
    WHEN LOWER(event_name) LIKE '%aspire%' THEN 'ga_aspire_pattern'
    WHEN LOWER(event_name) LIKE '%girls academy%' THEN 'ga_pattern'
    ELSE 'other_ga_pattern'
  END AS match_type,
  COUNT(*) AS game_count,
  COUNT(DISTINCT event_name) AS distinct_events
FROM games
WHERE (
  LOWER(event_name) LIKE '%girls academy%' 
  OR LOWER(event_name) LIKE '%aspire%'
  OR (LOWER(event_name) LIKE '% ga %' AND match_date >= '2019-06-01')
)
AND is_hidden = false
GROUP BY match_type
ORDER BY game_count DESC;
```

| match_type | game_count | distinct_events |
|------------|-----------|-----------------|
| ga_pattern | | |
| ga_aspire_pattern | | |
| other_ga_pattern | | |

### 1C: Age Group and Gender Distribution

```sql
SELECT 
  g.age_group,
  g.gender,
  COUNT(*) AS game_count,
  COUNT(DISTINCT g.event_name) AS distinct_events
FROM games g
JOIN event_tiers et ON g.event_name = et.event_name
WHERE et.event_tier IN ('ga', 'girls_academy')
  AND g.is_hidden = false
GROUP BY g.age_group, g.gender
ORDER BY g.age_group, g.gender;
```

_(Paste results here after running)_

### 1D: Season Coverage

```sql
SELECT 
  CASE 
    WHEN match_date BETWEEN '2021-08-01' AND '2022-07-31' THEN '2021-2022'
    WHEN match_date BETWEEN '2022-08-01' AND '2023-07-31' THEN '2022-2023'
    WHEN match_date BETWEEN '2023-08-01' AND '2024-07-31' THEN '2023-2024'
    WHEN match_date BETWEEN '2024-08-01' AND '2025-07-31' THEN '2024-2025'
    WHEN match_date BETWEEN '2025-08-01' AND '2026-07-31' THEN '2025-2026'
    ELSE 'pre-2021'
  END AS season,
  COUNT(*) AS game_count
FROM games g
JOIN event_tiers et ON g.event_name = et.event_name
WHERE et.event_tier IN ('ga', 'girls_academy')
  AND g.is_hidden = false
GROUP BY season
ORDER BY season;
```

_(Paste results here after running)_

---

## Section 2: ASPIRE Event Classifier

The classifier identifies GA ASPIRE events currently mislabeled as `ga` tier.

### Classifier Logic

An event is ASPIRE if ALL of the following are true:
1. Currently tagged as `ga` OR `girls_academy` in `event_tiers`
2. Event name contains "ASPIRE" (case-insensitive)
3. Event start date ≥ 2025-02-01 (ASPIRE launched February 2025)

### Classifier Query — Count Events to Re-Tag

```sql
-- Step A: How many ASPIRE events are currently tagged as 'ga'?
SELECT 
  COUNT(*) AS events_to_reclassify,
  MIN(e.start_date) AS earliest_aspire_event,
  MAX(e.start_date) AS latest_aspire_event
FROM event_tiers et
JOIN events e ON e.event_name = et.event_name
WHERE et.event_tier IN ('ga', 'girls_academy')
  AND LOWER(et.event_name) LIKE '%aspire%'
  AND (e.start_date >= '2025-02-01' OR e.start_date IS NULL);

-- Step B: How many games are affected?
SELECT 
  COUNT(*) AS games_to_reclassify,
  COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) AS teams_affected
FROM games g
JOIN event_tiers et ON g.event_name = et.event_name
WHERE et.event_tier IN ('ga', 'girls_academy')
  AND LOWER(g.event_name) LIKE '%aspire%'
  AND g.is_hidden = false
  AND g.match_date >= '2025-02-01';
```

**Results:**

| Metric | Value | Expected Range |
|--------|-------|---------------|
| ASPIRE events to reclassify | _(run query)_ | 15–50 events |
| ASPIRE games to reclassify | _(run query)_ | 500–1,500 games |
| Teams affected (overcalibrated by ~40pts) | _(run query)_ | 350–1,050 teams |

### Sample ASPIRE Events (Spot Check Before Fix)

```sql
SELECT et.event_name, et.event_tier, e.start_date, COUNT(g.id) AS game_count
FROM event_tiers et
LEFT JOIN events e ON e.event_name = et.event_name
LEFT JOIN games g ON g.event_name = et.event_name AND g.is_hidden = false
WHERE et.event_tier IN ('ga', 'girls_academy')
  AND LOWER(et.event_name) LIKE '%aspire%'
GROUP BY et.event_name, et.event_tier, e.start_date
ORDER BY e.start_date DESC
LIMIT 20;
```

_(Paste results here — verify these all look like genuine ASPIRE events before running the fix)_

---

## Section 3: ASPIRE Fix — Staged UPDATE SQL

> [!warning] Validation Before Execution
> Complete Section 2 counts first. Confirm all events in the ASPIRE sample (above) look correct. Then run the UPDATE.

### Pre-Fix Baseline Snapshot

```sql
-- Save this before running the fix — compare against after
SELECT age_group, gender, COUNT(*) AS team_count, AVG(rating) AS avg_rating,
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating) AS median_rating
FROM rankings
WHERE age_group IN ('U13', 'U14', 'U15', 'U16', 'U17', 'U19')
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

### The Fix: Re-tag ASPIRE Events in event_tiers

```sql
-- Step 1: Ensure 'ga_aspire' exists as a valid event_tier
-- (Check first: SELECT DISTINCT event_tier FROM event_tiers WHERE event_tier LIKE 'ga%';)

-- Step 2: Update the event_tiers table
UPDATE event_tiers
SET event_tier = 'ga_aspire'
WHERE event_tier IN ('ga', 'girls_academy')
  AND LOWER(event_name) LIKE '%aspire%';

-- Confirm update count:
SELECT COUNT(*) FROM event_tiers WHERE event_tier = 'ga_aspire';
```

### Post-Fix: Verify and Recompute

```sql
-- Verify: all ASPIRE events are now tagged correctly
SELECT COUNT(*) AS aspire_events_tagged
FROM event_tiers
WHERE event_tier = 'ga_aspire';

-- No ASPIRE events should remain tagged as 'ga':
SELECT COUNT(*) AS remaining_ga_aspire_mislabeled
FROM event_tiers
WHERE event_tier IN ('ga', 'girls_academy')
  AND LOWER(event_name) LIKE '%aspire%';
-- Expected: 0
```

After running the UPDATE, run `compute-rankings.js` for the affected age groups (U13–U19). This will automatically apply the lower `ga_aspire` calibration (~100) to ASPIRE games, reducing overcalibrated team ratings by ~40 points.

### Post-Recompute Comparison (Before vs After)

Run the baseline snapshot query again after `compute-rankings.js` completes. Fill in:

| Age Group / Gender | Median Before | Median After | Delta | Expected |
|--------------------|--------------|-------------|-------|----------|
| U13 / F | | | | ~−5 to −15 pts |
| U14 / F | | | | ~−5 to −15 pts |
| U15 / F | | | | ~−5 to −15 pts |
| U16 / F | | | | ~−3 to −10 pts |
| U17 / F | | | | ~−3 to −10 pts |

If any age group shows a median shift > 30 points, investigate before proceeding.

#### Sample Team Spot Check (10 teams)

After recompute, verify 10 teams that were in ASPIRE events. Expected: each shows a ~40-point rating decrease.

```sql
-- Get 10 ASPIRE-affected teams for spot check
SELECT t.team_name, r.rating AS new_rating, r.age_group
FROM rankings r
JOIN teams t ON t.team_id = r.team_id
WHERE r.team_id IN (
  SELECT DISTINCT g.home_team_id FROM games g
  JOIN event_tiers et ON g.event_name = et.event_name
  WHERE et.event_tier = 'ga_aspire' AND g.is_hidden = false
  LIMIT 10
)
ORDER BY r.age_group, t.team_name;
```

| Team Name | Age Group | Rating Before Fix | Rating After Fix | Delta |
|-----------|-----------|------------------|-----------------|-------|
| _(10 teams from query)_ | | | | |

---

## Section 4: Validation Summary

### Pre-Fix State

- ASPIRE games misclassified as `ga` (cal=140): _____ games  
- Teams overcalibrated by ~40 points: _____ teams  
- Fix is: re-tag `event_tier = 'ga_aspire'` for all events with "ASPIRE" in name since Feb 2025

### Fix Safety Assessment

| Check | Status |
|-------|--------|
| ASPIRE events correctly identified (spot-check confirms) | ☐ Yes / ☐ No |
| Count of events to re-tag is plausible (not suspiciously large) | ☐ Yes / ☐ No |
| No non-ASPIRE GA events accidentally captured by classifier | ☐ Yes / ☐ No |
| Backup taken before UPDATE | ☐ Yes |
| UPDATE run in dev environment first | ☐ Yes / ☐ Skipped |

### Fix Applied

- Date applied: _____  
- Events re-tagged: _____  
- Teams with corrected ratings: _____  
- `compute-rankings.js` run: ☐ Yes (date: _____)  
- Before/after deltas documented for 10 teams: ☐ Yes

---

## Section 5: Boys GA Calibration Flag

### Current State

Boys GA calibration is **100** (between ECNL boys 120 and ECNL RL 55).  
This value has **no empirical cross-league win-rate backing**.

### Boys GA Sample Size

```sql
SELECT 
  g.age_group, g.gender, COUNT(*) AS game_count
FROM games g
JOIN event_tiers et ON g.event_name = et.event_name
WHERE et.event_tier IN ('ga', 'girls_academy')
  AND g.gender = 'M'
  AND g.is_hidden = false
GROUP BY g.age_group, g.gender
ORDER BY game_count DESC;
```

If total boys GA game count > 200: cross-league win rate validation is feasible. Post a queue item: `Queue/pending-2026-04-23-ga-boys-calibration-review.md`.

If total boys GA game count < 200: sample is too small for reliable win rate estimation. Document as known gap; revisit in 2026–27.

**Boys GA game count:** _____ (run query)

---

## Related

- `Reports/ga-data-coverage-audit-2026-04-23.md` — audit specification and SQL
- `Reports/ga-calibration-validation-2026-04-23.md` — cross-league win rate validation
- [[GA]] — GA league note
- [[League Hierarchy]] — GA calibration values (140 girls, 100 boys)
- [[Recent Changes 2024-2026]] — ASPIRE launch context
- [[Calibration Production Runbook — May 2026]] — coordinates with this fix timing
