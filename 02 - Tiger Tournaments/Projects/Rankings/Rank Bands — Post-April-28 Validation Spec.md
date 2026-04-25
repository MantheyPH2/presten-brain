---
title: Rank Bands — Post-April-28 Validation Spec
type: validation-spec
author: ELO
date: 2026-04-25
status: ready
related: "[[Rank Bands — Implementation Plan]]", "[[Rank Bands View — Implementation SQL]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]"
tags: [rank-bands, validation, april-28, ga-aspire, calibration, dss, evo-draw]
---

# Rank Bands — Post-April-28 Validation Spec

> **Purpose:** Confirm that rank band thresholds remain correctly calibrated after the April 28 GA ASPIRE fix. The fix raises Girls GA ASPIRE ratings by +8 to +40 pts (per ELO's Pre-Run Model). Teams near band boundaries will cross into higher bands — this is expected and correct. This spec validates whether the threshold positions need revision or hold as designed.

---

## Section 1: Pre-Calibration Baseline

Band thresholds: Gold ≥1500, Silver ≥1350, Bronze ≥1200, Red ≥1050, Blue ≥900, Green <900.

**Pre-fix distribution (from Pre-April-28 Baseline Snapshot if filed; fill from snapshot if available):**

From the implementation plan's expected distribution and threshold validation data (Rank Bands — Threshold Validation, 2026-04-24):

| Band | Expected % (All age groups, all genders) | Girls GA ASPIRE Age Groups Near Boundary |
|------|------------------------------------------|-------------------------------------------|
| Gold | ~1% | Teams rated 1470–1499 may cross into Gold post-fix |
| Silver | ~4–6% | Teams rated 1320–1349 may cross into Silver post-fix |
| Bronze | ~18–22% | Teams rated 1170–1199 may cross into Bronze post-fix |
| Red | ~23–27% | |
| Blue | ~22–26% | |
| Green | ~22–26% | |

**U13G pre-fix distribution (expected, from Rank Bands View validation query V2):**

If the Pre-April-28 Baseline Snapshot has been filed as `Rankings/Pre-April-28 Baseline Snapshot.md`, pull the exact row counts from that document and fill the following table. If not yet filed, mark as `[To fill from snapshot — execute after snapshot is complete]`:

| Band | U13G Team Count (pre-fix) | % of U13G ranked teams (pre-fix) |
|------|--------------------------|----------------------------------|
| Gold | [from snapshot] | [from snapshot] |
| Silver | [from snapshot] | [from snapshot] |
| Bronze | [from snapshot] | [from snapshot] |
| Red | [from snapshot] | [from snapshot] |
| Blue | [from snapshot] | [from snapshot] |
| Green | [from snapshot] | [from snapshot] |

**Girls GA ASPIRE boundary concentration (pre-fix):** Based on the Pre-Run Model, the age groups with the highest GA ASPIRE game share are U14G (peak GA enrollment), U15G, then U13G. Within those age groups, GA ASPIRE teams carry ratings suppressed by ~20–40 pts due to the overcalibration. Teams that would sit at Silver (1350–1499) pre-fix may be at Bronze (1200–1349) pre-fix because of that suppression. This makes the Bronze-to-Silver boundary the highest-risk crossing point for GA ASPIRE teams post-fix.

---

## Section 2: Expected Post-Fix Band Distribution

The April 28 fix raises Girls GA ASPIRE ratings by +5 to +40 pts (most-affected teams: +20 to +40 pts; average across GA ASPIRE cohort: +8 to +20 pts).

**Key judgment: are these threshold crossings expected and correct, or a calibration concern?**

**ELO's answer: expected and correct.** The rank band thresholds were set based on the full Girls rating distribution, not specifically against Girls GA ASPIRE ratings. The April 28 fix corrects an overcalibration that was suppressing Girls GA ASPIRE ratings below where their game results justify. When their ratings increase post-fix, they move to the band that accurately reflects their competitive level. A team that was Bronze (1180 pts) when it should have been Silver (1200–1349) is correctly reassigned to Bronze or low-Silver after the fix. This is the fix working as intended.

**The case for threshold revision would require:** GA ASPIRE teams crossing into Gold at a rate that doesn't reflect their actual national competitive standing — i.e., teams that by any reasonable external benchmark are below Gold caliber appearing at Gold. This would indicate the fix overcorrected. This can be evaluated post-fix by checking whether any newly-Gold GA ASPIRE teams have head-to-head losses to known Silver/Bronze teams that have not yet been recomputed.

**Predicted band distribution change post-fix (Girls, GA ASPIRE-heavy age groups):**

| Age Group | Predicted Band Shifts | Predicted Distribution Change |
|-----------|-----------------------|-------------------------------|
| U13G | ~10–40 girls GA ASPIRE teams cross one band upward | Bronze % decreases slightly (~1–3%), Silver % increases slightly |
| U14G | ~15–50 girls GA ASPIRE teams cross one band upward | Same direction; slightly larger magnitude than U13G |
| U15G | ~12–40 girls GA ASPIRE teams cross one band upward | Similar to U13G |
| U16G | ~5–20 girls GA ASPIRE teams cross one band upward | Smaller effect; less ASPIRE exposure |
| U17G | ~2–10 girls GA ASPIRE teams cross one band upward | Minimal effect |
| Boys (all) | 0 teams cross band | No change expected |

**No threshold revision expected** if distribution shift is ≤ 5 percentage points in any band for any age group.

---

## Section 3: Post-April-28 Validation Queries

Run after April 28 recompute confirms complete (Gate A passed in Post-Fix Decision Tree).

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

Compare to pre-fix baseline from Section 1. Expected: Girls distribution shifts slightly upward (Bronze % decreases, Silver % increases by ~1–3%); Boys distribution unchanged.

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

Focus on U13G, U14G, U15G (highest GA ASPIRE exposure). Compare Silver and Bronze percentages to pre-fix baseline. A Bronze decrease of 1–4% with a corresponding Silver increase is expected and correct.

**Query 3 — Band Boundary Teams (Girls GA ASPIRE-relevant age groups):**

```sql
-- Teams within 15 pts of a band boundary, Girls U13G–U16G
-- These are the teams most likely to have changed bands post-fix
SELECT
  age_group,
  band,
  COUNT(*) AS teams_near_boundary,
  MIN(rating) AS band_floor_in_cohort,
  MAX(rating) AS band_ceiling_in_cohort
FROM rankings_with_bands
WHERE gender = 'F'
  AND age_group IN ('U13', 'U14', 'U15', 'U16')
  AND (
    -- Within 15 pts above Gold threshold (1500)
    (rating BETWEEN 1500 AND 1515) OR
    -- Within 15 pts below Gold (might have been here pre-fix)
    (rating BETWEEN 1485 AND 1499) OR
    -- Within 15 pts around Silver threshold (1350)
    (rating BETWEEN 1350 AND 1365) OR
    (rating BETWEEN 1335 AND 1349) OR
    -- Within 15 pts around Bronze threshold (1200)
    (rating BETWEEN 1200 AND 1215) OR
    (rating BETWEEN 1185 AND 1199)
  )
GROUP BY age_group, band
ORDER BY age_group, band;
```

A high count at the Bronze/Silver boundary (1185–1365) for Girls is expected post-fix and not alarming — these are the GA ASPIRE teams that moved. Review for obviously wrong results: if a well-known ECNL or MLS NEXT-adjacent Girls team appears in Green, the fix may have undercorrected; if a team known to be a developmental program appears in Gold, investigate for merge artifacts.

---

## Section 4: Recalibration Decision Criteria

**No revision needed** if all of the following hold after Query 1 and Query 2:
- For any single band, the % of Girls teams in that band changes by ≤ 5 percentage points vs. pre-fix baseline
- Boys band distribution is unchanged (0% change in any band)
- Gold band for Girls does not exceed 3% of Girls ranked teams (pre-fix Gold was ~1–2%; a post-fix Gold of 3% is explainable by suppressed GA ASPIRE teams being correctly elevated)

**Review required** if:
- Any Girls band changes by more than 5 percentage points. ELO investigates the specific teams driving the shift and files a review note in the next briefing before DSS readiness check.

**Recalibration needed** if:
- Gold band for Girls exceeds 5% of Girls ranked teams — suggests the fix overcorrected or a calibration interaction moved non-GA-ASPIRE teams into Gold unexpectedly
- Silver band for Girls increases by > 8 percentage points — larger shift than GA ASPIRE population can explain; suggests a wider rating inflation beyond the fix scope
- Any Boys band changes by > 1% — should be exactly 0; any Boys band change is anomalous

The N value for the "review required" criterion is **5 percentage points**. ELO sets this based on the estimated size of the GA ASPIRE-affected population: ~150–500 teams out of the total Girls population of ~1,500–2,000 ranked Girls teams. A 500-team shift represents ~25–33% of Girls — that would be a 5+ percentage point band change at the Bronze/Silver boundary. Such a shift is possible if GA ASPIRE coverage is at the high end of ELO's estimates; it requires investigation but does not automatically trigger recalibration.

---

## Section 5: Recalibration Protocol (If Triggered)

**If recalibration criteria are met:**

**Which thresholds need revision?** If Gold > 5% of Girls: review the Gold threshold only. The Gold threshold (≥1500) is the most exposure-sensitive to the fix because highly-rated GA ASPIRE teams sit closest to it. The Silver/Bronze thresholds are less likely to need adjustment — the GA ASPIRE population is large but its average correction (+8 to +20 pts) moves most teams across Bronze/Silver, not Silver/Gold.

**Recalibration method:** Re-run the pre-flight distribution query from the Implementation Plan against the post-fix ratings:

```sql
SELECT
  age_group, gender,
  PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY rating)::int AS p90,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY rating)::int AS p95,
  PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY rating)::int AS p99,
  COUNT(*) AS team_count
FROM rankings
WHERE rank IS NOT NULL
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

If the post-fix p95 for Girls is substantially different from the pre-fix p95, the Gold threshold should shift by the same delta (Threshold Contingency formula from the Implementation Plan: delta = actual_p95 − expected_p95, apply delta uniformly to all thresholds).

**Timeline:** If recalibration is triggered by the April 29 analysis session, ELO can compute new thresholds in < 30 minutes using the Threshold Contingency formula. New thresholds can be applied to the `rankings_with_bands` view by updating the CASE statement — no schema migration. New view can be created in the same session.

**Can recalibration be completed before May 9?** Yes, if triggered by April 29 analysis. ELO recalibrates by May 3, SENTINEL reviews by May 7, Presten updates the view before May 9 go/no-go.

**If Girls-only threshold adjustment is needed:** Apply the updated Gold threshold only to the Girls CASE branch in the view, not to Boys. Boys thresholds remain unchanged (Boys distribution is not affected by the fix).

**Fallback if recalibration cannot complete before May 9:**
The demo uses the current (post-fix, pre-recalibration) bands with a SENTINEL note: "Rank Bands are calibrated and live. The April 28 fix moved a small number of Girls GA ASPIRE teams to higher bands — this is expected and correct. ELO is in the process of confirming whether the top threshold (Gold) needs a minor adjustment." This is not a blocker for DSS unless Gold exceeds 5% — at that point the Gold band claim is not defensible without explanation.

---

## Definition of Done

- Spec filed at `Rankings/Rank Bands — Post-April-28 Validation Spec.md`
- All 5 sections present
- Three validation queries written with explicit column references from `rankings_with_bands`
- Recalibration criteria use explicit percentage thresholds (5% for review, 5%/8%/1% for recalibration triggers)
- ELO files a one-line summary in the next briefing after April 29 analysis:
  > "Rank bands post-fix validation: [no revision needed / review underway / recalibration triggered]. Distribution shift: Girls Bronze [+/−N%], Girls Silver [+/−N%], Girls Gold [+/−N%], Boys all bands [+/−0%]."

---

## References

- [[Rank Bands — Implementation Plan]] — band design, threshold values, and Threshold Contingency formula
- [[Rank Bands View — Implementation SQL]] — the SQL view this spec validates (column names: `band`, `within_band_rank`)
- [[Post-April-28 Expected Rating Shift — Pre-Run Model]] — predicted rating shifts that affect band placement (Section 3 magnitude estimates)
- `Rankings/Pre-April-28 Baseline Snapshot.md` — pre-fix distribution (fill Section 1 from here once snapshot is complete)
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — rank bands feature cell this spec validates

*ELO — 2026-04-25*
