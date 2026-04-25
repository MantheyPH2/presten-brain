---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: high
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Rank Bands View — Implementation SQL.md"
---

# Task: Rank Bands View — Implementation SQL

## Context

ELO has completed the Rank Bands design and threshold validation:
- 6-band system: Gold ≥1500, Silver ≥1350, Bronze ≥1200, Red ≥1050, Blue ≥900, Green <900
- Thresholds validated against actual rating distributions across age groups
- Presten brief written (`Rankings/Rank Bands — Presten Brief.md`)

The DSS demo (May 16) requires a `rankings_with_bands` view. The validation plan (`Product & Planning/DSS Data Readiness Validation Plan — May 14-15.md`) already references this view in Sections 1A–1D. What does NOT exist yet: the actual `CREATE OR REPLACE VIEW` SQL Presten runs to create it.

This task closes that gap.

## What to Write

Write `02 - Tiger Tournaments/Projects/Rankings/Rank Bands View — Implementation SQL.md`.

---

### Section 1: Schema Requirements Check

Before the view can be created, confirm the following columns exist on the `rankings` table (or equivalent):
- `team_id`
- `age_group`
- `rating` (numeric)
- `rank` (integer, within age_group)

If any column is absent or named differently, note the correct column name and adjust the SQL accordingly.

### Section 2: The View SQL

Write the complete `CREATE OR REPLACE VIEW rankings_with_bands AS ...` statement.

The view must include:
- `team_id`
- `age_group`
- `rating`
- `rank` (overall rank within age_group)
- `band` — assigned by CASE statement using the validated thresholds:
  ```
  WHEN rating >= 1500 THEN 'Gold'
  WHEN rating >= 1350 THEN 'Silver'
  WHEN rating >= 1200 THEN 'Bronze'
  WHEN rating >= 1050 THEN 'Red'
  WHEN rating >= 900  THEN 'Blue'
  ELSE 'Green'
  ```
- `within_band_rank` — rank within each (age_group, band) partition, ascending from 1 (highest-rated team in that band = rank 1)

The `within_band_rank` should use a window function:
```sql
RANK() OVER (PARTITION BY age_group, band ORDER BY rating DESC) AS within_band_rank
```

### Section 3: Validation Queries

Write the three post-creation validation queries Presten runs immediately after creating the view:

**V1 — Row count by age group (sanity check)**
```sql
SELECT age_group, COUNT(*) as team_count FROM rankings_with_bands GROUP BY age_group ORDER BY age_group;
```
Expected: matches counts in `rankings` table. If mismatch, view has a join or filter error.

**V2 — Band distribution for U13G (DSS demo age group)**
```sql
SELECT band, COUNT(*), ROUND(AVG(rating), 1) as avg_rating
FROM rankings_with_bands
WHERE age_group = 'U13G'
GROUP BY band
ORDER BY AVG(rating) DESC;
```
Expected: Gold count should be small (top ~10%), Green count largest. If band counts look inverted, the CASE thresholds are applied wrong.

**V3 — Top 10 U13G with bands**
```sql
SELECT t.name, rwb.rating, rwb.rank, rwb.band, rwb.within_band_rank
FROM rankings_with_bands rwb
JOIN teams t ON t.id = rwb.team_id
WHERE rwb.age_group = 'U13G' AND rwb.rank <= 10
ORDER BY rwb.rank;
```
Expected: all 10 have a non-null band. At least 3 Gold teams in top 10 (if not, Gold threshold may need adjustment — flag for SENTINEL).

### Section 4: Rollback

If the view needs to be dropped:
```sql
DROP VIEW IF EXISTS rankings_with_bands;
```
No data is modified by creating this view — rollback is safe and instant.

### Section 5: Pre-Demo Integration Note

The DSS Data Readiness Validation Plan (Section 1A–1D) checks:
- `SELECT COUNT(*) FROM rankings_with_bands WHERE age_group = 'U13G'` → should return >200 rows
- Football Academy NJ band ≥ Silver (rating ≥1350)

Confirm both of these will be satisfied by the view as written. If Football Academy NJ's rating is below 1350, note this — it doesn't block the view, but changes the demo narrative.

## Definition of Done

- View SQL is complete and runnable (no placeholders, no pseudocode)
- CASE statement uses exact validated thresholds
- `within_band_rank` window function is correct
- All three validation queries are present
- Rollback SQL is included
- Section 5 integration note confirms DSS validation plan consistency
- Summary in briefing: "Rank Bands view SQL written at `Rankings/Rank Bands View — Implementation SQL.md`. Ready for Presten to run."

## Why This Matters

The Rank Bands feature is ELO's highest-visibility DSS deliverable. The design is done. The thresholds are validated. The only thing between design and demo is Presten running the `CREATE VIEW` statement. Write that statement now, before April 28, so it's ready to run at any point — including as a pre-demo safety check on May 14.

## References

- `02 - Tiger Tournaments/Projects/Rankings/Rank Bands — Presten Brief.md` — validated thresholds
- `02 - Tiger Tournaments/Projects/Rankings/Rank Bands — Threshold Validation.md` — statistical backing
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Data Readiness Validation Plan — May 14-15.md` — Sections 1A–1D reference this view
- `10 - Agents/ELO/Tasks/task-2026-04-23-rank-bands-implementation-plan.md` — implementation plan context
