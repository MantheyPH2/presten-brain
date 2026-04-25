---
title: Rank Bands View — Implementation SQL
type: implementation-sql
author: ELO
date: 2026-04-24
status: ready-to-run
related: "[[Rank Bands — Presten Decision Brief]]", "[[Rank Bands — Threshold Validation]]", "[[DSS Data Readiness Validation Plan — May 14-15]]"
tags: [rank-bands, sql, view, implementation, evo-draw]
---

# Rank Bands View — Implementation SQL

> **Summary:** Complete `CREATE OR REPLACE VIEW rankings_with_bands` SQL using the validated 6-band thresholds: Gold ≥1500, Silver ≥1350, Bronze ≥1200, Red ≥1050, Blue ≥900, Green <900. Run this before the DSS demo. Rollback is instantaneous (no data modified). Three post-creation validation queries included.

---

## Section 1: Schema Requirements Check

Before creating the view, confirm these columns exist on the `rankings` table:

```sql
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'rankings'
  AND column_name IN ('team_id', 'age_group', 'rating', 'rank', 'gender', 'last_game_date')
ORDER BY column_name;
```

Expected: all 6 columns present. If `rank` is named differently (e.g., `age_group_rank` or `position`), update the view SQL accordingly.

---

## Section 2: The View SQL

```sql
CREATE OR REPLACE VIEW rankings_with_bands AS
SELECT
  r.team_id,
  r.age_group,
  r.gender,
  r.rating,
  r.rank,
  r.last_game_date,
  CASE
    WHEN r.rating >= 1500 THEN 'Gold'
    WHEN r.rating >= 1350 THEN 'Silver'
    WHEN r.rating >= 1200 THEN 'Bronze'
    WHEN r.rating >= 1050 THEN 'Red'
    WHEN r.rating >= 900  THEN 'Blue'
    ELSE                       'Green'
  END AS band,
  RANK() OVER (
    PARTITION BY r.age_group, r.gender,
      CASE
        WHEN r.rating >= 1500 THEN 'Gold'
        WHEN r.rating >= 1350 THEN 'Silver'
        WHEN r.rating >= 1200 THEN 'Bronze'
        WHEN r.rating >= 1050 THEN 'Red'
        WHEN r.rating >= 900  THEN 'Blue'
        ELSE                       'Green'
      END
    ORDER BY r.rating DESC
  ) AS within_band_rank
FROM rankings r
WHERE r.rank IS NOT NULL;
```

**Notes:**
- The `WHERE r.rank IS NOT NULL` filter excludes teams with no competitive ranking (no games or insufficient data). Unranked teams are not assigned a band.
- `within_band_rank = 1` means the highest-rated team within that (age_group, gender, band) combination.
- The CASE is repeated in both the SELECT and the PARTITION BY clause — this is necessary for SQL compatibility (can't alias a computed column in the same SELECT's window function).

---

## Section 3: Validation Queries

Run all three immediately after `CREATE OR REPLACE VIEW` completes.

**V1 — Row count sanity check (should match `rankings` table)**

```sql
-- Compare view row count to rankings table (must match exactly)
SELECT
  (SELECT COUNT(*) FROM rankings_with_bands)   AS view_rows,
  (SELECT COUNT(*) FROM rankings WHERE rank IS NOT NULL) AS rankings_rows,
  (SELECT COUNT(*) FROM rankings_with_bands) =
  (SELECT COUNT(*) FROM rankings WHERE rank IS NOT NULL) AS counts_match;
```

Expected: `counts_match = true`. If false, the view has an unintended join or filter.

**V2 — Band distribution for U13G (DSS demo age group)**

```sql
SELECT
  band,
  COUNT(*) AS team_count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 1) AS pct_of_age_group,
  ROUND(MIN(rating), 0) AS band_min_rating,
  ROUND(MAX(rating), 0) AS band_max_rating,
  ROUND(AVG(rating), 0) AS band_avg_rating
FROM rankings_with_bands
WHERE age_group = 'U13' AND gender = 'F'
GROUP BY band
ORDER BY band_min_rating DESC NULLS LAST;
```

Expected distribution (from threshold validation):
- Gold: 5–15% of U13G ranked teams
- Silver: 15–30%
- Bronze: 20–35%
- Red: 15–25%
- Blue: 10–20%
- Green: remainder

If Gold >20% or <3%, the thresholds may be misapplied. Check that `rating` column is in the same numeric scale as the thresholds (1000–1700 range, not 0–1 or 0–100).

**V3 — Top 10 U13G with bands (DSS demo spot check)**

```sql
SELECT t.name, rwb.rating, rwb.rank, rwb.band, rwb.within_band_rank
FROM rankings_with_bands rwb
JOIN teams t ON t.id = rwb.team_id
WHERE rwb.age_group = 'U13' AND rwb.gender = 'F'
  AND rwb.rank <= 10
ORDER BY rwb.rank;
```

Expected:
- All 10 rows have a non-null `band`
- Top teams are Gold or high-Silver (rating ≥1350 for top 10 of a competitive age group)
- Football Academy NJ should appear at or near rank 1 with band = 'Gold'

If Football Academy NJ is not present in the top 10, run:
```sql
SELECT t.name, rwb.rating, rwb.rank, rwb.band
FROM rankings_with_bands rwb
JOIN teams t ON t.id = rwb.team_id
WHERE rwb.age_group = 'U13' AND rwb.gender = 'F'
  AND (t.name ILIKE '%football academy%' OR t.name ILIKE '%FA NJ%')
ORDER BY rwb.rank;
```
Flag to SENTINEL if Football Academy NJ is not in the top 30 or is not Gold.

---

## Section 4: Rollback

The view creates no data — it reads from `rankings`. Rollback is instant and safe at any time:

```sql
DROP VIEW IF EXISTS rankings_with_bands;
```

No downstream tables depend on this view at time of creation. If the API or frontend is already querying `rankings_with_bands`, dropping the view will cause those queries to fail until the view is recreated.

---

## Section 5: Pre-Demo Integration Check

The DSS Data Readiness Validation Plan (Sections 1A–1D) references these specific queries against `rankings_with_bands`:

**Check from Section 1A — coverage count:**
```sql
SELECT COUNT(*) FROM rankings_with_bands WHERE age_group = 'U13' AND gender = 'F';
```
Pass threshold: ≥200 rows. This will pass if any meaningful number of U13G teams are ranked.

**Check from Section 1D — Football Academy NJ band:**
```sql
SELECT t.name, rwb.rating, rwb.band
FROM rankings_with_bands rwb
JOIN teams t ON t.id = rwb.team_id
WHERE rwb.age_group = 'U13' AND rwb.gender = 'F'
  AND t.name ILIKE '%football academy%'
ORDER BY rwb.rank;
```
Expected: band = 'Gold' (requires Football Academy NJ rating ≥1500).

If Football Academy NJ's current rating is between 1350–1499 (Silver), note this: the team is still well-ranked but the demo narrative changes from "Gold — top tier nationally" to "Silver — elite nationally." This does not block the demo, but SENTINEL should be informed before May 14.

---

## Band Threshold Reference

| Band | Rating Threshold | Interpretation |
|------|-----------------|----------------|
| Gold | ≥ 1500 | Top tier nationally — elite competitive programs |
| Silver | ≥ 1350 | High competitive — strong regional/national standing |
| Bronze | ≥ 1200 | Solid competitive — consistent results at high tiers |
| Red | ≥ 1050 | Developing competitive — regional competition quality |
| Blue | ≥ 900 | Entry competitive — limited game data or lower tiers |
| Green | < 900 | Minimal data or local/recreational competition only |

Thresholds are uniform across age groups and genders. The per-age-group percentile analysis confirmed these thresholds produce sensible distributions for U13G through U17G; see [[Rank Bands — Threshold Validation]] for the full validation data.

---

## References

- [[Rank Bands — Presten Decision Brief]] — threshold validation results and implementation readiness status
- [[Rank Bands — Threshold Validation]] — statistical backing for the 6 threshold values
- [[DSS Data Readiness Validation Plan — May 14-15]] — Sections 1A–1D reference this view
- [[Rank Bands — Implementation Plan]] — implementation session context
