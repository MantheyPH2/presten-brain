---
title: Event Strength Post-Compute Validation — 2026-04-24
aliases: [Event Strength Validation, Post-Compute Sanity Check]
tags: [rankings, events, strength, validation, evo-draw, dss]
created: 2026-04-24
updated: 2026-04-24
author: ELO Agent
status: ready
task: task-2026-04-24-event-strength-post-compute-validation
use_before: "2026-05-09 (demo-ready deadline)"
---

# Event Strength Post-Compute Validation Spec

> Run this after Phase 1 and Phase 2 of the Event Strength implementation (`Rankings/Event Strength Ratings — Implementation Plan.md`) complete. Takes ~30 minutes. Confirms output is demo-ready before frontend work begins.

---

## Section 1: Expected Distribution — What Correct Output Looks Like

### Strength Classification Thresholds (from implementation plan)

The `compute-event-strength.js` script classifies events based on `strength_percentile`:
- **High:** percentile ≥ 80 (top 20% of events by avg team rating in that age group)
- **Average:** percentile 40–79 (middle 40%)
- **Low:** percentile < 40 (bottom 40%)

By construction, approximately 20% of events should be High, 40% Average, 40% Low — but events are distributed with long tails (a small number of elite tournaments vs many smaller local events), so in practice:
- **High:** 5–15% of all classified events
- **Average:** 35–55% of classified events
- **Low:** 35–55% of classified events

The label is relative (percentile-based), so the exact split depends on the real rating distribution. A 20%/40%/40% split would indicate no skew.

### Ground Truth: Expected Named Event Classifications

| Event Name (approximate) | Expected Strength | Why |
|---|---|---|
| ECNL Girls National Event (any) | High | National showcase, top 1% competition |
| ECNL Boys National Event (any) | High | National showcase, top 1% competition |
| GA National League (Girls or Boys) | High | Top college-prep circuit nationally |
| MLS NEXT Fest / MLS NEXT National | High | Top MLS academy teams, elite field |
| Jefferson Cup (Virginia, open age groups) | High | Named invitational, historically strong field |
| Surf Cup (San Diego) | High | Top-tier invitational, national draw |
| ECNL Regional (non-showcase) | Average | Competitive but regional scope |
| GA Regional Event | Average | Competitive regional play |
| State Cup Elite Division | Average | Competitive within state, not national |
| NPL / ECNL RL league play | Average | Competitive tier 2 league |
| State Cup lower divisions | Low | Open entry, mixed quality |
| Local invitational (open entry) | Low | No field quality gate |
| Recreational league games | Low | Non-competitive |

These are the sanity check anchors. If an ECNL National event appears as Low, or a recreational league appears as High, the algorithm has a data or logic problem.

---

## Section 2: Sanity Check Queries

Run these immediately after Phase 2 (`compute-event-strength.js`) completes.

### Query A: Confirm table is populated with correct distribution

```sql
SELECT
  COUNT(*) AS total_classified,
  MIN(season) AS earliest_season,
  MAX(season) AS latest_season,
  COUNT(CASE WHEN strength_label = 'High' THEN 1 END) AS high_count,
  COUNT(CASE WHEN strength_label = 'Average' THEN 1 END) AS average_count,
  COUNT(CASE WHEN strength_label = 'Low' THEN 1 END) AS low_count,
  COUNT(CASE WHEN strength_label = 'Insufficient Data' THEN 1 END) AS insufficient_count,
  ROUND(COUNT(CASE WHEN strength_label = 'High' THEN 1 END) * 100.0 / NULLIF(COUNT(*), 0), 1) AS high_pct,
  ROUND(COUNT(CASE WHEN strength_label = 'Average' THEN 1 END) * 100.0 / NULLIF(COUNT(*), 0), 1) AS avg_pct,
  ROUND(COUNT(CASE WHEN strength_label = 'Low' THEN 1 END) * 100.0 / NULLIF(COUNT(*), 0), 1) AS low_pct
FROM event_strength
WHERE season = '2025-2026';
```

**Expected output:**
- `total_classified` ≥ 1,000 (the implementation plan expects 1,000+ rows for 2025-2026 season)
- `high_pct` between 5% and 15%
- `avg_pct` between 35% and 55%
- `low_pct` between 35% and 55%
- `insufficient_count` may be significant — this is expected for small local events with <4 teams

**Red flags:**
- `total_classified` < 500 — computation script may have failed or season filter is wrong
- `high_pct` > 25% — High threshold is set too low; check if percentile cutoff at 80 is working
- `high_pct` < 3% — High threshold may be too strict, or avg_team_rating computation is wrong
- `high_count` = 0 — critical failure, check computation script output

### Query B: Named event spot check

```sql
SELECT
  e.event_name,
  es.strength_label,
  ROUND(es.avg_team_rating, 0) AS avg_rating,
  ROUND(es.strength_percentile, 1) AS percentile,
  ROUND(es.rating_spread, 0) AS spread,
  ROUND(es.upset_rate * 100, 1) AS upset_pct,
  es.team_count,
  es.spread_label,
  es.upset_label
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE (
  e.event_name ILIKE '%ECNL%'
  OR e.event_name ILIKE '%GA National%'
  OR e.event_name ILIKE '%Surf Cup%'
  OR e.event_name ILIKE '%Jefferson%'
  OR e.event_name ILIKE '%MLS NEXT Fest%'
  OR e.event_name ILIKE '%MLS NEXT National%'
)
AND es.season = '2025-2026'
ORDER BY es.strength_percentile DESC
LIMIT 30;
```

**If no `events` table (fallback using games.event_name):**
```sql
SELECT
  es.event_name,
  es.strength_label,
  ROUND(es.avg_team_rating, 0) AS avg_rating,
  ROUND(es.strength_percentile, 1) AS percentile,
  ROUND(es.rating_spread, 0) AS spread,
  ROUND(es.upset_rate * 100, 1) AS upset_pct,
  es.team_count,
  es.spread_label,
  es.upset_label
FROM event_strength es
WHERE (
  es.event_name ILIKE '%ECNL%'
  OR es.event_name ILIKE '%GA National%'
  OR es.event_name ILIKE '%Surf Cup%'
  OR es.event_name ILIKE '%Jefferson%'
)
AND es.season = '2025-2026'
ORDER BY es.strength_percentile DESC
LIMIT 30;
```

**Compare results against Section 1 ground truth table.** Any ECNL National event classified as Low is a red flag requiring investigation before going live.

### Query C: Strength distribution by event tier

```sql
SELECT
  e.event_tier,
  es.strength_label,
  COUNT(*) AS event_count,
  ROUND(AVG(es.avg_team_rating), 0) AS avg_rating_in_cell
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE es.season = '2025-2026'
GROUP BY e.event_tier, es.strength_label
ORDER BY e.event_tier, es.strength_label;
```

**Expected cross-tab:**
- `ga` and `mlsnext` tier: ≥70% High or Average; <10% Low
- `ecnl` and `ecnl_regional` tier: ≥60% High or Average
- `state` and `local` tier: ≥70% Low or Average; <10% High

**Red flags:**
- Any `ga`-tier event classified as Low — unless team_count was <4, this is wrong
- Any `local`-tier event classified as High — check team counts and ratings; likely data quality issue

### Query D: Minimum viable data quality check

```sql
SELECT
  es.event_name,
  es.team_count,
  es.strength_label,
  ROUND(es.avg_team_rating, 0) AS avg_rating
FROM event_strength es
WHERE es.team_count < 4
  AND es.season = '2025-2026'
ORDER BY es.team_count ASC
LIMIT 20;
```

**Expected:** 0 rows. The `HAVING COUNT(*) >= 4` clause in the computation script should prevent events with fewer than 4 teams from being inserted. Any rows here mean the filter is not working correctly and those events should have `strength_label = 'Insufficient Data'` instead.

---

## Section 3: Pass/Fail Decision Criteria

After running Queries A–D, apply these criteria to decide if Event Strength is demo-ready:

| Check | Pass | Fail |
|---|---|---|
| Table populated | ≥ 1,000 classified rows for 2025-2026 | < 500 rows |
| High event count | 5–15% of all classified events | < 3% or > 25% |
| ECNL event classification | ≥ 80% of ECNL events are High or Average | Any ECNL National event classified as Low |
| GA tier classification | ≥ 90% of `ga`-tier events are High or Average | > 10% of `ga` events classified as Low |
| State/local tier | ≥ 70% of `state` and `local` events are Low or Average | > 20% of state events classified as High |
| Minimum team count filter | 0 events with team_count < 4 are classified | Any events with team_count < 4 have a strength label |
| Avg rating spread | High events avg rating ≥ 200 pts above Low events avg rating | High events avg rating ≤ Low events avg rating |

**All 7 checks must pass before frontend work begins.** Any single failure means investigate before proceeding.

---

## Section 4: Demo Event Selection Query

The DSS Demo Spec requires one specific High-strength event to feature. Run this query after Phase 2 completes and validation passes:

```sql
SELECT
  es.event_name,
  ROUND(es.avg_team_rating, 0) AS avg_rating,
  es.strength_label,
  ROUND(es.strength_percentile, 1) AS percentile,
  ROUND(es.rating_spread, 0) AS spread,
  ROUND(es.upset_rate * 100, 1) AS upset_pct,
  es.team_count,
  es.spread_label,
  es.upset_label
FROM event_strength es
WHERE es.age_group = 'U13'
  AND es.gender = 'F'
  AND es.strength_label = 'High'
  AND es.team_count >= 8
  AND es.season = '2025-2026'
  AND es.event_name NOT ILIKE '%friendly%'
  AND es.upset_rate IS NOT NULL
  AND es.rating_spread IS NOT NULL
ORDER BY es.strength_percentile DESC
LIMIT 10;
```

**Selection criteria (in priority order):**
1. Event name is recognizable to NJ/NE club directors — prefer ECNL or GA events with well-known names over generic invitationals
2. All four metrics are non-null: `avg_team_rating`, `rating_spread`, `upset_rate`, `team_count`
3. `team_count` ≥ 12 (more data = more credible metric display)
4. Event from current season (2025-2026) — shows live system capability, not historical

**Record the selected event name here once confirmed:**

```
Demo event (U13G): ____________________________
avg_team_rating: ________  percentile: ________
spread: ________  upset_rate: ________  team_count: ________
```

This must be confirmed before the DSS Data Readiness Validation Plan runs on May 14–15.

---

## Section 5: Red Flag Protocol

### Scenario A: Too many High events (> 25%)

**Likely cause:** The `strength_percentile >= 80` threshold is being applied correctly but the underlying `avg_team_rating` values are inflated — possibly because unranked teams are excluded from the average, leaving only the highest-rated teams in the avg_team_rating calculation, which pushes most events above the p80 cutoff.

**Diagnostic query:**
```sql
-- Check if avg_team_rating is suspiciously uniform across events
SELECT
  strength_label,
  ROUND(MIN(avg_team_rating), 0) AS min_avg,
  ROUND(MAX(avg_team_rating), 0) AS max_avg,
  ROUND(AVG(avg_team_rating), 0) AS mean_avg,
  ROUND(STDDEV(avg_team_rating), 0) AS std_avg,
  COUNT(*) AS event_count
FROM event_strength
WHERE season = '2025-2026'
GROUP BY strength_label
ORDER BY mean_avg DESC;
```

If `max_avg - min_avg` for all classified events is < 200 points, the distribution is too compressed. The PERCENT_RANK window function should naturally spread percentiles — if it isn't, check that `PARTITION BY age_group, gender` is working correctly. A missing partition would give all events the same percentile reference pool.

**Recommended fix:** Confirm the CTE partitioning in `compute-event-strength.js`. The `PERCENT_RANK() OVER (PARTITION BY age_group, gender ORDER BY avg_team_rating)` must partition correctly. If fixed, re-run the computation script.

### Scenario B: ECNL National event classified as Medium or Low

**Likely cause:** That specific event has low `team_count` in the DB (coverage gap) or unusually low `avg_team_rating` because many teams competing in it are new (no established ratings → excluded from avg, leaving only a few teams with ratings that may not reflect the true field quality).

**Diagnostic query:**
```sql
-- Investigate a specific event that should be High but isn't
SELECT
  es.event_name, es.strength_label, es.strength_percentile,
  es.team_count, ROUND(es.avg_team_rating, 0) AS avg_rating,
  es.min_rating, es.max_rating
FROM event_strength es
WHERE es.event_name ILIKE '%[EVENT NAME]%'
  AND es.season = '2025-2026';
```

If `team_count` is lower than expected (e.g., 6 for an ECNL national event that should have 32+ teams), the coverage gap is the cause — not algorithm failure. Flag for FORGE: check that all participating teams have game records in the DB and that the event is tagged with `is_hidden = false`.

**Recommended fix (if coverage gap):** Ensure event ingestion is complete for that event before the demo. Not a computation logic issue.

### Scenario C: avg_team_rating for High events is lower than expected (< 1,100)

**Likely cause:** Unranked or newly-formed teams are included in the avg_team_rating calculation despite having no stable ratings. The computation script joins to the `rankings` table — if newly ingested teams have default provisional Elo (~1000) rather than a NULL rating, they are included in the average and pull it down.

**Diagnostic query:**
```sql
-- Check for events where many teams have ratings near the provisional default
SELECT
  es.event_name,
  ROUND(es.avg_team_rating, 0) AS avg_rating,
  es.min_rating,
  es.team_count,
  es.strength_label
FROM event_strength es
WHERE es.avg_team_rating < 1100
  AND es.strength_label IN ('High', 'Average')
  AND es.season = '2025-2026'
ORDER BY es.avg_team_rating ASC
LIMIT 10;
```

If events have `min_rating` near 1000 and these are pulling the average down, the fix is to filter out provisional ratings (teams with <5 completed games) from the avg_team_rating CTE in `compute-event-strength.js`. This requires a schema join to a game count per team, or a minimum rating threshold (e.g., `WHERE r.rating > 1050`).

**Recommended fix:** Add `AND r.rating > 1050` to the `event_teams` CTE WHERE clause. This excludes provisional-default teams from the avg_team_rating without excluding them from the event entirely. Re-run after the fix.

---

## References

- [[Event Strength Ratings — Implementation Plan]] — what this validates (Phase 1 and Phase 2)
- [[Event Strength Ratings Design]] — threshold rationale (strength_label cutoffs)
- [[DSS Demo Spec — May 2026]] — demo event requirements
- [[DSS Data Readiness Validation Plan — May 14-15]] — uses event strength in query 2B; demo event must be confirmed before May 14
