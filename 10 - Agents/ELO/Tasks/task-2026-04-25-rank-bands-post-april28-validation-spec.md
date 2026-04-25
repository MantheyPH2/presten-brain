---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-25
priority: medium
due: 2026-05-07
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Rank Bands — Post-April-28 Validation Spec.md"
---

# Task: Rank Bands — Post-April-28 Validation Spec

## Context

The Rank Bands implementation (`Rankings/Rank Bands — Implementation Plan.md` and the view SQL filed 2026-04-24) was calibrated against pre-April-28 rating distributions. April 28 makes a material change: GA ASPIRE event_tier fix shifts Girls GA ASPIRE ratings by an estimated +15–40 pts (per ELO's Pre-Run Model).

If Girls GA ASPIRE ratings shift upward by ~20 pts on average, teams currently sitting near rank band boundaries will cross into the next band. That is expected and correct behavior — the fix is supposed to move these teams. But it raises a calibration question: were the rank band thresholds designed to accommodate this shift, or were they calibrated against the inflated (pre-fix) Girls GA ASPIRE ratings?

If the thresholds were calibrated against inflated values, the post-April-28 rank bands may assign too many Girls GA ASPIRE teams to the top band and too few to the lower bands — overclaiming at the top due to a corrected but now-above-threshold rating.

This task: write the spec for validating rank band calibration after April 28 runs.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Rank Bands — Post-April-28 Validation Spec.md`

---

### Section 1: Pre-Calibration Baseline

Before April 28 runs (ELO writes this now, or pulls from the Pre-April-28 Baseline Snapshot when it's filed):

For each rank band threshold, record the pre-fix distribution:
- What % of Girls teams fall in each band?
- What % of Boys teams fall in each band?
- Any age groups where Girls GA ASPIRE teams cluster heavily near a band boundary (these will be most affected post-fix)?

If the Pre-April-28 Baseline Snapshot has been filed (`Rankings/Pre-April-28 Baseline Snapshot.md`), pull from there. If not yet filed as of this task, mark this section as `[To fill from baseline snapshot — run after snapshot is complete]`.

---

### Section 2: Expected Post-Fix Band Distribution

After the GA ASPIRE fix (+15–40 pts for Girls GA ASPIRE teams), what should the rank band distribution look like?

For each Girls age group with significant GA ASPIRE exposure:
- Estimate how many teams cross upward across a band threshold
- Does this materially change the % distribution? (e.g., Platinum band goes from 8% to 12% of Girls teams)
- Is this distribution change **expected and correct** (the fix worked) or **a calibration concern** (bands need threshold revision)?

State ELO's answer. This is the key judgment call in this spec.

---

### Section 3: Post-April-28 Validation Queries

Three queries ELO runs after April 28 (as part of the April 29 analysis session or in the week following):

**Query 1 — Band Distribution by Gender:**
```sql
-- Returns: count and % of teams per rank band, broken out by gender
-- Compare to pre-fix baseline from Section 1
-- Expected: Girls distribution shifts slightly upward; Boys unchanged
SELECT rank_band, gender, COUNT(*) AS team_count,
       ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (PARTITION BY gender), 1) AS pct
FROM rank_bands_view
GROUP BY rank_band, gender
ORDER BY gender, rank_band;
```

**Query 2 — Band Distribution by Age Group (Girls Only):**
```sql
-- Returns: rank band distribution for Girls by age group
-- Focus on age groups with high GA ASPIRE exposure per ELO's Pre-Run Model
SELECT age_group, rank_band, COUNT(*) AS team_count
FROM rank_bands_view
WHERE gender = 'F'
GROUP BY age_group, rank_band
ORDER BY age_group, rank_band;
```

**Query 3 — Band Boundary Teams:**
```sql
-- Returns: teams within 10 pts of a band boundary on either side, Girls GA ASPIRE age groups only
-- These are the teams most likely to have changed bands post-fix
-- High count here is expected; review for any obviously wrong results (e.g., well-known ECNL teams in a lower band)
-- ELO writes the exact threshold check once band boundaries are confirmed
```

Confirm the exact column names and view name from the rank bands view implementation SQL.

---

### Section 4: Recalibration Decision Criteria

Under what conditions does ELO flag rank band thresholds as needing revision post-April-28?

Define specific criteria:
- **No revision needed:** Post-fix distribution is within [N]% of pre-fix distribution for any band. The shift is explained by the rating correction.
- **Review required:** One band's distribution changes by more than [N]% of teams. ELO investigates cause before DSS.
- **Recalibration needed:** Band thresholds were derived from inflated Girls GA ratings and now systematically misclassify Girls GA teams (e.g., below-average Girls GA teams in Platinum band). Requires threshold revision before DSS.

ELO sets the N values and files a verdict in the briefing.

---

### Section 5: Recalibration Protocol (If Triggered)

If recalibration is needed:
- Which thresholds need revision (Girls-specific, or all age groups)?
- What is the recalibration method (re-fit to post-fix distribution, manual threshold adjustment, or percentile-based re-anchor)?
- Timeline: can recalibration be completed before May 9 DSS readiness check?
- If not, what is the fallback? (Demo uses current bands with caveat, or demo defers rank bands feature)

---

## Definition of Done

- Spec written at `Rankings/Rank Bands — Post-April-28 Validation Spec.md`
- All 5 sections present
- Three validation queries written (Query 3 may be marked TBD if boundary values are pending)
- Recalibration criteria are explicit percentages, not "significant change"
- ELO files a one-line summary in the next briefing after April 29 analysis: "Rank bands post-fix validation: [no revision needed / review underway / recalibration triggered]. Distribution shift: [summary]."

## Why This Matters

The rank bands feature is planned for the DSS demo. If rank band thresholds are silently miscalibrated post-April-28 (because they were set against inflated Girls GA ratings), the DSS demo shows incorrect band assignments for a meaningful portion of Girls teams. No one catches it without a validation step. This spec ensures there is a validation step with explicit pass/fail criteria before May 9.

## References

- `Rankings/Rank Bands — Implementation Plan.md` — band design and threshold values
- `Rankings/Rank Bands — View Implementation SQL.md` — the SQL view this spec validates
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — predicted rating shifts that affect band placement
- `Rankings/Pre-April-28 Baseline Snapshot.md` — pre-fix distribution (baseline for comparison)
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — rank bands feature cell
