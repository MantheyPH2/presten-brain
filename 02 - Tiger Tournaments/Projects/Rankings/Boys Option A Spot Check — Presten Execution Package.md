---
title: Boys Option A Spot Check — Presten Execution Package
type: execution-package
author: ELO
date: 2026-04-25
status: ready
related: "[[Boys Calibration Status Summary — April 2026]]", "[[Boys Calibration Option A — Interpretation Guide]]", "[[Boys Calibration Gap Analysis — April 2026]]"
tags: [boys, calibration, dss, execution, spot-check, evo-draw]
---

# Boys Option A Spot Check — Presten Execution Package

**When to run:** April 29 through May 5 — after April 28 GA ASPIRE fix + full recompute confirmed complete.

---

## Pre-Run Gate

```
[ ] April 28 GA ASPIRE UPDATE has run (events re-tagged from `ga` → `ga_aspire`)
[ ] Full ranking recompute completed after April 28 UPDATE
[ ] April 29 post-fix verification passed (Event Strength Gate A confirmed)
[ ] psql access to youth_soccer DB is confirmed
→ All four checked: proceed.
→ Any unchecked: stop. Boys ratings may still reflect GA ASPIRE contamination. Run after April 29 verification.
```

---

## Query 1 — Boys vs. Girls Average Rating by Age Group

**Purpose:** Confirm Boys and Girls ratings are not anomalously divergent. Boys and Girls are calibrated with the same league structure (different cal values for GA, but same Glicko parameters). A divergence > 100 points in the same age group is a signal of miscalibration or coverage artifact.

```sql
SELECT
  r.age_group,
  r.gender,
  COUNT(*) AS team_count,
  ROUND(AVG(r.rating)::numeric, 1) AS avg_rating,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY r.rating) AS median_rating
FROM rankings r
WHERE r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND r.gender IN ('M', 'F')
  AND r.rank IS NOT NULL
GROUP BY r.age_group, r.gender
ORDER BY r.age_group, r.gender;
```

**Save command:**
```
\COPY (SELECT r.age_group, r.gender, COUNT(*) AS team_count, ROUND(AVG(r.rating)::numeric, 1) AS avg_rating, PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY r.rating) AS median_rating FROM rankings r WHERE r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17') AND r.gender IN ('M', 'F') AND r.rank IS NOT NULL GROUP BY r.age_group, r.gender ORDER BY r.age_group, r.gender) TO 'reports/boys-option-a-query1-[date].txt' WITH CSV HEADER
```

**Replace `[date]` with today's date in YYYY-MM-DD format.**

**Expected output:** 10 rows (5 age groups × 2 genders). Columns: `age_group, gender, team_count, avg_rating, median_rating`.

**PASS:** For every age group, the absolute difference between Boys avg_rating and Girls avg_rating is ≤ 100 points.

**FAIL:** Any age group where |Boys avg_rating − Girls avg_rating| > 100 points → flag to SENTINEL. Boys club rankings cannot be shown at DSS for that age group.

**Save output file to:** `02 - Tiger Tournaments/Projects/Reports/boys-option-a-query1-[date].md`

---

## Query 2 — Boys GA Game Volume Validation

**Purpose:** Confirm Boys GA coverage is sufficient for the cal=100 calibration value to be meaningfully applied. Below 500 games, Boys GA calibration is underpowered and may drift unpredictably.

```sql
SELECT
  r.age_group,
  COUNT(DISTINCT g.id) AS boys_ga_games,
  COUNT(DISTINCT r.team_id) AS teams_with_ga_games
FROM rankings r
JOIN games g ON (g.home_team_id = r.team_id OR g.away_team_id = r.team_id)
JOIN events e ON e.id = g.event_id
WHERE r.gender = 'M'
  AND r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND e.event_tier IN ('ga', 'ga_aspire')
  AND r.rank IS NOT NULL
GROUP BY r.age_group
ORDER BY r.age_group;
```

**Save command:**
```
\COPY (SELECT r.age_group, COUNT(DISTINCT g.id) AS boys_ga_games, COUNT(DISTINCT r.team_id) AS teams_with_ga_games FROM rankings r JOIN games g ON (g.home_team_id = r.team_id OR g.away_team_id = r.team_id) JOIN events e ON e.id = g.event_id WHERE r.gender = 'M' AND r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17') AND e.event_tier IN ('ga', 'ga_aspire') AND r.rank IS NOT NULL GROUP BY r.age_group ORDER BY r.age_group) TO 'reports/boys-option-a-query2-[date].txt' WITH CSV HEADER
```

**Replace `[date]` with today's date in YYYY-MM-DD format.**

**Expected output:** 5 rows (one per age group). Columns: `age_group, boys_ga_games, teams_with_ga_games`.

**PASS:** All U13B–U17B age groups have ≥ 500 Boys GA games.

**FAIL:** Any age group with < 500 Boys GA games → flag to SENTINEL. Do not claim Boys GA calibration is validated for that age group at DSS. Note in demo script if Boys club rankings are shown: Boys GA coverage is thin for [age group].

**Save output file to:** `02 - Tiger Tournaments/Projects/Reports/boys-option-a-query2-[date].md`

---

## Query 3 — MLS NEXT Boys Rating Distribution

**Purpose:** MLS NEXT is the best-validated Boys calibration anchor (cal=160 Homegrown, cal=135 Academy, ~5,800 cross-league validation games). If the MLS NEXT Boys rating distribution is outside the expected range, Boys calibration has a systematic problem that must be investigated before DSS.

```sql
SELECT
  r.age_group,
  COUNT(*) AS mlsnext_team_count,
  ROUND(AVG(r.rating)::numeric, 1) AS avg_rating,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY r.rating) AS median_rating,
  PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY r.rating) AS top_10pct_rating,
  PERCENTILE_CONT(0.1) WITHIN GROUP (ORDER BY r.rating) AS bottom_10pct_rating,
  MAX(r.rating) AS max_rating
FROM rankings r
JOIN teams t ON t.id = r.team_id
JOIN games g ON (g.home_team_id = t.id OR g.away_team_id = t.id)
JOIN events e ON e.id = g.event_id
WHERE r.gender = 'M'
  AND r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND e.event_tier IN ('mls_next_homegrown', 'mls_next_academy', 'mls_next')
  AND r.rank IS NOT NULL
GROUP BY r.age_group
ORDER BY r.age_group;
```

**Save command:**
```
\COPY (SELECT r.age_group, COUNT(*) AS mlsnext_team_count, ROUND(AVG(r.rating)::numeric, 1) AS avg_rating, PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY r.rating) AS median_rating, PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY r.rating) AS top_10pct_rating, PERCENTILE_CONT(0.1) WITHIN GROUP (ORDER BY r.rating) AS bottom_10pct_rating, MAX(r.rating) AS max_rating FROM rankings r JOIN teams t ON t.id = r.team_id JOIN games g ON (g.home_team_id = t.id OR g.away_team_id = t.id) JOIN events e ON e.id = g.event_id WHERE r.gender = 'M' AND r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17') AND e.event_tier IN ('mls_next_homegrown', 'mls_next_academy', 'mls_next') AND r.rank IS NOT NULL GROUP BY r.age_group ORDER BY r.age_group) TO 'reports/boys-option-a-query3-[date].txt' WITH CSV HEADER
```

**Replace `[date]` with today's date in YYYY-MM-DD format.**

**Expected output:** 5 rows (one per age group). Columns: `age_group, mlsnext_team_count, avg_rating, median_rating, top_10pct_rating, bottom_10pct_rating, max_rating`.

**PASS:** For each age group with ≥ 10 MLS NEXT teams:
- Median rating: 1,400–1,600
- Top 10th percentile: ≥ 1,700
- Bottom 10th percentile: ≤ 1,300

**FAIL:** Any age group where median or top/bottom decile is outside the expected range by > 100 pts → contact ELO immediately. ELO will assess whether Boys MLS NEXT calibration needs re-calibration before DSS.

**Save output file to:** `02 - Tiger Tournaments/Projects/Reports/boys-option-a-query3-[date].md`

---

## Decision Table

| Result | Action |
|--------|--------|
| All 3 PASS | Boys club rankings are demo-safe. Include Boys in Club Rankings implementation plan. Note in demo script: Boys rankings available for U13B–U17B. |
| Query 1 FAIL on any age group (>100 pt divergence) | Boys club rankings excluded from DSS entirely. Girls-only club rankings. Notify SENTINEL immediately. Do not improvise Boys rankings at DSS. |
| Query 2 FAIL on any age group (<500 Boys GA games) | Boys club rankings may still show at DSS, but do not cite Boys GA calibration as validated for that age group. Add to demo script: "Boys GA coverage is thinner than Girls — we're adding data sources for the 2026-27 season." |
| Query 3 FAIL (MLS NEXT distribution out of range) | Contact ELO immediately with the specific age group and distribution numbers. ELO will assess within one session whether Boys MLS NEXT cal values need adjustment before DSS. Do not mark Boys as demo-safe until ELO clears it. |
| All 3 PASS but Boys team counts < 50 for any age group | Boys club rankings may show thin results for that age group. Note in demo script. Do not present Boys U13/U14 club rankings as comprehensive if team count < 50. |

---

## Post-Run: Send Results to ELO

After completing all three queries, send the three output files to ELO. ELO will:

1. Classify each query result per the PASS/FAIL thresholds above.
2. Update the DSS Feature Readiness Tracker — Club Rankings row, Boys cell.
3. File an updated Boys calibration assessment in the next briefing to SENTINEL.
4. Confirm Boys club rankings go/no-go status by May 9.

**Run window:** April 29 (after GA ASPIRE recompute confirmed) through May 5.

**Deadline:** May 5 at the latest. ELO needs 3 days to analyze results and update the tracker before May 9 go/no-go.

---

## References

- [[Boys Calibration Status Summary — April 2026]] — rationale and overall risk assessment
- [[Boys Calibration Option A — Interpretation Guide]] — detailed threshold derivation and interpretation
- [[Boys Calibration Gap Analysis — April 2026]] — full gap analysis with additional queries
- [[Boys GA ASPIRE Calibration Decision — April 2026]] — Option A decision (cal=100)
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Club Rankings readiness row

*ELO — 2026-04-25*
