---
title: Post-Fix Rating Distribution Analysis Spec — 2026-04-27
type: analysis-spec
author: ELO
date: 2026-04-27
status: template-ready
related: "[[April 29 Gate Results — Structured Log]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]", "[[April 29 ELO Session Master Script]]"
tags: [calibration, ga-aspire, post-fix, analysis, april-29, evo-draw]
---

# Post-Fix Rating Distribution Analysis Spec — 2026-04-27

> **Purpose:** Provide a systematic analysis framework so April 29 produces criteria-driven conclusions rather than narrative observations. ELO fills in the "Actual" columns on April 29 based on Presten's query output. Estimated time to execute on April 29: < 20 minutes.

> **Pre-committed predictions:** All "Expected" values are derived from `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` (written 2026-04-25, before April 28 execution). Do NOT modify expected columns on April 29.

---

## Baseline Reference

| Source | Purpose |
|--------|---------|
| Pre-April-28 snapshot (Presten execution package) | Pre-fix baseline for all comparison queries |
| April 28 execution log | Confirms recompute ran and scope |
| Pre-Run Model (2026-04-25) | Source of all expected delta values below |

> **Note on snapshot:** These queries are designed to run against the post-recompute live `rankings` table, compared against the pre-April-28 snapshot. The snapshot source table is `rankings_snapshot` (created by Presten's April 27 pre-fix baseline capture). If Presten did not capture a snapshot before April 28, see the fallback note at the end of this document.

---

## Section 1: Pre-Fix Baseline Query

Run this against the pre-April-28 snapshot to establish the baseline for ASPIRE-heavy teams. An "ASPIRE-heavy team" is defined as any team with ≥3 distinct ASPIRE-tagged events in their game history.

```sql
-- Pre-fix baseline: average rating for ASPIRE-heavy teams by age group
-- Run against rankings_snapshot (pre-April-28)
SELECT
  r.age_group,
  ROUND(AVG(r.rating), 1)    AS pre_fix_avg_rating,
  COUNT(DISTINCT r.team_id)  AS aspire_heavy_team_count
FROM rankings_snapshot r
WHERE r.team_id IN (
  SELECT te.team_id
  FROM team_events te
  JOIN events e ON e.id = te.event_id
  WHERE e.event_tier IN ('ga_aspire', 'ga')
    AND e.event_name ILIKE '%ASPIRE%'
  GROUP BY te.team_id
  HAVING COUNT(DISTINCT e.id) >= 3
)
GROUP BY r.age_group
ORDER BY r.age_group;
```

Expected output: ratings for U13G–U17G (Girls age groups where GA ASPIRE exposure is concentrated).

---

## Section 2: Post-Fix Comparison Query

Run this against the post-April-28 recompute (live `rankings` table). Same ASPIRE-heavy team definition as Section 1.

```sql
-- Post-fix comparison: average rating for ASPIRE-heavy teams by age group
-- Run against live rankings (post-April-28 recompute)
SELECT
  r.age_group,
  ROUND(AVG(r.rating), 1)    AS post_fix_avg_rating,
  COUNT(DISTINCT r.team_id)  AS aspire_heavy_team_count
FROM rankings r
WHERE r.team_id IN (
  SELECT te.team_id
  FROM team_events te
  JOIN events e ON e.id = te.event_id
  WHERE e.event_tier IN ('ga_aspire', 'ga')
    AND e.event_name ILIKE '%ASPIRE%'
  GROUP BY te.team_id
  HAVING COUNT(DISTINCT e.id) >= 3
)
GROUP BY r.age_group
ORDER BY r.age_group;
```

Delta = post_fix_avg_rating − pre_fix_avg_rating per age group.

---

## Section 3: Expected Delta Table

All "Expected Delta" and "Confidence" values are pre-committed from the Pre-Run Model. Do not modify on April 29.

| Age Group | Pre-Fix Avg (fill April 29) | Post-Fix Avg (fill April 29) | Actual Delta | Expected Delta | In Range? |
|-----------|----------------------------|------------------------------|-------------|----------------|-----------|
| U13G | ___ | ___ | ___ | **+10 to +40 pts** | PASS / FAIL |
| U14G | ___ | ___ | ___ | **+10 to +40 pts** | PASS / FAIL |
| U15G | ___ | ___ | ___ | **+10 to +35 pts** | PASS / FAIL |
| U16G | ___ | ___ | ___ | **+5 to +25 pts** | PASS / FAIL |
| U17G | ___ | ___ | ___ | **+5 to +20 pts** | PASS / FAIL |

*Note: These are ASPIRE-heavy team averages, not full age-group cohort averages. The full cohort averages (from G2 in the Gate Results Structured Log) will be smaller — the ASPIRE-heavy teams are the concentrated signal.*

---

## Section 4: Interpretation Thresholds

For each age group row in Section 3, apply these criteria:

| Result | Classification | Action |
|--------|---------------|--------|
| Actual delta within expected range | **CONFIRMED** | Calibration fix applied as expected. Proceed. |
| Actual delta > upper bound (e.g., > +40 for U13G) | **FLAGGED** | Rating shift larger than predicted. Investigate whether ASPIRE event count threshold is capturing more teams than expected. Flag to SENTINEL. Hold Rank Bands session until investigated. |
| Actual delta < lower bound but still positive (e.g., +3 for U13G) | **INVESTIGATE** | Shift is positive but smaller than predicted. Confirm April 28 fix fully applied by checking G1 gate results. If G1 passed, log as a Pre-Run Model calibration miss — note for future pre-run accuracy. |
| Actual delta = 0 or < 0 (rating went DOWN or did not move) | **CRITICAL FLAG** | Fix either did not apply or applied in the wrong direction. Escalate immediately. Do not proceed with Event Strength Phase 1 or Boys Option A gates until this is resolved. |

---

## Section 5: Boys Check

**Expected: 0 ± 5 pts for all Boys age groups. Any shift > 10 pts is unexpected and must be explained before April 29 gates proceed.**

```sql
-- Boys cohort average rating — post-April-28
-- Compare against pre-April-28 snapshot (same query against rankings_snapshot)
SELECT
  r.age_group,
  ROUND(AVG(r.rating), 1) AS avg_rating,
  COUNT(*)                 AS team_count
FROM rankings r
WHERE r.age_group ILIKE '%B'
GROUP BY r.age_group
ORDER BY r.age_group;
```

| Age Group | Pre-Fix Avg | Post-Fix Avg | Actual Delta | Expected | PASS / FAIL |
|-----------|-------------|-------------|-------------|----------|-------------|
| U13B | ___ | ___ | ___ | 0 ± 5 pts | |
| U14B | ___ | ___ | ___ | 0 ± 5 pts | |
| U15B | ___ | ___ | ___ | 0 ± 5 pts | |
| U16B | ___ | ___ | ___ | 0 ± 5 pts | |
| U17B | ___ | ___ | ___ | 0 ± 5 pts | |

**If any Boys age group delta > 5 pts:** Mandatory investigation. The April 28 recompute scope should have been Girls-only or GA-tier-only. A Boys shift > 5 pts implies the recompute ran with broader scope than expected. Document the finding and hold G3 in the Gate Results Structured Log until explained.

**If any Boys age group delta > 15 pts:** Hard escalation. Do not authorize Event Strength Phase 1 or Boys Option A until root cause is confirmed.

---

## Section 6: Non-ASPIRE Teams Sanity Check

Select 3 stable, well-established teams with no ASPIRE event history. Confirm their ratings did not shift more than ±5 pts.

```sql
-- Sanity check: stable teams that have never appeared in an ASPIRE event
-- ELO selects 3 known stable reference teams based on prior knowledge
-- Suggested candidates: top-ranked ECNL or MLS NEXT Homegrown teams with 3+ years of data

SELECT
  r.team_id,
  r.team_name,
  r.age_group,
  r.rating             AS post_fix_rating,
  s.rating             AS pre_fix_rating,
  (r.rating - s.rating) AS delta
FROM rankings r
JOIN rankings_snapshot s ON s.team_id = r.team_id
                         AND s.age_group = r.age_group
WHERE r.team_id IN (
  [INSERT 3 KNOWN STABLE TEAM IDs]
);
```

| Team | Age Group | Pre-Fix Rating | Post-Fix Rating | Delta | Flag? |
|------|-----------|---------------|----------------|-------|-------|
| [Stable Ref 1] | | ___ | ___ | ___ | Delta > ±5 → FLAG |
| [Stable Ref 2] | | ___ | ___ | ___ | Delta > ±5 → FLAG |
| [Stable Ref 3] | | ___ | ___ | ___ | Delta > ±5 → FLAG |

**If any reference team shifted > ±5 pts:** The recompute had unintended scope — it changed ratings for teams with no connection to the GA ASPIRE fix. Investigate recompute parameters before proceeding. If all three are stable (delta ≤ ±5), the recompute was correctly scoped.

---

## Section 7: April 29 Summary Classification

Fill in after completing Sections 3–6:

```
Girls ASPIRE-Heavy Delta Check:
  U13G: [IN RANGE / FLAGGED / INVESTIGATE / CRITICAL]
  U14G: [IN RANGE / FLAGGED / INVESTIGATE / CRITICAL]
  U15G: [IN RANGE / FLAGGED / INVESTIGATE / CRITICAL]
  U16G: [IN RANGE / FLAGGED / INVESTIGATE / CRITICAL]
  U17G: [IN RANGE / FLAGGED / INVESTIGATE / CRITICAL]

Boys Unchanged Check:
  All Boys age groups 0 ± 5: [YES / NO — explain if NO]

Non-ASPIRE Sanity Check:
  All 3 stable teams within ±5 pts: [YES / NO — explain if NO]

Overall spec result: [CONFIRMED / FLAGGED / CRITICAL]
Note for Gate Results Structured Log:
  G2/G3 input from this spec: [summarize key values for gate log]
```

---

## Fallback: If No Pre-April-28 Snapshot Exists

If Presten did not run the pre-fix snapshot before April 28, the direct pre/post comparison is not possible. Fallback options:

1. **Use the Pre-Run Model absolute values** as the pre-fix baseline. The Pre-Run Model includes estimated pre-fix average ratings by age group. Delta = post-fix actual − pre-run model estimate. This is less precise but directionally valid.

2. **Use the April 26 briefing's rating distribution data** as a proxy baseline, if ELO filed rating distributions in a prior session. Note the date gap and apply conservatively.

3. **If neither exists:** Mark pre-fix baseline as unavailable. Proceed with Boys check and non-ASPIRE sanity check only (Sections 5 and 6). Note in the Gate Results Structured Log that G4 baseline comparison is incomplete.

---

## Related Notes

- `Rankings/April 29 Gate Results — Structured Log.md` — G2 and G3 values for this spec feed directly into that log
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — source of all expected delta values
- `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md` — snapshot source
- [[Ranking Engine]] — Phase 8 calibration, GA ASPIRE tier context

*ELO — 2026-04-27 | Task: task-2026-04-27-post-fix-rating-distribution-analysis-spec*
