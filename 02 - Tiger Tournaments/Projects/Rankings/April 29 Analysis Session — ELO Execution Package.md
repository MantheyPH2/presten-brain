---
title: April 29 Analysis Session — ELO Execution Package
type: execution-package
author: ELO
date: 2026-04-25
status: ready — execute April 29 after April 28 recompute confirmed
related: "[[April 29 Post-Fix Decision Tree]]", "[[Post-April-28 Rating Shift Analysis Spec]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]", "[[Rank Bands — Post-April-28 Validation Spec]]", "[[Post-Fix Calibration Stability Monitoring Spec]]", "[[Boys Option A Spot Check — Presten Execution Package]]"
tags: [april-29, ga-aspire, calibration, execution, post-fix, dss, evo-draw]
---

# April 29 Analysis Session — ELO Execution Package

> **One document. One session.** This package consolidates five separate post-fix specs into a single ordered checklist. On April 29, open a psql terminal, follow Phases 1–6 in sequence, and file the briefing when done. Do not open any other spec document during the session — everything you need is here.

---

## Pre-Conditions

Before running any queries, confirm all three:

```
[ ] April 28 GA ASPIRE UPDATE confirmed run (events re-tagged from `ga` → `ga_aspire`)
[ ] Full ranking recompute completed after April 28 UPDATE
[ ] Pre-April-28 Baseline Snapshot file path: reports/pre-april28-*.txt (confirm file exists)
```

**If any pre-condition is unchecked: do NOT proceed. Contact SENTINEL.** The analysis is meaningless without a completed recompute to compare against.

---

## Phase 1 — Decision Tree Gate G1: Did the Fix Run?

**Source:** `Rankings/April 29 Post-Fix Decision Tree.md`, Gates G1 and G3

Run this query to confirm the UPDATE ran and rankings are fresh:

```sql
-- Confirm GA ASPIRE events are re-tagged
SELECT COUNT(*) AS ga_aspire_event_count
FROM events
WHERE event_tier = 'ga_aspire';
-- Expected: > 0. If 0, the UPDATE did not run. STOP.

-- Confirm rankings recompute ran after April 28
SELECT MAX(updated_at) AS latest_rankings_update
FROM rankings
WHERE age_group = 'U13G';
-- Expected: timestamp on or after April 28 session start. If earlier, recompute incomplete. STOP.
```

Record results:

| Check | Pass Condition | Result | Pass/Fail |
|-------|----------------|--------|-----------|
| `ga_aspire` event count | > 0 rows | | |
| Rankings `updated_at` (U13G) | ≥ April 28 session start | | |

**If any G1 check fails:** STOP. File briefing: "G1 FAIL — April 28 fix did not complete. Event Strength Phase 1 HELD. SENTINEL notified." Do not proceed to Phase 2.

**If both pass:** continue to Phase 2.

---

## Phase 2 — Rating Shift vs. Pre-Run Model

**Source:** `Rankings/Post-April-28 Rating Shift Analysis Spec.md`, `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md`

Run the Girls GA ASPIRE average rating query:

```sql
-- Post-fix Girls vs Boys GA ASPIRE average rating by age group
SELECT r.age_group, r.gender,
       ROUND(AVG(r.rating)::numeric, 1) AS post_fix_avg_rating,
       COUNT(*) AS team_count
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
GROUP BY r.age_group, r.gender
ORDER BY r.age_group, r.gender;
```

Compare each row against the pre-fix baseline from `reports/pre-april28-ga-aspire-teams.txt`:

| Cohort | Pre-Fix Avg (from baseline file) | Post-Fix Avg | Delta | Pre-Run Prediction | Within Range? |
|--------|----------------------------------|-------------|-------|--------------------|--------------|
| U13G (Girls) | [from baseline] | | | +5 to +40 pts | |
| U14G (Girls) | [from baseline] | | | +5 to +40 pts | |
| U15G (Girls) | [from baseline] | | | +5 to +40 pts | |
| U16G (Girls) | [from baseline] | | | +5 to +40 pts | |
| U17G (Girls) | [from baseline] | | | +2 to +40 pts | |
| U13B (Boys) | [from baseline] | | | 0 ± 5 pts | |
| U14B (Boys) | [from baseline] | | | 0 ± 5 pts | |

**Phase 2 verdict:**
- All Girls deltas positive AND Boys delta ≤ ±5 pts → **proceed to Phase 3**
- Girls deltas negative OR Boys delta > 5 pts → **flag to SENTINEL before proceeding** (G1 or G4 concern)
- Boys delta > 15 pts → **HOLD Event Strength and Rank Bands.** File G4 FAIL per Decision Tree.

Save the filled table to: `reports/post-april28-rating-shift-analysis-2026-04-29.md`

---

## Phase 3 — Rank Bands Distribution Check

**Source:** `Rankings/Rank Bands — Post-April-28 Validation Spec.md`, Queries 1–2

**Query 1 — Band Distribution by Gender:**

```sql
SELECT
  gender,
  band,
  COUNT(*) AS team_count,
  ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (PARTITION BY gender), 1) AS pct_of_gender
FROM rankings_with_bands
GROUP BY gender, band
ORDER BY gender,
  CASE band
    WHEN 'Gold'   THEN 1
    WHEN 'Silver' THEN 2
    WHEN 'Bronze' THEN 3
    WHEN 'Red'    THEN 4
    WHEN 'Blue'   THEN 5
    WHEN 'Green'  THEN 6
  END;
```

**Query 2 — Band Distribution by Age Group (Girls only):**

```sql
SELECT
  age_group,
  band,
  COUNT(*) AS team_count,
  ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (PARTITION BY age_group), 1) AS pct_of_age_group
FROM rankings_with_bands
WHERE gender = 'F'
GROUP BY age_group, band
ORDER BY age_group,
  CASE band
    WHEN 'Gold'   THEN 1
    WHEN 'Silver' THEN 2
    WHEN 'Bronze' THEN 3
    WHEN 'Red'    THEN 4
    WHEN 'Blue'   THEN 5
    WHEN 'Green'  THEN 6
  END;
```

Save query output to: `reports/rank-bands-distribution-2026-04-29.md`

**Interpretation:**
- Girls Bronze % decreases slightly, Silver % increases by ~1–3% → **expected and correct**
- Any band shifts > 5 percentage points → **review required; file note in briefing**
- Girls Gold > 5% of ranked Girls teams → **recalibration needed; contact ELO immediately**
- Boys band distribution unchanged → **expected**

One-line summary for briefing:
> "Rank bands post-fix: Girls distribution shifted [N] pts in [band]. Verdict: [no revision needed / review underway / recalibration triggered]."

---

## Phase 4 — First Calibration Stability Baseline

**Source:** `Rankings/Post-Fix Calibration Stability Monitoring Spec.md`, Queries 1–3

This is the **first baseline run only.** No pass/fail criteria apply today — record numbers so future runs can compare.

**Query 1 — Girls GA ASPIRE Average Rating Trend:**

```sql
SELECT r.age_group,
       ROUND(AVG(r.rating)::numeric, 1) AS avg_rating,
       COUNT(*) AS team_count,
       NOW()::date AS run_date
FROM rankings r
WHERE r.gender = 'F'
  AND r.team_id IN (
    SELECT DISTINCT g.home_team_id FROM games g
    JOIN events e ON e.id = g.event_id
    WHERE e.event_tier = 'ga_aspire'
    UNION
    SELECT DISTINCT g.away_team_id FROM games g
    JOIN events e ON e.id = g.event_id
    WHERE e.event_tier = 'ga_aspire'
  )
GROUP BY r.age_group
ORDER BY r.age_group;
```

**Query 2 — Daily Rating Volatility Check:**

```sql
-- Count teams with rating changes > 30 pts vs. baseline
-- NOTE: first run only — no comparison available today; record avg_rating as baseline
SELECT r.age_group, r.gender,
       COUNT(*) AS team_count,
       ROUND(AVG(r.rating)::numeric, 1) AS avg_rating,
       MAX(r.rating) AS max_rating,
       MIN(r.rating) AS min_rating
FROM rankings r
WHERE r.rank IS NOT NULL
GROUP BY r.age_group, r.gender
ORDER BY r.age_group, r.gender;
```

**Query 3 — Cross-Gender Convergence Check:**

```sql
-- Compare Girls GA ASPIRE avg rating vs Girls ECNL avg rating
SELECT
  tier_group,
  ROUND(AVG(r.rating)::numeric, 1) AS avg_rating,
  COUNT(*) AS team_count
FROM rankings r
JOIN (
  SELECT DISTINCT g.home_team_id AS team_id, 'ga_aspire' AS tier_group
  FROM games g JOIN events e ON e.id = g.event_id WHERE e.event_tier = 'ga_aspire'
  UNION
  SELECT DISTINCT g.away_team_id, 'ga_aspire'
  FROM games g JOIN events e ON e.id = g.event_id WHERE e.event_tier = 'ga_aspire'
  UNION
  SELECT DISTINCT g.home_team_id, 'ecnl'
  FROM games g JOIN events e ON e.id = g.event_id WHERE e.event_tier = 'ecnl'
  UNION
  SELECT DISTINCT g.away_team_id, 'ecnl'
  FROM games g JOIN events e ON e.id = g.event_id WHERE e.event_tier = 'ecnl'
) tiers ON tiers.team_id = r.team_id
WHERE r.gender = 'F' AND r.rank IS NOT NULL
GROUP BY tier_group;
```

Save all three outputs to: `reports/ga-aspire-stability-2026-04-29.md`

One-line summary for briefing:
> "Monitoring baseline recorded April 29. Girls GA ASPIRE avg: [N]. Daily volatility run baseline: [N] teams tracked. Girls GA–ECNL delta: [N] pts."

---

## Phase 5 — Boys Option A Results Filing (If Run Today)

**Source:** `Rankings/Boys Option A Spot Check — Presten Execution Package.md`, `Rankings/Boys Calibration Option A — Interpretation Guide.md`

**If Presten ran Boys Option A queries on April 29:**
1. Save raw results to: `reports/boys-option-a-results-2026-04-29.md`
2. Classify each query per the Decision Table in the Execution Package:

| Query | Result | PASS/FAIL/INCONCLUSIVE |
|-------|--------|----------------------|
| Q1 — Boys vs Girls avg rating (divergence ≤ 100 pts per age group) | | |
| Q2 — Boys GA game volume (≥ 500 per age group) | | |
| Q3 — MLS NEXT Boys distribution (median 1,400–1,600) | | |

3. File Boys Option A verdict in briefing: "Boys Option A: [PASS / FAIL / NOT YET RUN]."
4. Update DSS Feature Readiness Tracker — Club Rankings Boys cell.

**If Presten did NOT run Boys Option A on April 29:** Note in briefing: "Boys Option A spot check not yet run. Window open through May 5. Recommend running by May 3 to allow ELO investigation time if borderline result."

---

## Phase 6 — DSS Readiness Tracker Update

**Source:** `Product & Planning/DSS Feature Readiness Tracker — May 2026.md`

Update the following cells based on today's results:

| Feature | Cell to Update | Update Condition |
|---------|---------------|-----------------|
| Rank Bands | Calibration Ready | ✅ if Phase 3 shows no revision needed; ⚠️ if review triggered; ❌ if recalibration needed |
| Event Strength Phase 1 | Calibration Ready | ✅ if G1 passed and stability baseline filed; ⚠️ if G1 borderline |
| Club Rankings — Boys | Status | ✅ demo-safe if Boys Option A PASS; ❌ if FAIL; ⚠️ pending if not yet run |

File confirmation in briefing: "DSS Readiness Tracker updated for April 29 results."

---

## Session Completion Checklist

Before filing the April 29 briefing, confirm all are checked:

```
[ ] G1 gate confirmed (GA ASPIRE events re-tagged, recompute timestamp verified)
[ ] Phase 2 rating shift table complete and compared to Pre-Run Model
[ ] Phase 2 verdict recorded (clean / flagged)
[ ] Rank bands distribution query outputs saved to reports/
[ ] Monitoring baseline (all 3 queries) saved to reports/
[ ] Boys Option A status noted (ran / not yet run / results filed)
[ ] DSS Readiness Tracker updated
[ ] All output files saved to reports/ with 2026-04-29 suffix
[ ] Briefing filed with Phase 1–6 summaries and Decision Tree gate status
```

---

## Escalation Summary (Quick Reference)

| Condition | Action |
|-----------|--------|
| G1 FAIL (recompute not complete or GA ASPIRE not re-tagged) | STOP. Do not run any further phases. Contact SENTINEL. |
| Girls delta negative (ratings went DOWN) | G1 fail. Stop. File immediately. |
| Boys delta > 15 pts | G4 fail. Hold Event Strength and Rank Bands. File SENTINEL flag. |
| Girls Gold > 5% post-fix | Recalibration required. Contact ELO before any Rank Bands implementation. |
| Boys Option A Q1 FAIL (>100 pt divergence) | Boys club rankings excluded from DSS. Girls-only. Notify SENTINEL immediately. |

---

## References

- `Rankings/April 29 Post-Fix Decision Tree.md` — full G1–G4 gate framework and authorization block
- `Rankings/Post-April-28 Rating Shift Analysis Spec.md` — detailed shift analysis methodology
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — predicted values for Phase 2 comparison
- `Rankings/Rank Bands — Post-April-28 Validation Spec.md` — band distribution thresholds for Phase 3
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — monitoring queries for Phase 4
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — Boys queries and decision table for Phase 5
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — tracker updated in Phase 6

*ELO — 2026-04-25*
