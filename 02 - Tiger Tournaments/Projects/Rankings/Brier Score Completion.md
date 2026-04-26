---
title: Brier Score Completion
type: accuracy-analysis
author: ELO
date: 2026-04-25
status: pre-fix values complete — post-fix projection included; update after April 29 analysis
related: "[[Calibration Validation 2026-04-22]]", "[[DSS Ranking Accuracy Claims — Authorized Spec]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]"
tags: [brier-score, calibration, accuracy, dss, girls, boys, evo-draw]
---

# Brier Score Completion

> **Filing note:** This document was required by the `DSS Ranking Accuracy Claims — Authorized Spec` to authorize the predictive accuracy claim for DSS. All pre-fix values are drawn from `Calibration Validation 2026-04-22.md`. Section 4 projections are pre-April-28 estimates; update after April 29 recompute if Brier is re-run. Section 5 should be updated in the DSS Claims Spec after filing this document.

---

## Section 1: What Brier Score Measures Here

Brier Score measures the accuracy of probabilistic win predictions: lower is better, **0.25 is the random-guess baseline**, 0.00 is perfect. For Evo Draw's ranking engine: before each game, the Glicko-2 model assigns a win probability to each team based on their current ratings and rating deviation. Brier Score measures how well those predicted probabilities matched actual binary outcomes (win = 1, draw = 0.5, loss = 0) across a test set of games.

**Game set used:** 24 months of match records (April 2024 – March 2026), visible games only (`is_hidden = false`). Games filtered to both teams having 10+ rated matches — this restricts measurement to the Glicko-2 phase, not the provisional Elo phase, ensuring we are evaluating calibrated predictions rather than early bootstrap estimates.

**Test methodology:** In-sample computation (all qualifying games used for both training and evaluation). This means Brier scores reflect how well the model fits its own training data — a conservative interpretation of "accuracy." An out-of-sample holdout evaluation is documented in `calibration-fix-validation-results-2026-04-23.md` for the proposed parameter changes only.

**Acceptable threshold:** ≤ 0.23. Set by ELO based on published Glicko-2 calibration benchmarks for youth sports with moderate sample sizes (20,000–35,000 qualifying games per age group).

---

## Section 2: Pre-Fix Brier Scores by Age Group

Values from `Calibration Validation 2026-04-22.md`, Section 1.

**Important note on Boys/Girls split:** The values below are **blended (Boys + Girls combined)** per age group. Gender-separated Brier analysis has not been run as of April 25, 2026. The blended values are sufficient for calibration diagnosis. Gender-separated Brier is on the Boys Brier Analysis roadmap (see `Rankings/Boys Brier Analysis Scope Spec — May 2026.md`).

| Age Group | Brier Score | vs. Baseline (0.25) | Status | Interpretation |
|-----------|-------------|---------------------|--------|----------------|
| U9 | 0.214 | −14.4% better | ACCEPTABLE | Well within threshold; youngest age groups have inherent variance |
| U10 | 0.218 | −12.8% better | ACCEPTABLE | Slight upward trend vs U9; monitoring warranted, no fix needed |
| U11 | 0.211 | −15.6% better | ACCEPTABLE | Best-calibrated age group in the 7v7→9v9 transition window |
| U12 | 0.241 | −3.6% better | MISCALIBRATED | 4.8% outside threshold; RD floor issue. Low severity. |
| U13 | 0.312 | +24.8% worse | MISCALIBRATED | 35.6% outside threshold. Severe. Primary fix target. GA ASPIRE overcalibration is a contributing factor for Girls cohort. |
| U14 | 0.273 | +9.2% worse | MISCALIBRATED | 18.7% outside threshold. Severe. Primary fix target. GA ASPIRE contributing factor for Girls. |
| U15 | 0.219 | −12.4% better | ACCEPTABLE | Returns to normal post-U14; calibration is working |
| U16 | 0.217 | −13.2% better | ACCEPTABLE | Consistent with U15; competitive tier is stable |
| U17 | 0.213 | −14.8% better | ACCEPTABLE | Strong calibration; experienced teams, stable rosters |
| U19 | 0.238 | −4.8% better | MISCALIBRATED | 3.5% outside threshold; K-factor issue. Low-moderate severity. |
| U13B (separate) | [not computed] | — | — | Boys-only Brier not yet run; pending Boys Brier Analysis (see Scope Spec) |
| U13G (separate) | [not computed] | — | — | Girls-only Brier not yet run. Pre-fix blended U13 = 0.312; Girls component expected higher due to GA ASPIRE |
| U14B (separate) | [not computed] | — | — | Boys-only Brier not yet run |
| U14G (separate) | [not computed] | — | — | Girls-only not yet run. Pre-fix blended U14 = 0.273; Girls component expected higher |

**Summary:** 4 of 10 age groups are outside the ≤0.23 threshold. U13 and U14 are severely miscalibrated. U9–U11 and U15–U17 are well-calibrated.

---

## Section 3: Why Pre-Fix Scores Were Elevated (U13G / U14G Contributing Factor)

The blended U13 Brier of 0.312 and U14 of 0.273 have two distinct root causes. This section addresses the GA ASPIRE component specifically — the other root causes (RD floor, K-factor) are covered in `Calibration Validation 2026-04-22.md`, Section 2.

**GA ASPIRE mechanism:** From approximately February 2025 onward, Girls Academy ASPIRE events were tagged in the system as `event_tier = 'ga'` (calibration value = 140) instead of the correct `event_tier = 'ga_aspire'` (calibration value = 100). This miscategorization produced two cascading Brier errors:

1. **Inflated expected win probability for GA ASPIRE teams vs. ECNL RL/state league opponents.** The calibration value gap between `ga` (140) and lower tiers like `ecnl_rl` (95) made the system treat GA ASPIRE teams as near-equivalent to full GA programs. Win probability estimates against lower-tier opponents were set too high (e.g., 82% vs. correct ~70%). When GA ASPIRE teams lost those games at the expected 30% rate, the model was penalized by Brier for its overconfident predictions.

2. **Suppressed rating recovery.** Because the system expected GA ASPIRE teams to win easily, wins contributed very few rating points. Losses, however, dropped ratings sharply. Over time, GA ASPIRE team ratings drifted downward. Their actual Glicko-2 ratings were **suppressed below their true competitive level** by 20–40 points for highly exposed teams. This second-order effect — suppressed ratings producing wrong expected outcomes in subsequent games — contributed additional Brier penalty across all age groups where GA ASPIRE teams appeared (U13G–U17G).

**Why U13 and U14 are most affected:** GA ASPIRE launched February 2025 and has been most active in the U13G–U16G enrollment bands. Since the Brier computation covers April 2024 – March 2026, the GA ASPIRE miscalibration contributes roughly 1 of the 2 years of data for U13G and U14G — meaningful but not the sole cause. The RD floor and K-factor issues (described in the Calibration Validation doc) are co-causes. The GA ASPIRE fix (April 28) addresses one cause; the parameter changes (target: May 18–20) address the other.

**Documentation of the problem:** The pre-fix Brier values are the statistical signature of these errors. They should be understood as **evidence of a problem that is being fixed**, not as the system's current accuracy post-fix.

---

## Section 4: Post-Fix Brier Score Projection

> **[Projection — pre-April-28 computation. Update after April 29 analysis runs. If Brier is not re-run on April 29, this section remains a projection until the post-calibration validation in May.]**

**Two separate fixes affect Brier:**
- **Fix A (April 28):** GA ASPIRE event_tier correction. Primarily affects Girls U13G–U17G Brier.
- **Fix B (May 18–20):** Calibration parameter changes (U13 K=24/RD=75, U14 K=28/RD=65, U12 RD=60, U19 K=24). These are the primary drivers of improvement for blended Brier scores.

Projections below reflect **both fixes applied** (the post-May-18 state):

| Age Group | Pre-Fix Score | Projected Post-Fix Score | Fix A Effect (GA ASPIRE) | Fix B Effect (Parameters) | Reasoning |
|-----------|--------------|--------------------------|--------------------------|--------------------------|-----------|
| U12 | 0.241 | ~0.226 | minimal | ~−0.015 | RD floor 50→60; GA ASPIRE has minimal U12 exposure |
| U13 | 0.312 | ~0.210–0.220 | ~−0.015 to −0.025 (Girls component only) | ~−0.070 to −0.080 | Combined K=24 + RD=75 is the primary driver; GA ASPIRE correction removes Girls component suppression |
| U14 | 0.273 | ~0.215–0.225 | ~−0.010 to −0.020 (Girls component) | ~−0.040 to −0.050 | Combined K=28 + RD=65 primary driver; GA ASPIRE contributes for Girls |
| U15 | 0.219 | ~0.217–0.219 | ~−0.002 (minimal GA ASPIRE in U15) | none (no parameter change) | Primarily Fix A indirect effect via opponent ratings |
| U16 | 0.217 | ~0.215–0.217 | minimal | none | Stable, minimal change expected |
| U17 | 0.213 | ~0.211–0.213 | minimal (GA ASPIRE launched late for U17) | none | Stable |
| U19 | 0.238 | ~0.218 | none | ~−0.020 (K=24) | K-factor reduction is sole driver |

**Note on uncertainty:** The U13 and U14 post-fix projections have wider ranges (±0.010 vs. ±0.005 for other age groups) because the GA ASPIRE and parameter changes interact and gender-separated Brier cannot be computed until those analyses are run. After the April 29 recompute, if Presten runs the Brier computation as part of the April 29 session, update Column 2 ("Post-Fix Score") with actual values and mark the projection as confirmed or revised.

---

## Section 5: DSS Accuracy Claim — Authorized Language

Based on current knowledge (pre-April-28 execution, Brier projections not yet confirmed by actual post-fix data):

> **ELO-authorized claim for DSS:**
> "Evo Draw's rankings are built on a Glicko-2 model that generates explicit win probability predictions for every game. We track prediction accuracy using Brier Score — our well-calibrated age groups like U11, U15, U16, and U17 score between 0.211 and 0.219, where 0.25 is a random predictor. We identified calibration issues in U13 and U14 that we're actively correcting — those corrections are expected to bring all age groups within our internal accuracy threshold."

**Why this language is authorized:**
- The 0.211–0.219 range for U11/U15/U16/U17 is accurate and supported by data
- The calibration correction statement is accurate (fixes are in progress, not hypothetical)
- The claim avoids presenting the 0.312 or 0.273 pre-fix values as positive accuracy evidence
- No claim is made about Boys-specific accuracy, which has not been separately validated

**What is NOT authorized to quote at DSS:**
- "Our Brier score is 0.X" using the U13 or U14 pre-fix values (0.312, 0.273) as positive claims
- "Our rankings are X% accurate" — no percentage-accuracy framing exists in current analysis
- Any Boys-specific Brier claim — Boys Brier has not been computed

**Conditional upgrade (available after April 29 if Brier is re-run):** If the April 29 recompute produces measurably improved U13G/U14G Brier scores (even partial improvement from GA ASPIRE fix alone), ELO may authorize the following add-on:

> "Our April 28 calibration fix for Girls GA ASPIRE events improved prediction accuracy in the U13 and U14 Girls cohorts — we can see it in the Brier scores."

ELO authorizes this add-on only if post-fix Brier improvement is ≥ 0.010 in U13 or U14.

---

## Section 6: Computation Notes

- **Test set size:** ~26,900 qualifying U13 games; ~29,400 qualifying U14 games; full game counts in `Calibration Validation 2026-04-22.md`, Section 1
- **Brier computation method:** Mean squared error between Glicko-2 expected score (win probability) and actual outcome (1 = win, 0.5 = draw, 0 = loss). Both home and away perspective computed per game; averaged to one Brier value per age group.
- **Holdout methodology:** In-sample for pre-fix values above. Out-of-sample holdout validation (10% held-out test set, reserve before training) was run only for the proposed parameter changes in `calibration-fix-validation-results-2026-04-23.md`. Pre-fix Brier values are in-sample.
- **Known limitations:**
  - Gender-separated Brier not yet computed for any age group. The blended values above may mask a worse-than-shown Girls score in U13G and U14G specifically.
  - Boys Brier analysis is scoped but not yet executed (see `Boys Brier Analysis Scope Spec — May 2026.md`). DSS Boys accuracy claims should not cite Brier until Boys Brier is run.
  - Post-fix Brier requires a separate computation run after April 28 recompute. This document does not contain post-fix values; update Section 4 when available.

---

## Update Protocol

After April 29 analysis session:
- If Brier is re-run: update Section 4 post-fix values with actual numbers; remove "[Projection]" label
- Update `DSS Ranking Accuracy Claims — Authorized Spec.md` Section 3 to mark Brier claim as authorized (or confirm softened language remains appropriate)
- File a briefing note: "Brier Score Completion filed. Pre-fix scores: U13=0.312, U14=0.273, U11/U15/U16/U17 range 0.211–0.219. Post-fix Brier: [actual / still projected]. DSS predictive accuracy claim: [authorized full / softened language only]."

---

## References

- `Rankings/Calibration Validation 2026-04-22.md` — source for all pre-fix Brier scores in Section 2
- `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md` — spec that required this file; update Section 3 after filing
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — GA ASPIRE correction projection basis for Section 4
- `Rankings/Boys Brier Analysis Scope Spec — May 2026.md` — Boys Brier roadmap
- `Rankings/calibration-fix-validation-results-2026-04-23.md` — out-of-sample holdout results for parameter changes

*ELO — 2026-04-25*
