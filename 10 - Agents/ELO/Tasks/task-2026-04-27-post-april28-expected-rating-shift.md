---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: urgent
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md"
---

# Task: Post-April-28 Expected Rating Shift — Pre-Run Model

## Context

ELO filed `Rankings/Post-April-28 Rating Shift Analysis Spec.md` — the methodology for analyzing rating changes after April 28 runs. ELO also filed `Rankings/April 29 Post-Fix Decision Tree.md` — the G1–G4 gate framework.

What neither document contains: ELO's **prediction** of what the fix should produce. The April 29 analysis compares actual output against... what, exactly? Without a pre-run prediction, ELO's April 29 analysis is reactive description ("ratings moved X points") rather than calibration science ("ratings moved X points, which is consistent / inconsistent with the predicted effect of removing GA ASPIRE overcalibration from Girls ratings").

Writing the prediction **before** April 28 runs turns the April 29 analysis into a real test. It also forces ELO to reason through the mechanism now, when there is no time pressure.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md`

This document is ELO's signed prediction. Written April 27 (or before April 28). Compared against actual results April 29.

---

### Section 1: Mechanism Summary

One paragraph: what does the GA ASPIRE event_tier fix actually change about the rating computation?

Specifically:
- Before the fix: GA ASPIRE events had `event_tier = 'ga'` for Girls teams. Calibration value for `ga` = 100.
- After the fix: GA ASPIRE events get `event_tier = 'ga_aspire'`. Calibration value = [ELO states the correct cal value from the League Hierarchy].
- Net effect: games in GA ASPIRE events now use a [higher / lower] K-factor for Girls teams. This means each game contributes [more / less] to rating changes.

State whether this fix is expected to:
- Raise or lower average Girls ratings in GA-dominant age groups
- Narrow or widen the spread between Girls GA teams and Girls MLS NEXT/ECNL teams
- Have any effect on Boys ratings (should be zero — GA ASPIRE is a Girls-only overcalibration; confirm this)

---

### Section 2: Affected Population Estimate

How many teams and games are affected by this fix?

Pull from `Rankings/GA Coverage Audit Results.md` (filed 2026-04-23) and any available game volume data:
- Estimated count of Girls GA ASPIRE games in the current season
- Estimated count of Girls teams that have ≥ 10 games tagged as GA ASPIRE
- Age groups with the highest GA ASPIRE game share (these will see the largest rating changes)

If exact counts are unavailable, provide ranges from the coverage audit data. A "rough model" based on estimated game share is better than no prediction.

---

### Section 3: Predicted Rating Shift Ranges

For each Girls age group with significant GA ASPIRE exposure:

| Age Group | Estimated % of Games Tagged GA ASPIRE | Predicted Direction | Predicted Magnitude (pts) |
|-----------|---------------------------------------|---------------------|--------------------------|
| U13G | | | |
| U14G | | | |
| U15G | | | |
| U16G | | | |
| U17G | | | |

**Calibration:** The Girls GA overcalibration case was described as "40+ points of distortion" in previous briefings. Use this as the upper bound for the most-exposed teams. Teams with fewer GA ASPIRE games will shift less. State the formula ELO is using to derive the magnitude estimate.

**Boys prediction:** Boys ratings should shift 0 points. Any Boys rating shift post-April-28 is a signal of an unintended change. State this explicitly as a diagnostic assertion.

---

### Section 4: Distribution Shift Predictions

Beyond individual team shifts, what should the aggregate distribution look like post-fix?

- Girls GA ASPIRE average rating: expected to move [up / down] by approximately [N] points
- Girls non-GA ASPIRE average rating: expected to move [minimally / not at all] because [reasoning]
- Expected convergence / divergence between Girls GA and Girls ECNL/MLS NEXT average ratings

This section feeds the April 29 Decision Tree G2 gate (rating distribution check).

---

### Section 5: Anomaly Detection Thresholds

What results from the April 29 analysis would indicate the fix did NOT work as expected?

| Signal | Expected | If Outside Range: Interpretation |
|--------|----------|----------------------------------|
| Girls GA ASPIRE avg rating change | [predicted direction, N pts] | If opposite direction → UPDATE may have failed |
| Boys avg rating change | 0 pts | If > 5 pts → recompute had unintended scope |
| Girls ECNL avg rating change | < 5 pts | If > 10 pts → fix scope exceeded GA ASPIRE |
| Total teams with > 50-pt shift | [estimate] | If much higher → cal value change larger than expected |

These become ELO's G2 pass/fail criteria on April 29.

---

## Quality Standard

- This is a **prediction document**, not a result document. Do not write hedged language that avoids committing to a direction. ELO should state its best estimate with reasoning.
- If ELO is uncertain about the cal value for `ga_aspire`, check the League Hierarchy and state the value used. Do not leave it blank.
- The anomaly detection thresholds must be explicit numbers, not ranges so wide they would pass any outcome.
- Mark this document with a header: `[Pre-Run Prediction — Written before April 28 execution]` so it is clear this was not reverse-engineered from results.

## Why This Matters

The April 29 analysis is only meaningful if ELO has something to compare against. Without a pre-run model, "ratings shifted 25 points" is just a number — ELO cannot say whether it is expected, surprising, or alarming. With a pre-run model, "ratings shifted 25 points, consistent with the predicted 20–30 point correction" is a confident calibration statement SENTINEL can relay to Presten with confidence. Writing this document now, before April 28, is the only way to make that statement honest.

## References

- `Rankings/Post-April-28 Rating Shift Analysis Spec.md` — analysis methodology
- `Rankings/April 29 Post-Fix Decision Tree.md` — G2 gate this model informs
- `Rankings/GA Coverage Audit Results — 2026-04-23.md` — affected game volume data
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys baseline (should not change)
- `Rankings/League Hierarchy.md` — cal values including ga_aspire
