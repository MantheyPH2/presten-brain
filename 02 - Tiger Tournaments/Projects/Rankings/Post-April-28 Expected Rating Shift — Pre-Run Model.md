---
title: Post-April-28 Expected Rating Shift — Pre-Run Model
type: prediction-document
author: ELO
date: 2026-04-25
status: pre-run prediction
tags: [calibration, ga-aspire, prediction, april-28, girls, evo-draw]
related: "[[Post-April-28 Rating Shift Analysis Spec]]", "[[April 29 Post-Fix Decision Tree]]", "[[ga-coverage-audit-results-2026-04-23]]", "[[Boys Calibration Status Summary — April 2026]]"
---

# Post-April-28 Expected Rating Shift — Pre-Run Model

> **[Pre-Run Prediction — Written before April 28 execution]**
>
> This document is ELO's signed prediction of what the April 28 GA ASPIRE fix should produce. Written April 25, 2026. Compare against actual results April 29 using `Rankings/Post-April-28 Rating Shift Analysis Spec.md`.

---

## Section 1: Mechanism Summary

The April 28 fix changes `event_tier` from `ga` to `ga_aspire` for all Girls Academy ASPIRE events in `event_tiers`. This changes the calibration value applied to those games when `compute-rankings.js` runs Phase 8.

**Before the fix:** GA ASPIRE events are tagged as `event_tier = 'ga'` for Girls teams. Calibration value for `ga` (Girls) = **140** (from League Hierarchy). _Note: the Boys GA value is 100, but Girls GA is 140 — this is the correct Girls GA value and the source of the overcalibration issue._

**After the fix:** GA ASPIRE events get `event_tier = 'ga_aspire'`. Calibration value for `ga_aspire` (Girls) = **100** (League Hierarchy, validated post-fix).

**Net effect:** Games in GA ASPIRE events now use a **lower** calibration value for Girls teams (140 → 100). This means each GA ASPIRE game contributes **less** K-factor weight to rating changes per game — the calibration delta between GA ASPIRE and lower-tier leagues shrinks from 85 (140–55) to 45 (100–55) when playing against ECNL RL-level opponents.

**Why Girls GA ASPIRE ratings are expected to INCREASE:**

In a Glicko-2 system with calibration, the expected outcome of a game uses the calibration-adjusted strength difference between teams. When GA ASPIRE was tagged at cal=140, the system expected GA ASPIRE teams to dominate ECNL RL, NPL, and state league opponents by a wide margin — essentially treating them as full-GA-level programs. Because the expected win probability was so high, wins yielded very few Glicko points (the system said "expected outcome, minimal new information"). Losses, conversely, were heavily penalized.

The result: GA ASPIRE teams' stored Glicko ratings were **suppressed** — held below the level their actual game results would justify if correctly calibrated at 100. After the fix, those same win records are recomputed with a more modest expected margin, and the system awards significantly more points per win → ratings **increase**.

This fix is expected to:
- **Raise** average Girls ratings in GA ASPIRE-heavy age groups (U13G–U17G)
- **Narrow the gap** between Girls GA ASPIRE team ratings and Girls ECNL/MLS NEXT ratings (correcting the suppression that made GA ASPIRE teams appear further behind elite clubs than they actually are)
- Have **zero net effect on Boys ratings** — Boys GA = 100 and Boys GA ASPIRE Option A = 100; the event_tier reclassification changes the label but not the calibration value. Boys rating change should be 0 ± 5 pts from floating-point recompute variance only.

---

## Section 2: Affected Population Estimate

From `Rankings/ga-coverage-audit-results-2026-04-23.md` (Sections 1–2):

| Metric | Estimated Range | Source |
|--------|----------------|--------|
| ASPIRE events to reclassify | 15–50 events | GA Coverage Audit §2A |
| ASPIRE games to reclassify | 500–1,500 games | GA Coverage Audit §2A |
| Teams with ≥1 GA ASPIRE game | 350–1,050 teams | GA Coverage Audit §2A |
| Teams with ≥10 GA ASPIRE games (most affected) | ~150–500 teams | ELO estimate — roughly 40–50% of affected teams |

**Age groups with highest GA ASPIRE game share:**

GA ASPIRE launched February 2025, so all exposure is from the 2024–2025 and 2025–2026 seasons. Given that GA programs are most active in U13G–U16G (the core Girls Academy enrollment bands), the expected exposure ranking is:

1. **U14G** — peak GA enrollment age group; likely highest ASPIRE game count
2. **U15G** — close second; strong GA presence
3. **U13G** — DSS demo cohort; meaningful ASPIRE presence, slightly lower than U14–15
4. **U16G** — tapering as older teams transition
5. **U17G** — lowest GA ASPIRE exposure (ASPIRE launched only Feb 2025; limited time to accumulate games)

If exact counts from the audit are below 500 total games when executed, the effect will be smaller in magnitude. The ranges above represent the expectation prior to execution.

---

## Section 3: Predicted Rating Shift Ranges

For each Girls age group with significant GA ASPIRE exposure:

**Derivation formula:** Predicted magnitude ≈ (GA ASPIRE game share × calibration delta × avg K-factor per game). The calibration delta is 40 points (140 → 100). For teams with 10+ GA ASPIRE games, this produces a 20–40 pt rating correction. For teams with 2–9 GA ASPIRE games, the correction is proportionally smaller (5–20 pts). Teams with 0 GA ASPIRE games should be unchanged (±5 pts indirect effect through opponent ratings only).

| Age Group | Estimated % Games Tagged GA ASPIRE | Predicted Direction | Predicted Magnitude (pts) — Highly Exposed Teams | Predicted Magnitude — Avg Across Cohort |
|-----------|---------------------------------------|---------------------|--------------------------------------------------|----------------------------------------|
| U13G | 15–30% of GA-affiliated teams | **INCREASE** | +20 to +40 pts | +5 to +15 pts |
| U14G | 20–35% of GA-affiliated teams | **INCREASE** | +25 to +40 pts | +8 to +18 pts |
| U15G | 18–30% of GA-affiliated teams | **INCREASE** | +20 to +40 pts | +6 to +15 pts |
| U16G | 10–20% of GA-affiliated teams | **INCREASE** | +15 to +35 pts | +4 to +12 pts |
| U17G | 5–15% of GA-affiliated teams | **INCREASE** | +10 to +30 pts | +2 to +8 pts |

**Upper bound:** Teams with the highest GA ASPIRE game share (≥ 15 games) may see corrections up to 40 pts, which is the full distortion magnitude identified in previous analysis. This is the ceiling — few teams will be at 40 pts because it requires nearly all of a team's games to be in ASPIRE events.

**Boys prediction:** Boys ratings should shift **0 points**. Any Boys cohort showing a rating shift > ±5 pts after the April 28 recompute is a diagnostic signal of an unintended scope change in the UPDATE or computation script. State this explicitly: a Boys rating shift > 5 pts is a false positive and must be investigated before proceeding with Club Rankings or Event Strength Phase 1.

---

## Section 4: Distribution Shift Predictions

Beyond individual team shifts, the aggregate distribution should shift as follows:

**Girls GA ASPIRE average rating** (across all age groups): expected to move **UP** by approximately **+8 to +20 points** at the cohort level. The range is wide because it depends on the fraction of GA-affiliated teams in each cohort that have ASPIRE game exposure.

**Girls non-GA-ASPIRE average rating** (ECNL, MLS NEXT adjacent, state league): expected to move **minimally or not at all** — ±3 pts maximum. These teams' opponents include GA ASPIRE teams, and as GA ASPIRE ratings increase, the expected outcomes of cross-league matchups shift slightly (GA ASPIRE teams are now correctly recognized as stronger). This produces a small indirect downward pull on non-GA-ASPIRE teams' ratings via opponent strength recalibration, but the effect is < 5 pts average.

**Convergence between Girls GA ASPIRE and Girls ECNL:** expected **convergence** (gap narrows). Before the fix: Girls GA ASPIRE teams' suppressed ratings made them look ~40 pts weaker than their actual performance relative to ECNL teams would suggest. After the fix: the gap closes. Girls GA ASPIRE average should move closer to Girls ECNL average, not farther away.

This feeds the April 29 Decision Tree G2 gate: if the Girls GA ASPIRE average DIVERGES from Girls ECNL after the fix (gap widens), the fix produced the wrong outcome and G1 fails.

---

## Section 5: Anomaly Detection Thresholds

Explicit pass/fail thresholds for the April 29 Decision Tree G1 and G4 gates:

| Signal | Expected (This Model) | If Outside Range: Interpretation |
|--------|-----------------------|----------------------------------|
| Girls GA ASPIRE avg rating change (U13G) | **+10 to +40 pts** (positive) | If < +10 pts → distortion only partially resolved; G1 partial fail. If negative (any amount) → fix applied incorrectly or UPDATE failed; G1 hard fail. |
| Girls GA ASPIRE avg rating change (U14G) | **+10 to +40 pts** (positive) | Same as U13G |
| Boys avg rating change (any age group) | **0 pts ± 5 pts** | If > +5 or < −5 → recompute had unintended Boys scope; investigate before Club Rankings proceeds. If > ±15 pts → G4 fail; hold all downstream. |
| Girls ECNL avg rating change | **< 5 pts** | If > 10 pts → fix scope exceeded GA ASPIRE; unexpected propagation via opponent ratings. |
| Total teams with > 50-pt shift (any direction) | **< 80 teams** | If >> 100 teams → cal value change produced larger effect than modeled; investigate outliers before Rank Bands. |
| GA ASPIRE avg change vs ECNL avg change | **GA ASPIRE ↑, ECNL minimal** (convergence) | If ECNL shifts > 10 pts while GA ASPIRE shifts < 10 pts → calibration applied to wrong tier; stop immediately. |

**Boys assertion (diagnostic, not negotiable):** Boys avg rating change must be ≤ ±5 pts. Any Boys movement > 5 pts triggers an investigation regardless of Girls-side results. This is not a gray area.

---

## ELO Confidence Statement

Confidence in predicted direction: **High**. The mechanism (suppressed Glicko ratings from over-expected wins at cal=140) is well-established in Glicko-2 literature. The fix reduces the expected win advantage, which recovers the suppressed points.

Confidence in predicted magnitude: **Medium**. The 15–40 pt range for most-affected teams is ELO's best estimate based on GA Coverage Audit population data. Actual magnitude depends on each team's ratio of GA ASPIRE games to total games, which ELO does not have exact counts for prior to execution.

If the actual shift falls outside the predicted ranges, ELO will investigate mechanism, not revise the prediction retroactively.

---

## References

- `Rankings/Post-April-28 Rating Shift Analysis Spec.md` — analysis methodology; execute April 29 against this model
- `Rankings/April 29 Post-Fix Decision Tree.md` — G1 and G4 gate thresholds this model informs
- `Rankings/ga-coverage-audit-results-2026-04-23.md` — affected game volume data
- `Rankings/Boys Calibration Status Summary — April 2026.md` — confirms Boys should not be affected
- `02 - Tiger Tournaments/Projects/Leagues/League Hierarchy.md` — cal values (ga Girls = 140, ga_aspire Girls = 100, ga Boys = 100)

*ELO — 2026-04-25*
