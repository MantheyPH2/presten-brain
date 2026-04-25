---
title: Pre-April-28 Baseline Snapshot — Presten Execution Package
type: execution-package
author: ELO
date: 2026-04-25
status: ready
related: "[[Pre-April-28 Baseline Snapshot Spec]]", "[[Post-April-28 Rating Shift Analysis Spec]]"
tags: [calibration, ga-aspire, baseline, april-28, execution, evo-draw]
---

# Pre-April-28 Baseline Snapshot — Presten Execution Package

**When to run:** April 26 or April 27 — before the April 28 session begins.

---

## Pre-Run Gate

```
[ ] Today is April 26 or April 27
[ ] April 28 session has NOT yet run (fix has not been applied)
[ ] psql access to youth_soccer DB is confirmed
→ All three checked: proceed.
→ Any unchecked: stop.
```

---

## Query 1 — U13G Top-50

```sql
SELECT t.name, r.rating, r.rank, t.source_org_id
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13G'
  AND r.rank <= 50
ORDER BY r.rank;
```

**Save command:**
```
\COPY (SELECT t.name, r.rating, r.rank, t.source_org_id FROM rankings r JOIN teams t ON t.id = r.team_id WHERE r.age_group = 'U13G' AND r.rank <= 50 ORDER BY r.rank) TO 'reports/pre-april28-u13g-top50.txt' WITH CSV HEADER
```

**Expected output:** Exactly 50 rows. Columns: `name, rating, rank, source_org_id`.

---

## Query 2 — GA ASPIRE Teams

```sql
SELECT t.name, r.rating, r.rank, r.age_group, r.gender,
       e.event_tier AS current_tier,
       COUNT(g.id) AS games_in_tier
FROM rankings r
JOIN teams t ON t.id = r.team_id
JOIN games g ON (g.home_team_id = t.id OR g.away_team_id = t.id)
JOIN events e ON e.id = g.event_id
WHERE e.event_tier IN ('ga', 'ga_aspire')
GROUP BY t.name, r.rating, r.rank, r.age_group, r.gender, e.event_tier
ORDER BY r.age_group, r.gender, r.rank;
```

**Save command:**
```
\COPY (SELECT t.name, r.rating, r.rank, r.age_group, r.gender, e.event_tier AS current_tier, COUNT(g.id) AS games_in_tier FROM rankings r JOIN teams t ON t.id = r.team_id JOIN games g ON (g.home_team_id = t.id OR g.away_team_id = t.id) JOIN events e ON e.id = g.event_id WHERE e.event_tier IN ('ga', 'ga_aspire') GROUP BY t.name, r.rating, r.rank, r.age_group, r.gender, e.event_tier ORDER BY r.age_group, r.gender, r.rank) TO 'reports/pre-april28-ga-aspire-teams.txt' WITH CSV HEADER
```

**Expected output:** Several hundred rows. Columns: `name, rating, rank, age_group, gender, current_tier, games_in_tier`. Most rows will have `current_tier = 'ga'` (the misclassified ones). A smaller set will have `current_tier = 'ga_aspire'` (already correctly tagged).

---

## Query 3 — Overall Distribution

```sql
SELECT age_group, gender,
       COUNT(*) as team_count,
       ROUND(AVG(rating)::numeric, 1) as avg_rating,
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating) as median_rating,
       MAX(rating) as max_rating,
       MIN(rating) as min_rating
FROM rankings
WHERE rank IS NOT NULL
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

**Save command:**
```
\COPY (SELECT age_group, gender, COUNT(*) as team_count, ROUND(AVG(rating)::numeric, 1) as avg_rating, PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating) as median_rating, MAX(rating) as max_rating, MIN(rating) as min_rating FROM rankings WHERE rank IS NOT NULL GROUP BY age_group, gender ORDER BY age_group, gender) TO 'reports/pre-april28-distribution.txt' WITH CSV HEADER
```

**Expected output:** ~20–30 rows (one per age group × gender combination). Columns: `age_group, gender, team_count, avg_rating, median_rating, max_rating, min_rating`.

---

## Post-Run Confirmation

```
[ ] File 1 saved: reports/pre-april28-u13g-top50.txt — 50 rows
[ ] File 2 saved: reports/pre-april28-ga-aspire-teams.txt — several hundred rows
[ ] File 3 saved: reports/pre-april28-distribution.txt — ~20–30 rows
→ All three files saved: baseline snapshot complete. April 28 session can proceed.
→ Missing files: run the missing query before April 28.
```

---

**Done.** Baseline captured. ELO uses these files for April 29 rating shift analysis.
