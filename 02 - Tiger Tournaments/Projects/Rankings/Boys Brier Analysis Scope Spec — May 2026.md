---
title: Boys Brier Analysis Scope Spec — May 2026
type: analysis-scope
author: ELO
date: 2026-04-24
status: ready
related: "[[Boys GA ASPIRE Calibration Decision — April 2026]]", "[[League Hierarchy]]"
tags: [boys, ga-aspire, brier, calibration, post-dss, evo-draw]
---

# Boys Brier Analysis Scope Spec — May 2026

> Scope document for the post-DSS Boys GA ASPIRE Brier analysis. Written now so execution can begin immediately after DSS completes. Do not start this analysis before May 17.

---

## Section 1: Goal

The Girls GA ASPIRE calibration fix (140 → 100, executed April 28) was driven by Brier score analysis showing overcalibration relative to ECNL Girls opponents. Boys GA ASPIRE is currently calibrated at 100 — the same as Boys GA — by conservative April 28 decision. The rationale: Boys GA already sits 40 points below Girls GA (100 vs 140), so Boys GA ASPIRE at 100 may not be overcalibrated the way Girls GA ASPIRE at 140 was.

This analysis answers the question: **Is Boys GA ASPIRE correctly calibrated at 100, or should it be lower (~70)?**

Specifically: do predictions for Boys GA ASPIRE vs Boys GA cross-tier games improve when Boys GA ASPIRE calibration is reduced from 100 to ~70? If yes, and the improvement exceeds the significance threshold below, revise Boys GA ASPIRE to 70. If no, retain 100 and close the question.

---

## Section 2: Data Requirements

### Minimum game count for statistical validity

The Girls GA ASPIRE Brier analysis used a held-out set of approximately 1,200–1,500 cross-tier games (`calibration-held-out-validation-2026-04-23.md`). For Boys GA ASPIRE, the minimum usable sample is **300 cross-tier games** — smaller than Girls due to the more targeted question (one calibration value vs six parameters). Below 300, variance in the Brier score estimate will exceed the expected signal.

### Data filter

```sql
-- Count available Boys GA ASPIRE games (run this first)
SELECT
  COUNT(*) AS boys_ga_aspire_games,
  COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) AS teams_participating
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_tier = 'ga_aspire'
  AND g.gender = 'M';
```

Run this after the April 28 fix executes (Boys GA ASPIRE events will then have `event_tier = 'ga_aspire'`).

### Cross-tier games requirement

The Brier analysis requires Boys GA ASPIRE teams playing against Boys GA or Boys MLS NEXT teams in the same games — otherwise there's no cross-calibration signal.

```sql
-- Count cross-tier games where Boys GA ASPIRE teams face Boys GA or MLS NEXT teams
SELECT COUNT(*) AS cross_tier_games
FROM games g
JOIN events e ON e.id = g.event_id
WHERE g.gender = 'M'
  AND (
    (e.event_tier = 'ga_aspire' AND EXISTS (
      SELECT 1 FROM rankings r WHERE r.team_id = g.away_team_id
        AND r.age_group = g.age_group
    ))
    -- Simplified: refine with actual cross-tier join if needed
  );
```

If cross-tier game count < 100, the analysis is data-limited. Recommend deferring until more Boys GA ASPIRE data accumulates — likely after the 2026–2027 season begins.

### Date range

Use the last 2 full seasons (matching the Girls analysis): **2024–2025 and 2025–2026 seasons**.

---

## Section 3: Analysis Method

Follow the same held-out validation methodology used for Girls in `calibration-held-out-validation-2026-04-23.md`. Four steps:

**Step 1: Baseline Brier score (Option A, cal=100)**
Using the current Boys GA ASPIRE calibration (100), compute Brier scores on a held-out sample of Boys cross-tier games. "Cross-tier" = Boys GA ASPIRE teams playing against Boys GA or Boys MLS NEXT teams.

Brier score formula: `B = (1/N) × Σ(predicted_win_prob − actual_outcome)²`

Lower Brier score = better-calibrated predictions.

**Step 2: Counterfactual Brier score (Option B, cal=70)**
Recompute predicted win probabilities using cal=70 for Boys GA ASPIRE events. Recalculate ratings for Boys GA ASPIRE teams under the new calibration. Compute Brier score on the same held-out sample.

**Step 3: Brier score comparison**
| Calibration | Boys GA ASPIRE Value | Brier Score | Notes |
|-------------|---------------------|-------------|-------|
| Option A (current) | 100 | [result] | Girls analysis baseline: ~0.236 before fix |
| Option B (proposed) | 70 | [result] | Expected to improve if overcalibrated |

Delta = Option A Brier − Option B Brier. Positive delta = Option B is better.

**Step 4: Interpretation**
See Section 5 for the significance threshold.

---

## Section 4: Timeline and Execution Window

**Target completion:** Before June 1 Boys age-group transition. If Boys GA ASPIRE teams' ratings are miscalibrated, the error should be corrected before they roll into new age groups with inflated or deflated starting ratings.

**Do not start before:** May 17, 2026. DSS is May 14–15. ELO and Presten's attention is on DSS prep through May 16.

**Execution estimate:** 3–4 hours of ELO analysis time. The Girls analysis framework exists — this is adaptation, not construction.

**Presten DB time required:** Approximately 30 minutes to run the data-count queries (Section 2), retrieve the cross-tier game set, and if calibration change is recommended, run the production update and recompute.

**Prerequisite:** April 28 GA ASPIRE fix must have executed, so Boys GA ASPIRE events are correctly tagged as `ga_aspire` in the events table. Do not run this analysis against `event_tier = 'ga'` events.

**Preferred execution window:** May 17–May 30. June 1 transition is the deadline.

---

## Section 5: Decision Criteria

**Threshold for recommending a change from Option A to Option B:**

Brier score improvement delta > **0.008** (absolute, not relative).

Rationale: The Girls GA ASPIRE analysis showed a delta of ~0.015 between the overcalibrated (140) and corrected (100) configurations. Boys GA ASPIRE overcalibration risk is smaller — Boys GA (100) is already below Girls GA (140), so if Boys GA ASPIRE is too high, the error magnitude is smaller. A threshold of 0.008 captures meaningful improvement while filtering out noise from the smaller Boys sample size.

| Delta (Option A − Option B) | Recommendation |
|----------------------------|----------------|
| > 0.008 | Revise Boys GA ASPIRE to 70. Apply in same session format as Girls fix. |
| 0.003–0.008 | Borderline. Review individual age groups. If U13B and U14B both show improvement > 0.005, apply the change. |
| < 0.003 | Retain Option A (cal=100). No further analysis needed. Close the Boys GA ASPIRE question. |
| Data-limited (game count < 300) | Retain Option A. Re-scope after June 1 transition adds 2026–2027 Boys GA ASPIRE data. |

**If Option B is recommended:** the application follows the same pattern as the Girls GA ASPIRE fix: UPDATE the calibration constant to 70 for `ga_aspire` Boys events, run ranking recompute, run post-fix verification queries from the Boys Calibration Option A Interpretation Guide.

---

## Related

- [[Boys GA ASPIRE Calibration Decision — April 2026]] — source of Option B deferral; this spec fulfills the deferred action
- [[League Hierarchy]] — Boys GA ASPIRE calibration value (100, assumed) and Boys GA (100, validated)
- [[Boys Calibration Gap Analysis — April 2026]] — broader Boys calibration validation context; Option B Boys Brier analysis referenced there
- `02 - Tiger Tournaments/Projects/Reports/calibration-held-out-validation-2026-04-23.md` — Girls methodology to replicate
- [[June 1 Risk Assessment — Age Group Transition + ECNL Migration]] — June 1 deadline context
