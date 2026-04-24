---
title: Rank Bands Threshold Validation — 2026-04-24
tags: [rankings, rank-bands, validation, evo-draw, dss]
created: 2026-04-24
updated: 2026-04-24
author: ELO Agent
status: ready-to-run
related_task: task-2026-04-24-rank-bands-threshold-validation
---

# Rank Bands Threshold Validation — 2026-04-24

> Pre-flight document for Presten to run before the Rank Bands 5.5-hour implementation session. Run all queries in sequence. Total time: ~15 minutes in psql. Confirm pass/fail at each step before proceeding to implementation.

---

## Step 1: Planned Thresholds

These are the thresholds from the Rank Bands Implementation Plan, locked in here for reference. Do not adjust before running the validation queries below.

| Band   | Rating Threshold | Expected % of All Ranked Teams |
|--------|-----------------|-------------------------------|
| Gold   | ≥ 1500          | ~1%                           |
| Silver | 1350–1499       | ~4%                           |
| Bronze | 1200–1349       | ~20%                          |
| Red    | 1050–1199       | ~25%                          |
| Blue   | 900–1049        | ~25%                          |
| Green  | < 900           | ~25%                          |

**Design rationale:** Thresholds are anchored to the Glicko v7 rating distribution. The system starts teams near 1000 and calibrates upward via league bonuses (MLS NEXT Boys +160, GA Girls +140, ECNL +120–130). Gold at ≥1500 corresponds to approximately p99 — national-caliber ECNL and MLS NEXT programs only. Silver (~p95–p99) captures strong ECNL regular attendees. Bronze (~p75–p95) covers competitive ECNL RL/NPL programs.

---

## Step 2: Threshold Validation Query — U13G

Run this for the DSS demo age group. U13G is the primary demo anchor.

```sql
-- Rank Bands threshold validation — U13G
SELECT
  CASE
    WHEN rating >= 1500 THEN 'Gold'
    WHEN rating >= 1350 THEN 'Silver'
    WHEN rating >= 1200 THEN 'Bronze'
    WHEN rating >= 1050 THEN 'Red'
    WHEN rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS band,
  COUNT(*) AS team_count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 1) AS pct_of_total,
  MIN(rating)::int AS band_min,
  MAX(rating)::int AS band_max,
  ROUND(AVG(rating), 0)::int AS band_avg_rating
FROM rankings
WHERE age_group = 'U13' AND gender = 'F' AND rank IS NOT NULL
GROUP BY band
ORDER BY band_max DESC NULLS LAST;
```

**Expected output (approximate):**

| band   | team_count | pct_of_total | band_min | band_max | band_avg_rating |
|--------|-----------|-------------|---------|---------|----------------|
| Gold   | 3–15      | 0.5–1.5%   | 1500    | 1700+   | 1550           |
| Silver | 15–80     | 2–6%       | 1350    | 1499    | 1400           |
| Bronze | 200–600   | 15–25%     | 1200    | 1349    | 1265           |
| Red    | 300–700   | 20–30%     | 1050    | 1199    | 1120           |
| Blue   | 300–700   | 20–30%     | 900     | 1049    | 975            |
| Green  | 200–500   | 15–25%     | <900    | 899     | 820            |

---

## Step 3: Ratings Distribution Query — U13G

Run this to see the actual percentile breakpoints. Use these values to adjust thresholds if the Step 2 validation fails.

```sql
-- Ratings distribution for U13G — percentile breakpoints
SELECT
  PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY rating)::int AS p99_rating,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY rating)::int AS p95_rating,
  PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY rating)::int AS p90_rating,
  PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY rating)::int AS p75_rating,
  PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY rating)::int AS p50_median,
  PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY rating)::int AS p25_rating,
  PERCENTILE_CONT(0.10) WITHIN GROUP (ORDER BY rating)::int AS p10_rating,
  COUNT(*) AS total_ranked_teams
FROM rankings
WHERE age_group = 'U13' AND gender = 'F' AND rank IS NOT NULL;
```

**Reading this output:**

- If `p99_rating` ≈ 1500 → Gold threshold is calibrated correctly (top 1% maps to ≥1500)
- If `p95_rating` ≈ 1350 → Silver threshold is calibrated correctly (top 5% starts at 1350)
- If `p75_rating` ≈ 1200 → Bronze threshold is calibrated correctly (top 25% starts at 1200)
- If `p50_median` ≈ 1050 → Red threshold is at the median, which is correct (average team = Red)

The design assumes a median near 1050. If median is significantly different, see the Adjustment Algorithm in Step 5.

---

## Step 4: Pass/Fail Decision Rules

Run the Step 2 query, then apply these rules for each band.

### Gold (≥ 1500)

| Result | Status | Action |
|--------|--------|--------|
| 0.5%–2% of U13G teams | ✅ PASS | Proceed |
| < 0.3% (fewer than ~5 teams) | 🚨 FAIL | Threshold too high — lower Gold to ≥1450 |
| > 4% of U13G teams | 🚨 FAIL | Threshold too low — raise Gold to ≥1550 |

**Why 0.5–2%:** Gold should represent national-caliber programs that any club director would recognize. If 4% of teams are Gold, the badge loses prestige at the DSS demo. If fewer than 5 teams qualify, the demo claim ("see your Gold-band ranking") has no one to point to.

### Silver (1350–1499)

| Result | Status | Action |
|--------|--------|--------|
| 2%–8% of U13G teams | ✅ PASS | Proceed |
| < 1% | 🚨 FAIL | Silver threshold too high — lower Silver floor to ≥1300 |
| > 12% | 🚨 FAIL | Silver too broad — raise Silver floor to ≥1400 |

### Bronze (1200–1349)

| Result | Status | Action |
|--------|--------|--------|
| 12%–28% of U13G teams | ✅ PASS | Proceed |
| < 8% | 🚨 FAIL | Threshold too high — lower Bronze floor to ≥1150 |
| > 35% | 🚨 FAIL | Threshold too low — raise Bronze floor to ≥1250 |

### Red, Blue, Green

No hard pass/fail. These exist to fill the distribution. Check that no band has 0 teams, which would indicate a threshold collision or data issue. If any of Gold/Silver/Bronze pass, proceed — Red/Blue/Green will be correct by construction.

### Overall distribution sanity check

After running Step 2: add up Gold + Silver + Bronze. This "upper half" should be approximately 25–35% of total ranked U13G teams. If it is below 20% or above 40%, the distribution has drifted from the design assumptions and the Adjustment Algorithm (Step 5) applies.

---

## Step 5: Adjustment Algorithm

**Use only if one or more bands fail the Step 4 decision rules.**

**Step 5A:** Run the Step 3 distribution query. Record actual p99, p95, p75, p50 values.

**Step 5B:** Compare actual percentiles to planned thresholds:

```
If actual p99 = X, and X differs from 1500 by more than 50 points:
  → shift ALL thresholds by (X − 1500)
  → New Gold:   ≥ X
  → New Silver: ≥ X − 150  (maintains 150-pt band width)
  → New Bronze: ≥ X − 300
  → New Red:    ≥ actual p50 (median = Red floor, always)
  → New Blue:   ≥ actual p50 − 150
```

**Step 5C:** Re-run the Step 2 validation query with adjusted threshold values (replace 1500, 1350, 1200, 1050, 900 in the CASE statement with the new values).

**Step 5D:** Check pass/fail again. If all three bands pass, proceed with adjusted thresholds. Update the implementation plan before running the session — document the new values in the Rank Bands Implementation Plan pre-flight section.

**Example:** If actual p99 is 1440, delta = −60:
- Gold ≥ 1440, Silver ≥ 1290, Bronze ≥ 1140, Red ≥ 990, Blue ≥ 840, Green < 840

**Time to execute:** < 10 minutes (one distribution query + one recalculation of the CASE thresholds).

---

## Step 6: Demo Team Spot Check Query

Verify that Football Academy NJ and other known anchor teams land in appropriate bands. This is a go/no-go check for the DSS demo specifically.

```sql
-- Demo team band assignment spot check
SELECT t.name, r.rating, r.rank,
  CASE
    WHEN r.rating >= 1500 THEN 'Gold'
    WHEN r.rating >= 1350 THEN 'Silver'
    WHEN r.rating >= 1200 THEN 'Bronze'
    WHEN r.rating >= 1050 THEN 'Red'
    WHEN r.rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS band
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND (t.name ILIKE '%football academy%'
    OR t.name ILIKE '%wall soccer%'
    OR t.name ILIKE '%wall sc%'
    OR t.name ILIKE '%FC Stars%'
    OR t.name ILIKE '%PA Classics%')
ORDER BY r.rank;
```

**Expected results:**

| Team | Expected Band | If Wrong |
|------|--------------|----------|
| Football Academy NJ (any variant) | **Gold** | If Silver: lower Gold threshold or investigate merge |
| Wall SC (if present at U13G) | Gold or high Silver | Informational only |
| FC Stars (if present at U13G) | Silver or Bronze | Informational only |

**Critical:** If Football Academy NJ does not appear at all, or appears twice, stop. The team may be split across two records in the DB. Run the merge investigation queries from `Pre-DSS Team Merge Audit — U13G.md` before proceeding with Rank Bands implementation.

**If Football Academy NJ is Silver instead of Gold:** Check their actual rating. If their rating is 1450–1499, consider lowering the Gold threshold to ≥1450 — this is a judgment call. Football Academy NJ is the most visible team in the DSS audience's geography. If they're near the Gold/Silver boundary, the boundary should move to include them if their rating is within 50 points of 1500.

---

## Run Sequence Summary

Run these queries in order before starting the implementation session:

1. **Step 2** — band distribution validation (5 minutes)
2. **Step 4** — apply pass/fail rules (2 minutes reading)
3. **Step 3** — distribution percentiles if any band fails (2 minutes)
4. **Step 5** — adjust thresholds if needed (10 minutes if adjustment required, 0 if all pass)
5. **Step 6** — demo team spot check (3 minutes)

**Total time:** 10–25 minutes depending on whether adjustment is needed.

**Go/No-Go:** If all Step 4 rules pass AND Football Academy NJ is Gold → proceed to implementation session. If any rule fails or Football Academy NJ is not Gold → resolve before starting the 5.5-hour session.

---

## Related

- [[Rank Bands — Implementation Plan]] — the implementation session this document gates
- [[Rank Bands Design]] — threshold design rationale
- [[Pre-DSS Team Merge Audit — U13G]] — Football Academy NJ merge investigation
- [[DSS Demo Spec — May 2026]] — Football Academy NJ as Gold anchor requirement
