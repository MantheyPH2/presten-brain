---
title: April 29 Gate Results — Structured Log
type: gate-log
author: ELO
date: 2026-04-27
status: template-ready
related: "[[April 29 Post-Fix Decision Tree]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]", "[[April 29 ELO Session Master Script]]"
tags: [gate, april-29, calibration, ga-aspire, elo, evo-draw]
---

# April 29 Gate Results — Structured Log

> **[Template — ELO fills on April 29. Do not pre-fill result fields.]**
>
> Pre-Run predicted columns are filled from `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` (written 2026-04-25, before April 28 execution). These values must NOT be modified on April 29 — they are the pre-committed prediction ELO is testing against. Fill only the "Actual" and "PASS/FAIL" columns during the session.

---

## Header Block (ELO fills on April 29)

```
Session date: April 29, 2026
Filled by: ELO (based on Presten query output)
April 28 confirmed: YES / NO — [source: April 28 Execution Log]
Pre-run model reference: Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md (2026-04-25)
```

---

## G1 — Source Data Gate

Confirm the GA ASPIRE UPDATE ran and the ecnl_verified prerequisite is in place. If G1 fails, do not proceed to G2–G4. File blocker and contact SENTINEL.

| Field | Expected | Actual (April 29) | PASS / FAIL |
|-------|----------|-------------------|-------------|
| GA ASPIRE events tagged `ga_aspire` (not `ga`) | COUNT(*) > 0 WHERE event_name ILIKE '%ASPIRE%' AND event_tier = 'ga_aspire' | | |
| GA events still tagged `ga` with ASPIRE in name | COUNT(*) = 0 WHERE event_name ILIKE '%ASPIRE%' AND event_tier = 'ga' | | |
| `ecnl_verified` column present on games table | TRUE (column exists, has non-null default) | | |
| ECNL game count (Section 4 source check) | Consistent with pre-April-28 baseline (within ±5% of snapshot count) | | |

**G1 SQL — ASPIRE fix confirmation:**
```sql
-- Must return 0 for G1 to pass on the 'still-tagged-ga' row
SELECT COUNT(*) AS aspire_still_tagged_ga
FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga';

-- Must return > 0 for G1 to pass on the 'tagged-ga_aspire' row
SELECT COUNT(*) AS aspire_correctly_tagged
FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga_aspire';
```

**G1 overall: PASS / FAIL**

If FAIL: do not run G2–G4. Identify which row failed. If ASPIRE events are still tagged `ga`, the April 28 UPDATE did not execute — rerun the GA ASPIRE UPDATE SQL before proceeding. File the blocker in briefing and contact SENTINEL.

---

## G2 — Rating Distribution Gate

Reference: `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` Section 3.

**Pre-Run Predicted columns filled from the Pre-Run Model (2026-04-25) — do not modify on April 29.**

The "Predicted Magnitude" column refers to the average cohort shift (all teams in that age group, not just GA ASPIRE-exposed teams). Highly-exposed teams will see larger individual shifts (up to 40 pts); the cohort average will be smaller.

| Age Group | Pre-Run Predicted Direction | Pre-Run Predicted Magnitude — Cohort Avg (pts) | Actual Direction | Actual Magnitude — Cohort Avg (pts) | In Range? |
|-----------|----------------------------|-------------------------------------------------|-----------------|--------------------------------------|-----------|
| U13G | INCREASE | +5 to +15 | | | |
| U14G | INCREASE | +8 to +18 | | | |
| U15G | INCREASE | +6 to +15 | | | |
| U16G | INCREASE | +4 to +12 | | | |
| U17G | INCREASE | +2 to +8 | | | |
| Boys U13B | 0 pts | 0 ± 5 pts | | | |
| Boys U14B | 0 pts | 0 ± 5 pts | | | |
| Boys U15B | 0 pts | 0 ± 5 pts | | | |
| Boys U16B | 0 pts | 0 ± 5 pts | | | |
| Boys U17B | 0 pts | 0 ± 5 pts | | | |

**G2 SQL:**
```sql
-- Average rating by age group — run post-April-28 recompute and compare to pre-April-28 snapshot
SELECT
  age_group,
  LEFT(age_group, LENGTH(age_group) - 1) AS gender,
  ROUND(AVG(rating), 1) AS post_fix_avg_rating,
  COUNT(*) AS team_count
FROM rankings
GROUP BY age_group
ORDER BY age_group;
```

Compare against pre-April-28 snapshot values from `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md`.

**G2 overall: PASS / FAIL**

Pass criterion: All Girls age groups show positive actual direction AND actual magnitude within predicted range. Boys delta must be 0 ± 5 pts for all age groups. A Girls age group showing negative direction (ratings went DOWN) is a G2 hard fail — the fix applied backward or the UPDATE scope was wrong.

---

## G3 — Calibration Stability Gate

Reference: `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` Section 5 (Anomaly Detection Thresholds).

**Thresholds pre-committed from the Pre-Run Model — do not modify on April 29.**

| Signal | Threshold (from Pre-Run Model) | Actual | PASS / FAIL |
|--------|-------------------------------|--------|-------------|
| Girls GA ASPIRE avg rating change direction | POSITIVE (any increase = expected) | | |
| Girls GA ASPIRE avg rating change magnitude (U13G) | +10 to +40 pts | | |
| Girls GA ASPIRE avg rating change magnitude (U14G) | +10 to +40 pts | | |
| Girls ECNL avg rating change | < 5 pts (small indirect effect only) | | |
| Boys avg rating change — any age group | 0 ± 5 pts (hard: > 15 pts = G3 fail) | | |
| Total teams with > 50-pt rating shift (any direction) | < 80 teams | | |
| GA ASPIRE avg change vs. ECNL avg change direction | GA ASPIRE ↑ while ECNL minimal (convergence) | | |

**G3 SQL — GA ASPIRE avg rating:**
```sql
SELECT
  r.age_group,
  ROUND(AVG(r.rating), 1) AS post_fix_avg
FROM rankings r
JOIN team_events te ON te.team_id = r.team_id
JOIN events e ON e.id = te.event_id
WHERE e.event_tier = 'ga_aspire'
  AND e.season >= '2025-2026'
GROUP BY r.age_group
ORDER BY r.age_group;
```

**G3 SQL — Teams with > 50-pt shift:**
```sql
-- Requires pre-April-28 baseline snapshot to be in a separate table/snapshot
-- Compare against snapshot from Pre-April-28 Baseline Snapshot package
SELECT COUNT(*) AS large_movers
FROM rankings r
JOIN rankings_snapshot s ON s.team_id = r.team_id AND s.age_group = r.age_group
WHERE ABS(r.rating - s.rating) > 50;
```

**G3 overall: PASS / FAIL**

If Boys avg rating change > 5 pts for any age group: mandatory investigation before Event Strength Phase 1 proceeds. If > 15 pts: G3 hard fail — hold all downstream decisions. If total teams > 50-pt shift exceeds 80: investigate outlier teams before marking G3 pass.

---

## G4 — Baseline Snapshot Delta Gate

Reference: `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md` for snapshot values.

| Field | Pre-April-28 Snapshot Value (ELO fills from snapshot doc) | Post-April-28 Value (ELO fills April 29) | Delta | Flag? |
|-------|-----------------------------------------------------------|------------------------------------------|-------|-------|
| Total ranked teams (all age groups) | | | | |
| Girls GA ASPIRE avg rating (U13G) | | | | |
| Girls GA ASPIRE avg rating (U14G) | | | | |
| Girls ECNL avg rating | | | | |
| Boys top-10 avg rating (all age groups combined) | | | | |
| Total teams ranked (Girls GA combined) | | | | |

**Flag criterion:** Delta outside the range predicted by the Pre-Run Model for that row. For Girls GA ASPIRE: a delta in the NEGATIVE direction is a flag. For Boys: any delta > ±5 pts is a flag. For ECNL: delta > ±10 pts is a flag.

**G4 overall: PASS / FAIL**

Note: G4 also establishes the April 29 baseline that the Event Strength Phase 1 ELO Execution Package Query 2 (rating drift check) references. ELO should save the "Post-April-28 Value" column from this table — it becomes the Phase 1 baseline.

---

## Summary Disposition

```
G1: PASS / FAIL
G2: PASS / FAIL
G3: PASS / FAIL
G4: PASS / FAIL

All 4 gates PASS: YES / NO

ELO recommendation: AUTHORIZE Event Strength Phase 1 / HOLD pending [specific gate failure]

If HOLD — gate failed:
  Reason: [specific row and value that failed]
  Required to clear: [specific action — e.g., "rerun GA ASPIRE UPDATE", "investigate Boys drift"]
  ELO will re-check: [next session date]
```

---

## SENTINEL Authorization Field

> SENTINEL fills — do not pre-fill. ELO leaves this blank.

```
Decision document reviewed: [ ]
G1–G4 all pass: YES / NO
Event Strength Phase 1: AUTHORIZED / HELD
If HELD: reason and condition to clear: ___________
SENTINEL authorization date: ___________
SENTINEL sign-off: ___________
```

---

## April 29 Filing Instructions

At end of April 29 session:
1. ELO fills all "Actual" and "PASS/FAIL" cells from Presten's query output
2. ELO fills Summary Disposition with recommendation
3. ELO includes in April 29 briefing: "Gate results log filed: G1 [result], G2 [result], G3 [result], G4 [result]. All pass: [YES/NO]. Recommendation: [authorize/hold]."
4. If any gate fails: include the specific failing row and value, and the required remediation step
5. Leave SENTINEL Authorization Field blank

---

## References

- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — predicted values for G2 and anomaly thresholds for G3 (written 2026-04-25)
- `Rankings/April 29 Post-Fix Decision Tree.md` — G1–G4 gate framework
- `Rankings/April 29 ELO Session Master Script.md` — session steps this log records
- `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md` — G4 baseline values
- `Rankings/Event Strength Phase 1 — ELO Execution Package.md` — uses G4 values as Phase 1 baseline

*ELO — 2026-04-27 (template; filled April 29)*
