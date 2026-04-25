---
title: Event Strength Phase 1 — Full Execution Package
type: implementation-sql
author: ELO
date: 2026-04-24
status: ready-to-run
ordering_constraint: "MUST run AFTER April 28 GA ASPIRE fix and ranking recompute"
related: "[[Event Strength Ratings — Implementation Plan]]", "[[Event Strength — League Tier Coverage Audit]]", "[[DSS Data Readiness Validation Plan — May 14-15]]"
tags: [event-strength, sql, phase-1, implementation, evo-draw]
---

# Event Strength Phase 1 — Full Execution Package

> **Summary:** Complete Phase 1 execution package for computing `event_strength` ratings. Includes pre-flight gates, remediation SQL for 6 unconfirmed leagues, schema DDL, computation INSERT, and 4 validation queries. **Critical: do not run before the April 28 GA ASPIRE fix.** Running before that fix will produce incorrect strength ratings for GA ASPIRE events that require a full Phase 1 re-run.

---

## Section 0: Pre-Flight Gates

Run both gates before any Phase 1 SQL. Both must pass.

**Gate A — Confirm April 28 GA ASPIRE fix has run**

```sql
-- Must return 0. If >0, GA ASPIRE fix has NOT been applied. Stop.
SELECT COUNT(*) AS aspire_events_still_tagged_ga
FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga';
```

Expected: `0`. If this returns any positive number, the April 28 GA ASPIRE UPDATE has not run. Do not proceed with Phase 1 — run the GA ASPIRE fix first (see `02 - Tiger Tournaments/Projects/Reports/ga-aspire-tier-fix-2026-04-24.md`).

**Gate B — Confirm null event_tier count**

```sql
-- Count events with no event_tier assigned
SELECT COUNT(*) AS null_tier_events
FROM events
WHERE event_tier IS NULL
  AND event_name IS NOT NULL;
```

- If result = 0: skip Section 1 and proceed directly to Section 2.
- If result > 0: run Section 1 remediation SQL, then re-run Gate B until it returns 0.

---

## Section 1: Remediation SQL (Run Only If Gate B > 0)

Assigns `event_tier` to the 6 circuits that had no confirmed tier assignment as of the League Tier Coverage Audit (2026-04-24). Run steps in order.

```sql
-- STEP 1: USL Academy events
UPDATE events
SET event_tier = 'usl_academy'
WHERE event_tier IS NULL
  AND (event_name ILIKE '%USL Academy%' OR event_name ILIKE '%USLA%');

-- STEP 2: EDP events
UPDATE events
SET event_tier = 'edp'
WHERE event_tier IS NULL
  AND event_name ILIKE '%EDP%';

-- STEP 3: Elite 64 events
UPDATE events
SET event_tier = 'elite_64'
WHERE event_tier IS NULL
  AND (event_name ILIKE '%Elite 64%' OR event_name ILIKE '%E64%');

-- STEP 4: NAL events
UPDATE events
SET event_tier = 'nal'
WHERE event_tier IS NULL
  AND event_name ILIKE '%NAL%';

-- STEP 5: USYS State Cup and state league events
UPDATE events
SET event_tier = 'state'
WHERE event_tier IS NULL
  AND (event_name ILIKE '%State Cup%' OR event_name ILIKE '%USYS%State%');

-- STEP 6: Remaining null-tier events (ODP, unidentified local/regional)
UPDATE events
SET event_tier = 'local'
WHERE event_tier IS NULL;
```

> [!warning] These tier assignments are calibration estimates, not empirically validated values.
> USL Academy (55), EDP (55), Elite 64 (55), NAL (25–55), and State (35) are structural estimates based on competitive pathway position. Do NOT apply these as Ranking Engine calibration values without Brier analysis. For Event Strength classification (which uses avg_team_rating percentile, not calibration values directly), these assignments are used for the tier-vs-strength cross-tab validation only.

After running all 6 steps, confirm:
```sql
SELECT COUNT(*) AS remaining_null_tiers FROM events WHERE event_tier IS NULL;
-- Expected: 0
```

Re-run Gate B. Must return 0 before continuing to Section 2.

---

## Section 2: Schema — event_strength Table

```sql
CREATE TABLE IF NOT EXISTS event_strength (
  id                  SERIAL PRIMARY KEY,
  event_id            INTEGER NOT NULL REFERENCES events(id),
  age_group           VARCHAR(10) NOT NULL,
  season              VARCHAR(10) NOT NULL,
  avg_team_rating     NUMERIC(8,2),
  strength_class      VARCHAR(20),
  strength_percentile NUMERIC(5,2),
  competitive_spread  NUMERIC(8,2),
  upset_rate          NUMERIC(5,4),
  team_count          INTEGER,
  computed_at         TIMESTAMP DEFAULT NOW(),
  UNIQUE (event_id, age_group)
);
```

If the table already exists from a prior attempt:
```sql
TRUNCATE event_strength;
```
This clears all rows without dropping the table — useful if re-running Phase 1 after a correction.

---

## Section 3: Computation INSERT

This is a single-pass CTE that computes all metrics and inserts them in one statement. Single-pass is preferred over multi-step inserts because it avoids partial-result state if the session is interrupted.

```sql
INSERT INTO event_strength (
  event_id, age_group, season,
  avg_team_rating, strength_class, strength_percentile,
  competitive_spread, upset_rate, team_count, computed_at
)

WITH

-- Step 1: All games with team ratings and event metadata
game_context AS (
  SELECT
    g.id              AS game_id,
    g.event_id,
    e.age_group,
    e.season,
    g.home_team_id,
    g.away_team_id,
    g.home_score,
    g.away_score,
    g.game_date,
    rh.rating         AS home_rating,
    ra.rating         AS away_rating
  FROM games g
  JOIN events e ON e.id = g.event_id
  LEFT JOIN rankings rh ON rh.team_id = g.home_team_id AND rh.age_group = e.age_group
  LEFT JOIN rankings ra ON ra.team_id = g.away_team_id AND ra.age_group = e.age_group
  WHERE e.season IS NOT NULL
    AND e.age_group IS NOT NULL
    AND g.home_score IS NOT NULL
    AND g.away_score IS NOT NULL
),

-- Step 2: Per-event, per-age-group metrics
event_metrics AS (
  SELECT
    event_id,
    age_group,
    season,
    AVG(rating)                                   AS avg_team_rating,
    STDDEV(rating)                                AS competitive_spread,
    COUNT(DISTINCT team_id)                       AS team_count,
    -- Upset rate: fraction of games where lower-rated team won
    SUM(CASE
      WHEN home_rating < away_rating AND home_score > away_score THEN 1
      WHEN away_rating < home_rating AND away_score > home_score THEN 1
      ELSE 0
    END)::NUMERIC / NULLIF(SUM(CASE
      WHEN home_rating IS NOT NULL AND away_rating IS NOT NULL
        AND home_rating <> away_rating THEN 1
      ELSE 0
    END), 0)                                      AS upset_rate
  FROM (
    -- Unnest home and away teams into a single ratings column for avg/stddev
    SELECT
      event_id, age_group, season,
      home_team_id AS team_id, home_rating AS rating,
      home_score, away_score, home_rating, away_rating
    FROM game_context
    WHERE home_rating IS NOT NULL
    UNION ALL
    SELECT
      event_id, age_group, season,
      away_team_id, away_rating,
      away_score, home_score, away_rating, home_rating
    FROM game_context
    WHERE away_rating IS NOT NULL
  ) all_teams
  GROUP BY event_id, age_group, season
  HAVING COUNT(DISTINCT team_id) >= 4  -- minimum 4 teams for a meaningful event
),

-- Step 3: Percentile ranking within (age_group, season)
event_with_percentile AS (
  SELECT
    *,
    ROUND(
      PERCENT_RANK() OVER (
        PARTITION BY age_group, season
        ORDER BY avg_team_rating ASC
      ) * 100, 2
    ) AS strength_percentile
  FROM event_metrics
  WHERE avg_team_rating IS NOT NULL
),

-- Step 4: Assign strength_class from percentile
event_classified AS (
  SELECT
    *,
    CASE
      WHEN strength_percentile >= 75 THEN 'High'
      WHEN strength_percentile >= 33 THEN 'Medium'
      ELSE                                'Low'
    END AS strength_class
  FROM event_with_percentile
)

SELECT
  event_id,
  age_group,
  season,
  ROUND(avg_team_rating, 2)     AS avg_team_rating,
  strength_class,
  strength_percentile,
  ROUND(competitive_spread, 2)  AS competitive_spread,
  ROUND(upset_rate, 4)          AS upset_rate,
  team_count,
  NOW()                         AS computed_at
FROM event_classified;
```

**Runtime estimate:** On a dataset of 500K games across 3 seasons, expect 2–8 minutes. The LEFT JOIN on `rankings` is the most expensive step. If the query runs over 15 minutes, add a filter to `game_context` limiting to the last 2 seasons: `AND e.season IN ('2024-2025', '2025-2026')`.

**Assumption on ratings:** This computation uses current `rankings.rating` as a proxy for team rating at time of game (not a historical rating at game date). This is standard for Phase 1; historical ratings require a separate `rating_history` table. Note this assumption in any published documentation of the Event Strength methodology.

---

## Section 4: Post-Computation Validation Queries

Run all 4 immediately after the INSERT completes. Address any failures before marking Phase 1 complete.

**V1 — Row count by season (confirm data was inserted)**

```sql
SELECT season, COUNT(*) AS event_count, ROUND(AVG(avg_team_rating), 1) AS avg_strength_rating
FROM event_strength
GROUP BY season
ORDER BY season;
```

Expected: at least 50 events for '2025-2026'. If result is 0 rows total, the INSERT failed silently — check for constraint violations or schema mismatches.

**V2 — Strength class distribution (algorithm sanity check)**

```sql
SELECT
  strength_class,
  COUNT(*) AS event_count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 1) AS pct_of_total,
  ROUND(AVG(avg_team_rating), 1) AS avg_rating_in_class
FROM event_strength
WHERE season = '2025-2026'
GROUP BY strength_class
ORDER BY avg_rating_in_class DESC;
```

Expected distribution (from PERCENT_RANK cutoffs of 75/33):
- High: ~25% of events
- Medium: ~42% of events
- Low: ~33% of events

A large skew (e.g., 60% High or 5% Low) indicates the percentile computation is off — check that `PERCENT_RANK()` PARTITION BY is including all events in the same (age_group, season) window.

**V3 — Null metric check (data completeness)**

```sql
SELECT COUNT(*) AS rows_with_null_metrics
FROM event_strength
WHERE avg_team_rating IS NULL
   OR strength_class IS NULL
   OR strength_percentile IS NULL
   OR team_count IS NULL;
```

Expected: 0. Any non-zero result means some rows were inserted with incomplete data. Investigate the specific events:
```sql
SELECT event_id, age_group, season, avg_team_rating, strength_class, team_count
FROM event_strength
WHERE avg_team_rating IS NULL OR strength_class IS NULL
LIMIT 20;
```

**V4 — DSS demo query pre-check (High-strength U13G event)**

This is the exact query from the DSS Data Readiness Validation Plan Section 2B. Running it now confirms whether the demo's Event Strength card will have data before May 14.

```sql
SELECT e.event_name, es.avg_team_rating, es.strength_class, es.strength_percentile,
       es.competitive_spread, es.upset_rate, es.team_count
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE es.age_group = 'U13G'
  AND es.strength_class = 'High'
  AND es.team_count >= 8
  AND e.season >= '2025-2026'
  AND e.event_name NOT ILIKE '%friendly%'
ORDER BY es.strength_percentile DESC
LIMIT 10;
```

Expected: at least 1 row, ideally 3–5 well-known High-strength U13G events (ECNL, GA, major invitationals).

**If this returns 0 rows:** Flag to SENTINEL immediately. The DSS demo's Event Strength card requires a High-strength U13G event to feature. Zero rows means either:
1. No U13G events in the DB have sufficient team count (≥8) and strength_class = 'High' — check if the `age_group` field on `events` uses 'U13G' or 'U13'/'F' separately.
2. The computation ran but classified no U13G events as High — check the percentile distribution from V2 filtered to age_group = 'U13G'.

**Select the demo event:** From the V4 output, identify the best candidate:
- Must be recognizable to NJ/NE club directors (ECNL or GA event preferred)
- Must have clean metrics (all fields non-null)
- Prefer a named showcase or qualifying event over a generic league game set

Record the selected event name here once confirmed: ___________________________

---

## Section 5: Rollback

Phase 1 modifies only the `event_strength` table and (if Section 1 ran) the `events.event_tier` column. Both are reversible.

**Clear event_strength data:**
```sql
TRUNCATE event_strength;
-- OR drop the table entirely:
DROP TABLE IF EXISTS event_strength;
```

**Revert event_tier remediation (if Section 1 ran and caused problems):**

The remediation SQL assigns tiers only to events where `event_tier IS NULL`. It cannot overwrite existing tier assignments. To revert:
```sql
-- Remove usl_academy assignments from remediation
UPDATE events SET event_tier = NULL
WHERE event_tier = 'usl_academy'
  AND (event_name ILIKE '%USL Academy%' OR event_name ILIKE '%USLA%');

-- Remove edp assignments from remediation
UPDATE events SET event_tier = NULL
WHERE event_tier = 'edp'
  AND event_name ILIKE '%EDP%';

-- And so on for elite_64, nal, state, local
-- Use the same WHERE conditions from Section 1 to identify remediation-assigned rows
```

Note: the `local` fallback (Step 6) cannot be cleanly reverted because it catches all remaining nulls — any event assigned `local` may have legitimately had a null tier. Revert the specific named circuits first; `local`-tagged events can be reset to null if needed:
```sql
UPDATE events SET event_tier = NULL WHERE event_tier = 'local';
```

---

## Ordering Reference

```
April 28 (required first):
  GA ASPIRE UPDATE → ranking recompute
  ↓
  Gate A passes (ASPIRE events show event_tier = 'ga_aspire', not 'ga')
  ↓
April 28 or later (same session or next session):
  Event Strength Phase 1: Section 0 gates → Section 1 remediation (if needed) 
  → Section 2 schema → Section 3 computation → Section 4 validation
  ↓
May 14:
  DSS Data Readiness Validation Plan Section 2B — confirm High-strength U13G event
```

---

## References

- [[Event Strength Ratings — Implementation Plan]] — design context and Phase 1 specification
- [[Event Strength — League Tier Coverage Audit]] — source for Section 1 remediation SQL and the 6 unconfirmed leagues
- [[DSS Data Readiness Validation Plan — May 14-15]] — Section 2B (the V4 pre-check query)
- [[Event Strength Post-Compute Validation — 2026-04-24]] — additional post-Phase-1 validation (named event spot check)
- `02 - Tiger Tournaments/Projects/Reports/ga-aspire-tier-fix-2026-04-24.md` — Gate A dependency
