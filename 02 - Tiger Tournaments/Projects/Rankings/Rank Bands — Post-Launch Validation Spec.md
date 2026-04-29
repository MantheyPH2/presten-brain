---
type: elo-validation-spec
topic: rank-bands-post-launch
date: 2026-04-29
execution_window: 2026-05-02 to 2026-05-08
status: ready-for-execution
author: ELO
tags: [elo, rank-bands, validation, post-launch, may1, evo-draw]
---

# Rank Bands — Post-Launch Validation Spec

> **When to execute:** Run queries RB-V-1 through RB-V-4 any time May 2–8 after the May 1 pipeline has run. Send results to ELO. ELO files pass/fail verdict in the Pre-May-9 Calibration State Dashboard by May 8.

---

## Section 1 — What Rank Bands Are and What Could Go Wrong

Rank bands assign each ranked team a tier label (Gold / Silver / Bronze / Red / Blue / Green) based on absolute Elo rating thresholds: Gold ≥ 1500, Silver 1350–1499, Bronze 1200–1349, Red 1050–1199, Blue 900–1049, Green < 900. Bands are computed as a SQL view (`rankings_with_bands`) — not stored — so they always reflect the current ratings.

**Failure modes being checked:**

- **Band collapse:** All teams fall into 1–2 bands (thresholds miscalibrated relative to actual rating distribution)
- **Band overfill:** One band absorbs > 50% of all ranked teams (threshold too broad)
- **Empty bands:** A band has zero teams (threshold too aggressive or rating floor/ceiling issues)
- **Rating drift:** May 1 pipeline new-game ingestion shifts Elo enough to push large numbers of teams across band boundaries unexpectedly

---

## Section 2 — Validation Queries

### RB-V-1: Band Distribution Count

**What it measures:** Number of teams per rank band, by age group and gender. Primary sanity check.

```sql
SELECT
  age_group,
  gender,
  CASE
    WHEN rating >= 1500 THEN 'Gold'
    WHEN rating >= 1350 THEN 'Silver'
    WHEN rating >= 1200 THEN 'Bronze'
    WHEN rating >= 1050 THEN 'Red'
    WHEN rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS rank_band,
  COUNT(*) AS team_count
FROM rankings
WHERE last_game_date > NOW() - INTERVAL '7 months'
  AND rank IS NOT NULL
GROUP BY age_group, gender, rank_band
ORDER BY age_group, gender,
  CASE
    WHEN rating >= 1500 THEN 1
    WHEN rating >= 1350 THEN 2
    WHEN rating >= 1200 THEN 3
    WHEN rating >= 1050 THEN 4
    WHEN rating >= 900  THEN 5
    ELSE 6
  END;
```

**Expected output:** Multiple rows per age_group/gender combination, one per non-empty band. Distribution should be roughly bottom-heavy (Green/Blue/Red largest; Gold smallest).

**PASS:** All six bands have at least 1 team for age groups with ≥ 100 ranked teams; no single band exceeds 50% of teams in any age/gender group.

**FAIL:** Any band with 0 teams for a large age group (≥ 100 ranked teams); OR any band containing > 50% of an age group's ranked teams.

---

### RB-V-2: Band Threshold Coverage Check

**What it measures:** Min and max Elo rating within each band. Confirms thresholds are contiguous (no gaps, no overlaps).

```sql
SELECT
  CASE
    WHEN rating >= 1500 THEN 'Gold'
    WHEN rating >= 1350 THEN 'Silver'
    WHEN rating >= 1200 THEN 'Bronze'
    WHEN rating >= 1050 THEN 'Red'
    WHEN rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS rank_band,
  MIN(rating)::int AS min_rating_in_band,
  MAX(rating)::int AS max_rating_in_band,
  COUNT(*) AS team_count
FROM rankings
WHERE last_game_date > NOW() - INTERVAL '7 months'
  AND rank IS NOT NULL
GROUP BY rank_band
ORDER BY MIN(rating) DESC;
```

**Expected output:** 6 rows (one per band). For contiguous thresholds: each band's min_rating_in_band should be ≥ the band's threshold; max_rating_in_band should be < the next band's threshold (or the full rating max for Gold).

**PASS:** No rating appears in two bands; band min/max ranges are contiguous with no gaps.

**FAIL:** Any gap between max of one band and min of the next (indicates a rating that belongs to no band — possible if ratings table has NULL or negative values). OR any band shows ratings outside its defined threshold range (indicates view logic error).

---

### RB-V-3: Post-Pipeline Rating Shift per Band

**What it measures:** Average Elo change per band from before to after the May 1 pipeline run. Confirms new-game ingestion did not produce anomalous rating drift.

*Note: This query requires a pre-May-1 snapshot to compare against. Use the baseline in `Rankings/May 1 Pipeline Launch — ELO Ratings Baseline.md` as the reference. If a snapshot table (e.g., `rankings_snapshot_pre_may1`) was created before May 1, use it. Otherwise, this check can be approximated using the expected stddev baseline values from that document.*

```sql
-- Run this after May 1 pipeline if a pre-launch snapshot table exists:
SELECT
  CASE
    WHEN r.rating >= 1500 THEN 'Gold'
    WHEN r.rating >= 1350 THEN 'Silver'
    WHEN r.rating >= 1200 THEN 'Bronze'
    WHEN r.rating >= 1050 THEN 'Red'
    WHEN r.rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS post_launch_band,
  COUNT(*) AS team_count,
  ROUND(AVG(r.rating - s.rating)::numeric, 1) AS avg_rating_shift,
  ROUND(STDDEV(r.rating - s.rating)::numeric, 1) AS stddev_rating_shift,
  MAX(ABS(r.rating - s.rating))::int AS max_abs_shift
FROM rankings r
JOIN rankings_snapshot_pre_may1 s ON s.team_id = r.team_id
WHERE r.last_game_date > NOW() - INTERVAL '7 months'
  AND r.rank IS NOT NULL
GROUP BY post_launch_band
ORDER BY MIN(r.rating) DESC;
```

**Expected output:** 6 rows. avg_rating_shift should be small (< ±10 pts average) for most bands in the first week of a new pipeline run.

**PASS:** All bands: avg_rating_shift within ±10 pts; no band with max_abs_shift > 100 pts for a large fraction of teams.

**FAIL:** Any band with avg_rating_shift > 25 pts OR max_abs_shift indicating a large population crossed thresholds unexpectedly → flag in Pre-May-9 Calibration State Dashboard; escalate to SENTINEL if Red-band or higher is affected.

---

### RB-V-4: Cross-Gender Band Contamination Check

**What it measures:** Confirms that Boys and Girls rank band populations are computed independently with no cross-gender contamination (no team appearing in both Boys and Girls band sets).

```sql
SELECT
  boys.team_id,
  boys.age_group,
  boys.gender AS boys_gender,
  girls.gender AS girls_gender
FROM rankings boys
JOIN rankings girls ON girls.team_id = boys.team_id
WHERE boys.gender = 'M'
  AND girls.gender = 'F'
  AND boys.rank IS NOT NULL
  AND girls.rank IS NOT NULL;
```

**Expected output:** Zero rows (no team should appear in both Boys and Girls rankings).

**PASS:** Query returns zero rows.

**FAIL:** Any rows returned → pipeline data quality bug. A team_id appears in both Boys and Girls rankings. FORGE investigation required; pause band display in UI until resolved.

---

## Section 3 — Pass/Fail Decision Table

| Query | PASS | FAIL | Action on Fail |
|-------|------|------|----------------|
| RB-V-1 | All bands non-empty (for large age groups); no band > 50% | Any band empty or any band > 50% in an age group | Adjust band thresholds in `rankings_with_bands` view; alert Presten |
| RB-V-2 | No rating gaps between bands; all min/max values inside expected threshold ranges | Gap or overlap detected; rating outside band range | Review `rankings_with_bands` view SQL; file as FORGE data quality bug if ratings are out of range |
| RB-V-3 | All bands: avg shift ≤ ±10 pts; no band with extreme outlier concentration | Any band: avg shift > 25 pts | Flag in Pre-May-9 Calibration State Dashboard; update R7 in May 9 Risk Register; escalate if anomaly persists past May 4 |
| RB-V-4 | Zero cross-gender team_ids | Any cross-gender team_id found | Immediate FORGE investigation; pause band display in rankings UI; do not show at DSS until resolved |

---

## Section 4 — Execution Timeline

| Step | When | Who | Action |
|------|------|-----|--------|
| Pre-stage queries | April 29–30 | ELO | File this spec (complete) |
| May 1 pipeline runs | May 1 | Presten / FORGE | First pipeline execution with new calibration; GA ASPIRE fix active (if April 29 session ran) |
| Run RB-V-1 through RB-V-4 | May 2–8 | Presten | Run all four queries; share output with ELO |
| ELO verdict | May 8 | ELO | Fill rank bands section in Pre-May-9 Calibration State Dashboard with pass/fail for each query |
| SENTINEL review | May 8–9 | SENTINEL | Incorporate into May 9 DSS gate assessment |

---

## Section 5 — Expected Band Shape

Based on the [[Rank Bands Design]] document and the estimated Elo distribution for the current `rankings` table (estimated ~30,000–50,000 teams with sufficient games):

| Band | Rating Range | Approx % of Ranked Teams | Approx Team Count |
|------|-------------|--------------------------|-------------------|
| Gold | ≥ 1500 | ~1% | 300–500 |
| Silver | 1350–1499 | ~4% | 1,200–2,000 |
| Bronze | 1200–1349 | ~20% | 6,000–10,000 |
| Red | 1050–1199 | ~25% | 7,500–12,500 |
| Blue | 900–1049 | ~25% | 7,500–12,500 |
| Green | < 900 | ~25% | 7,500–12,500 |

**Validation note:** If RB-V-1 shows Gold at > 5% or Green at < 10%, the absolute rating thresholds may be miscalibrated for the current rating distribution. ELO should re-run the distribution query from the [[Rank Bands Design]] document (Section 1) to check actual percentile anchors and adjust thresholds if needed. A threshold tuning at this stage is a view change only — zero migration cost.

---

## References

- [[Rank Bands Design]] — band thresholds, SQL view, expected distribution
- `Rankings/Rank Bands View — Implementation SQL.md` — production view implementation
- `Rankings/Rank Bands Threshold Validation — 2026-04-24.md` — prior validation (pre-launch)
- `Rankings/May 1 Pipeline Launch — ELO Ratings Baseline.md` — pre-launch baseline for RB-V-3 comparison
- `Rankings/Pre-May-9 Calibration State Dashboard.md` — ELO files verdict here by May 8

*ELO — 2026-04-29*
