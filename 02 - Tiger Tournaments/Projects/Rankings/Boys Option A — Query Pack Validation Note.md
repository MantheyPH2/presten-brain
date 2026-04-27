---
title: Boys Option A — Query Pack Validation Note
type: validation-note
author: ELO
date: 2026-04-27
status: cleared
related: "[[Boys Option A Spot Check — Presten Execution Package]]", "[[Boys Option A — Decision Document]]"
tags: [boys, calibration, dss, validation, evo-draw]
---

# Boys Option A — Query Pack Validation Note

> **Purpose:** Confirms that the 3 queries in the execution package are aligned with the Decision Document tables and are valid for post-April-28 DB state. Filed before April 29 session.

---

## Section 1: Query Inventory

| Query | Description | Output Columns | Filters |
|-------|-------------|----------------|---------|
| Query 1 | Boys vs. Girls average rating by age group | `age_group, gender, team_count, avg_rating, median_rating` | age_group IN (U13–U17), gender IN ('M','F'), rank IS NOT NULL |
| Query 2 | Boys GA game volume by age group | `age_group, boys_ga_games, teams_with_ga_games` | gender = 'M', age_group IN (U13–U17), event_tier IN ('ga','ga_aspire'), rank IS NOT NULL |
| Query 3 | MLS NEXT Boys rating distribution | `age_group, mlsnext_team_count, avg_rating, median_rating, top_10pct_rating, bottom_10pct_rating, max_rating` | gender = 'M', age_group IN (U13–U17), event_tier IN ('mls_next_homegrown','mls_next_academy','mls_next'), rank IS NOT NULL |

---

## Section 2: Decision Document Alignment Check

| Query | Decision Doc Section | Output Column Match? | Row Structure Match? | Notes |
|-------|---------------------|---------------------|---------------------|-------|
| Query 1 | Section 1 — Query 1 Results Table | ✅ | ⚠️ | Query returns 10 rows (5 age groups × 2 genders). Decision Document expects 5 rows with Boys and Girls as separate columns. ELO must pivot the output: `avg_rating` where gender='M' → Boys Avg Rating; where gender='F' → Girls Avg Rating. `median_rating` column is extra — not needed for Decision Document. ELO fills the Delta column by subtraction. This is a straightforward manual pivot, not a data mismatch. |
| Query 2 | Section 1 — Query 2 Results Table | ✅ | ✅ | `boys_ga_games` → "Boys GA Games in DB". 5 rows, one per age group. Decision Document uses `U13B/U14B` format vs. query result `U13/U14` — ELO appends 'B' when filing (gender filter is already applied in WHERE clause). `teams_with_ga_games` column is extra — useful context, not required by Decision Doc. |
| Query 3 | Section 1 — Query 3 Results Table | ✅ | ✅ | All columns map directly: `mlsnext_team_count` → "MLS NEXT Team Count", `avg_rating` → "Avg Rating", `median_rating` → "Median", `top_10pct_rating` → "Top 10%", `bottom_10pct_rating` → "Bottom 10%", `max_rating` → "Max". 5 rows, one per age group. Clean slot-in. |

**Column match note:** Query 1 is the only case requiring interpretation. The data is complete — the query returns all values ELO needs. The pivot is ELO's step, not Presten's. Presten saves the raw 10-row file; ELO pivots when filling the Decision Document.

---

## Section 3: Post-April-28 Filter Validity

| Query | References event_tier? | References ecnl_verified? | Post-April-28 Filter Valid? | Notes |
|-------|----------------------|--------------------------|---------------------------|-------|
| Query 1 | No | No | ✅ | Filters only on age_group, gender, rank IS NOT NULL. Not affected by April 28 changes. |
| Query 2 | Yes — `IN ('ga', 'ga_aspire')` | No | ✅ | **Key finding:** After April 28, the fix moves Girls ASPIRE events from `event_tier = 'ga'` to `event_tier = 'ga_aspire'`. Query 2 uses `IN ('ga', 'ga_aspire')` for Boys — this correctly captures all Boys GA games in both pre-fix and post-fix state. Boys GA games were never classified as 'ga_aspire' by the April 28 fix (that fix targeted Girls ASPIRE games only), so including 'ga_aspire' in the IN clause is forward-safe and does not introduce risk. Boys teams that played in any ASPIRE-named event would now be correctly retrieved under 'ga_aspire' if tagged. |
| Query 3 | Yes — `IN ('mls_next_homegrown', 'mls_next_academy', 'mls_next')` | No | ✅ | Filters on MLS NEXT tiers only. No overlap with GA ASPIRE fix. No ecnl_verified dependency. Valid for post-April-28 DB state. |

**GA ASPIRE Boys scope confirmation:** The April 28 fix reclassified Girls ASPIRE events from `event_tier = 'ga'` to `event_tier = 'ga_aspire'`. Boys ratings should shift 0 points as a result. If Boys ratings shift materially post-April-28, it is a diagnostic signal of unintended scope — not a query design problem. Query 2's dual-tier filter (`IN ('ga', 'ga_aspire')`) is correct and intentional.

**ecnl_verified field:** None of the three Boys Option A queries reference `ecnl_verified`. The April 28 schema change (adding `ecnl_verified` column, default FALSE) has no impact on Boys Option A query validity. All three queries will run correctly against the post-April-28 schema.

---

## Section 4: Required Corrections

No corrections required.

All three queries:
- Produce output that fills the Decision Document tables (Query 1 via ELO pivot; Queries 2–3 directly)
- Are valid for post-April-28 DB state
- Do not reference schema fields that change meaning after the April 28 fix in ways that break result interpretation

No edits to `Rankings/Boys Option A Spot Check — Presten Execution Package.md` are needed before April 29.

---

## Section 5: ELO Clearance

```
Boys Option A query pack cleared for April 29 execution: YES
Last reviewed: 2026-04-27

Note for ELO when filling Decision Document from Query 1 output:
  - 10-row output → pivot to 5-row table
  - Boys row: gender = 'M', avg_rating → Boys Avg Rating
  - Girls row: gender = 'F', avg_rating → Girls Avg Rating
  - Delta = Boys Avg Rating − Girls Avg Rating (per age group)
  - median_rating column not needed for Decision Document; use as context only
```

*ELO — 2026-04-27*
