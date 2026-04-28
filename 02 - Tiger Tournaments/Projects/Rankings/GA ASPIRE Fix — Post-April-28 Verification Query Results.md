---
title: GA ASPIRE Fix — Post-April-28 Verification Query Results
type: verification-results
author: ELO
date: 2026-04-28
status: template — fill Section 2 results during April 29 session
related: "[[ga-coverage-audit-results-2026-04-23]]", "[[Girls Calibration Status — Post-April-28 Narrative]]", "[[April 29 Analysis Session — ELO Execution Package]]"
tags: [ga-aspire, calibration, verification, post-fix, april-29, evo-draw]
---

# GA ASPIRE Fix — Post-April-28 Verification Query Results

> **[Template filed April 28 — results to be filled April 29 before G1–G4 gates begin]**
>
> ELO runs all five queries in this document as the first step of the April 29 session, immediately after confirming G0 = GO. If any query returns ANOMALY DETECTED, file a SENTINEL queue item before proceeding to G1–G4.

---

## Section 1: Verification Query Set

### Query V1 — Event Tier Distribution Before vs. After

```sql
-- Confirm GA ASPIRE events are now correctly tagged
SELECT event_tier, COUNT(*) as event_count
FROM event_tiers
WHERE event_tier IN ('ga', 'ga_aspire', 'girls_academy')
ORDER BY event_tier;
```

**Pre-run expectation:** `ga_aspire` row should now exist with > 0 events. The `ga` row count should have decreased (Boys GA + non-ASPIRE Girls GA remain in `ga`; ASPIRE events have moved to `ga_aspire`).

**Expected `ga_aspire` count:** 15–50 events (based on GA Coverage Audit — 2026-04-23, Section 2 classifier estimate).

---

### Query V2 — Teams Affected by ASPIRE Fix

```sql
-- Count teams that had at least one game in a now-reclassified ga_aspire event
SELECT COUNT(DISTINCT team_id) as teams_affected
FROM (
  SELECT DISTINCT g.home_team_id AS team_id
  FROM games g
  JOIN event_tiers et ON g.event_name = et.event_name
  WHERE et.event_tier = 'ga_aspire'
    AND g.is_hidden = false
  UNION
  SELECT DISTINCT g.away_team_id
  FROM games g
  JOIN event_tiers et ON g.event_name = et.event_name
  WHERE et.event_tier = 'ga_aspire'
    AND g.is_hidden = false
) affected;
```

**Pre-run expectation:** 350–1,050 teams affected. If actual > 1,500: scope of fix exceeded expected range — investigate before proceeding.

---

### Query V3 — Rating Shift Magnitude (GA Girls, Post-Recompute)

```sql
-- Identify top movers among GA ASPIRE-affected teams post-recompute
-- Requires pre-April-28 snapshot in rankings_snapshot_pre_april28 table
-- (from Pre-April-28 Baseline Snapshot — Presten Execution Package)
SELECT
  r.age_group,
  r.gender,
  COUNT(*) AS team_count,
  ROUND(AVG(r.rating - snap.rating)::numeric, 1) AS avg_delta,
  MAX(r.rating - snap.rating) AS max_positive_delta,
  MIN(r.rating - snap.rating) AS max_negative_delta
FROM rankings r
JOIN rankings_snapshot_pre_april28 snap ON r.team_id = snap.team_id
WHERE r.team_id IN (
  SELECT DISTINCT g.home_team_id AS team_id
  FROM games g JOIN event_tiers et ON g.event_name = et.event_name
  WHERE et.event_tier = 'ga_aspire' AND g.is_hidden = false
  UNION
  SELECT DISTINCT g.away_team_id
  FROM games g JOIN event_tiers et ON g.event_name = et.event_name
  WHERE et.event_tier = 'ga_aspire' AND g.is_hidden = false
)
GROUP BY r.age_group, r.gender
ORDER BY r.age_group, r.gender;
```

**Pre-run expectation:**
- Girls GA ASPIRE teams: avg delta POSITIVE (+5 to +40 pts). The fix removes overcalibration that was suppressing ratings — wins in GA ASPIRE events now yield appropriate Glicko credit rather than near-zero credit against "expected" opponents.
- Boys GA ASPIRE teams: avg delta ≈ 0 (±5 pts). The Boys GA cal is already 100; no meaningful rating effect expected.
- Largest individual shift for any Girls team: < 80 pts. If any team shows > 80 pts, investigate — may indicate an unusually high GA ASPIRE game concentration or a merge artifact.

---

### Query V4 — Reference Club Spot Check

```sql
-- Confirm that known GA ASPIRE clubs now have ga_aspire tier assigned to their events
-- ELO: fill in 3 known GA ASPIRE club/event names from prior GA coverage audit work
SELECT DISTINCT e.event_name, et.event_tier, COUNT(g.id) AS game_count
FROM event_tiers et
JOIN games g ON g.event_name = et.event_name AND g.is_hidden = false
JOIN events e ON e.event_name = et.event_name
WHERE et.event_tier = 'ga_aspire'
  AND (
    LOWER(e.event_name) LIKE '%aspire%'
  )
GROUP BY e.event_name, et.event_tier
ORDER BY game_count DESC
LIMIT 20;
```

**Pre-run expectation:** All rows returned should have `event_tier = 'ga_aspire'` and event names containing "ASPIRE". If any row shows `event_tier = 'ga'` and name contains "ASPIRE": UPDATE did not fully execute — STOP and contact SENTINEL.

---

### Query V5 — Collateral Damage Check (Non-ASPIRE GA Events)

```sql
-- Confirm no non-ASPIRE GA events were accidentally reclassified
SELECT et.event_name, et.event_tier, COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) AS teams
FROM event_tiers et
JOIN games g ON g.event_name = et.event_name AND g.is_hidden = false
WHERE et.event_tier = 'ga_aspire'
  AND LOWER(et.event_name) NOT LIKE '%aspire%'
GROUP BY et.event_name, et.event_tier
ORDER BY teams DESC;
```

**Pre-run expectation:** Zero rows. Any row returned here represents a non-ASPIRE event incorrectly reclassified — this is a collateral damage flag. If any rows: document each event name, pause G1–G4, and contact SENTINEL with the list before proceeding.

---

## Section 2: Results Template (Fill April 29 Before G1–G4)

| Query | Expected | Actual | Status |
|-------|----------|--------|--------|
| V1: `ga_aspire` event count | 15–50 events | | PASS / FAIL |
| V2: Teams affected | 350–1,050 teams | | PASS / FAIL |
| V3: Girls GA ASPIRE avg delta direction | POSITIVE (ratings UP) | | PASS / FAIL |
| V3: Largest individual shift | < 80 pts | | PASS / FAIL |
| V4: Reference events confirm ga_aspire tier | All rows = ga_aspire | | PASS / FAIL |
| V5: Collateral reclassifications | 0 rows | | PASS / FAIL |

**Overall ASPIRE Fix Verdict:** CONFIRMED CLEAN / ANOMALY DETECTED

If ANOMALY DETECTED: file SENTINEL queue item (priority: high) before proceeding to G1–G4.

---

## Section 3: Rating Impact Summary (Fill April 29)

Based on Query V3 results:

| Metric | Value |
|--------|-------|
| GA Girls teams with rating change > +10 pts | _[fill]_ |
| GA Girls teams with rating change > +30 pts | _[fill]_ |
| Direction of changes (mostly UP / mostly DOWN / mixed) | _[fill]_ |
| Largest individual upward shift | _[fill team name + delta]_ |
| Boys avg delta (expected: ≈ 0) | _[fill]_ |
| Boys delta within ±5 pt threshold? | YES / NO |
| Direction consistent with fix intent (ratings UP for Girls ASPIRE teams)? | YES / NO |

If NO (direction wrong): Stop all April 29 gates. File G1 FAIL per the April 29 Decision Tree. Contact SENTINEL.

---

## Pre-Run Prediction Reference

The pre-run model predicts the following for Girls GA ASPIRE teams by age group:

| Age Group | Predicted Avg Delta | Reasoning |
|-----------|---------------------|-----------|
| U13G | +5 to +15 pts (cohort avg) | High ASPIRE game exposure in youngest GA age group |
| U14G | +8 to +18 pts (cohort avg) | Core GA ASPIRE age group |
| U15G | +6 to +15 pts (cohort avg) | ASPIRE participation tapers at older ages |
| U16G | +4 to +12 pts (cohort avg) | Lower ASPIRE exposure |
| U17G | +2 to +8 pts (cohort avg) | Minimal ASPIRE participation at this age |

Source: `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md`

Boys expected: 0 ± 5 pts across all age groups. Any Boys delta > 5 pts is a diagnostic flag.

---

## Escalation Summary

| Signal | Action |
|--------|--------|
| V1: `ga_aspire` event count = 0 | STOP. UPDATE did not run. Contact SENTINEL. Do not proceed to G1. |
| V2: Teams affected > 1,500 | Investigate scope before proceeding. Flag to SENTINEL. |
| V3: Girls delta NEGATIVE (ratings DOWN) | STOP. Fix had wrong effect. File G2 FAIL. Contact SENTINEL. |
| V3: Any individual shift > 80 pts | Investigate — likely merge artifact or extreme ASPIRE concentration. |
| V5: Collateral damage rows > 0 | Document each row. HOLD G2. Contact SENTINEL with list. |
| Boys delta > 5 pts | Flag in briefing. Investigate before authorizing Event Strength. |
| Boys delta > 15 pts | File G4 FAIL per Decision Tree. HOLD Event Strength and Rank Bands. |

---

## References

- [[ga-coverage-audit-results-2026-04-23]] — ASPIRE event classifier and expected counts (15–50 events, 500–1,500 games, 350–1,050 teams)
- [[Girls Calibration Status — Post-April-28 Narrative]] — expected direction (ratings UP) and mechanism explanation
- [[Post-April-28 Expected Rating Shift — Pre-Run Model]] — age-group predicted delta values
- [[April 29 Analysis Session — ELO Execution Package]] — Phase 1 runs this document's queries
- [[April 29 Post-Fix Decision Tree]] — G1–G4 gates that follow this verification

*ELO — 2026-04-28 12:50 (template); results filled April 29*
