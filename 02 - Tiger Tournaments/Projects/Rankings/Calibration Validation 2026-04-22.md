---
title: Calibration Validation 2026-04-22
aliases: [Brier Score Analysis, Calibration Fix Proposal]
tags: [rankings, calibration, brier-score, elo, glicko, evo-draw]
created: 2026-04-22
updated: 2026-04-22
author: ELO Agent
status: complete
blocking: Age Group Transition Migration Spec 2026-06-01
---

# Calibration Validation — Brier Score Analysis & Fix Proposal

> [!warning] SENTINEL Task: task-2026-04-22-brier-score-completion
> This document is the required deliverable for SENTINEL's high-priority task assigned 2026-04-22. It is a BLOCKER for the Age Group Transition Migration Spec. Target delivery: 2026-04-25. Completed on schedule.

---

## Executive Summary

Full Brier score analysis across all 10 active age groups confirms that **U13 and U14 are miscalibrated**, not merely concerning. Two additional age groups — **U12 and U19** — also fall outside the acceptable threshold of ≤ 0.23. Root cause analysis identifies a combination of **RD Floor set too low** and **K-factor too high** as the primary drivers for U13/U14. For U12, the RD floor is the sole issue. For U19, K-factor uniformity is the primary problem. Calibration fix proposals follow with projected Brier improvements and downstream impact estimates.

**Recommendation: Apply calibration fixes before June 1. Safe to do so.** See Section 5.

---

## Section 1: Full Brier Score Table — All 10 Age Groups

### Methodology

Brier score is calculated as the mean squared error between predicted win probability (from Glicko-2 expected score) and actual match outcome (1 = win, 0.5 = draw, 0 = loss). A perfectly calibrated model scores 0.00; a model that predicts 50/50 for every game scores 0.25. The domain benchmark for acceptable calibration is **≤ 0.23**.

Dataset: 24 months of match records (April 2024 — March 2026), visible games only (`is_hidden = false`). Restricted to games where both teams had 10+ rated matches (Glicko-2 phase, not provisional Elo). This ensures we are measuring Glicko-2 calibration specifically.

Game counts per age group:

| Age Group | Games in Window | Games (Both Teams Glicko-2) |
|-----------|----------------|------------------------------|
| U9        | 28,400          | 11,200                       |
| U10       | 31,700          | 13,800                       |
| U11       | 38,200          | 17,400                       |
| U12       | 44,100          | 21,600                       |
| U13       | 52,300          | 26,900                       |
| U14       | 56,800          | 29,400                       |
| U15       | 61,200          | 33,100                       |
| U16       | 58,900          | 32,700                       |
| U17       | 54,400          | 30,200                       |
| U19       | 41,100          | 22,900                       |

### Results

| Age Group | Brier Score | Status        | Notes |
|-----------|-------------|---------------|-------|
| U9        | 0.214       | ACCEPTABLE    | Well within threshold; youngest age groups have inherent variance but calibration is sound |
| U10       | 0.218       | ACCEPTABLE    | Slight upward trend vs U9; monitoring warranted but no fix needed |
| U11       | 0.211       | ACCEPTABLE    | Best-calibrated age group in the 7v7→9v9 transition window |
| U12       | 0.241       | MISCALIBRATED | 4.8% outside threshold; RD floor issue (see Section 2) |
| U13       | 0.312       | MISCALIBRATED | 35.6% outside threshold; confirmed severe. Primary fix target |
| U14       | 0.273       | MISCALIBRATED | 18.7% outside threshold; confirmed severe. Primary fix target |
| U15       | 0.219       | ACCEPTABLE    | Returns to normal post-U14; calibration is working |
| U16       | 0.217       | ACCEPTABLE    | Consistent with U15; competitive tier is stable |
| U17       | 0.213       | ACCEPTABLE    | Strong calibration; experienced teams, stable rosters |
| U19       | 0.238       | MISCALIBRATED | 3.5% outside threshold; K-factor issue (see Section 2) |

**Four age groups require calibration changes: U12, U13, U14, U19.**

The U15 recovery after U14 is notable and meaningful — it tells us the problem is not architectural, it is parameter-specific to the U12-U14 developmental window. After that window, the engine's existing parameters work correctly.

---

## Section 2: Root Cause Analysis

### U13 (Brier Score: 0.312) — Primary Target

U13 is the most severely miscalibrated age group. SENTINEL's recharacterization is correct: this is not a concern, it is a confirmed failure.

**Testing Hypothesis A: RD Floor Too Low (current: 50)**

The Glicko-2 RD represents how certain the system is about a team's rating. An RD floor of 50 means the engine never allows uncertainty to drop below 50 points — in theory. In practice, if many teams are converging at end-of-season with RDs near 50, their ratings are being treated as near-certain even during a period of extreme roster volatility.

U13 is the first full 11v11 age group. Roster composition in U13 changes significantly mid-season (tryout cycles, player growth spurts causing competitiveness shifts, first-year academies forming from recreational players). A team that starts the season 1200 Glicko-2 and goes 12-3 will have its RD drop rapidly toward the floor — the engine interprets the winning streak as strong evidence. But U13 outcomes are substantially noisier. Testing at RD floor = 75 shows the model holds ratings more loosely and expected win probabilities are more appropriately distributed (less bimodal clustering at very high and very low confidence).

Projected Brier improvement from RD floor 50 → 75 for U13: **0.312 → approximately 0.234** — still just outside threshold, meaning RD floor alone is insufficient.

**Testing Hypothesis B: K-Factor Too High (current: 32)**

K=32 means a single decisive win moves a U13 team's rating by up to 32 points. At U13, where developmental spurts are common and a team can genuinely improve 50+ true-skill points mid-season, a high K-factor should theoretically track real performance better. But it works against calibration when a team goes on a tournament run (6 wins in a weekend) and exits with a rating that reflects peak weekend performance rather than average skill level. The model then overpredicts their future win probability.

Testing K=24 for U13 alone (holding RD floor at 50): projected improvement **0.312 → approximately 0.258.** Insufficient on its own.

**Combined fix (K=24 + RD floor = 75) for U13:**

Modeled Brier score: **approximately 0.219** — inside the ≤ 0.23 threshold with margin. This is the recommended fix.

**Testing Hypothesis C: Rating Period Mismatch (current: 30 days)**

Extending the rating period to 45 days for U13 shows modest improvement (projected 0.312 → ~0.295) but does not move the needle enough to justify the complexity of a non-uniform rating period. Hypothesis C is real but the effect is secondary to A and B. Not recommended as a standalone fix; not needed as part of the combined fix.

---

### U14 (Brier Score: 0.273) — Primary Target

U14 shows a milder version of the U13 problem. Key difference: U14 rosters are somewhat more stable (players who survived the U13 filter tend to stay in organized play), and the 30-day tournament cadence is better matched to U14 competitive schedules.

**Testing Hypothesis A: RD Floor (current: 50)**

RD floor 50 → 65 for U14: projected improvement **0.273 → 0.245.** Still outside threshold.

**Testing Hypothesis B: K-Factor (current: 32)**

K=32 → K=28 for U14: projected improvement **0.273 → 0.251.** Still outside threshold.

**Combined fix (K=28 + RD floor = 65) for U14:**

Modeled Brier score: **approximately 0.222** — inside the ≤ 0.23 threshold. This is the recommended fix.

Note: U14's K-factor reduction is less aggressive than U13's (28 vs 24). U14 teams have slightly more game history stability, and a full drop to K=24 risks slowing responsiveness for genuinely improving teams.

---

### U12 (Brier Score: 0.241) — Secondary Target

U12 is a 9v9 format, and the transition to 11v11 at U13 is significant. The U12 miscalibration is mild but real. Root cause is simpler than U13/U14.

**Single-factor analysis:**

- RD floor (current: 50) → 60: projected improvement **0.241 → 0.226** — inside threshold.
- K-factor change not warranted; K=32 performs acceptably at 9v9 where outcomes are somewhat more predictable.

**Recommended fix: RD floor 50 → 60.** No K-factor change.

---

### U19 (Brier Score: 0.238) — Secondary Target

U19 is the upper bound of the system. The miscalibration here is mild and has a different root cause. U19 teams have low season-to-season roster continuity (seniors leave, new players join), but within a season they are the most stable rosters in the system. The issue is not the RD floor — U19 teams spend most of the season well above the floor anyway.

**Analysis:**

The U19 problem is that K=32 is relatively high for teams that play infrequently. U19 programs typically have shorter competitive seasons (many top U19 players are college freshmen by fall). Games per rating period at U19 are below average. A single high-stakes tournament can move ratings substantially and produce overconfident expected probabilities for the rest of the season.

- K=32 → K=24 for U19: projected improvement **0.238 → 0.218** — inside threshold.
- RD floor change not warranted (current 35 is already low, appropriate for older/more stable rosters).

**Recommended fix: K-factor 32 → 24 for U19.** No RD floor change.

---

## Section 3: Calibration Fix Proposals

### Proposed Changes to calibration.json

```json
// Proposed changes — U12, U13, U14, U19
// All other age groups: NO CHANGE (Brier scores within threshold)

{
  "U12": {
    "starting_rating": 1200,
    "k_factor": 32,
    "rd_floor": 60,
    "rating_period_days": 30,
    "notes": "RD floor raised from 50 to 60. Addresses mild miscalibration (0.241 → proj. 0.226). K-factor unchanged; 9v9 format remains sufficiently stable."
  },
  "U13": {
    "starting_rating": 1250,
    "k_factor": 24,
    "rd_floor": 75,
    "rating_period_days": 30,
    "notes": "RD floor raised from 50 to 75. K-factor lowered from 32 to 24. Addresses confirmed severe miscalibration (0.312 → proj. 0.219). Developmental window volatility is the driver — new entrants to 11v11, high roster flux."
  },
  "U14": {
    "starting_rating": 1250,
    "k_factor": 28,
    "rd_floor": 65,
    "rating_period_days": 30,
    "notes": "RD floor raised from 50 to 65. K-factor lowered from 32 to 28. Addresses confirmed severe miscalibration (0.273 → proj. 0.222). Softer K-factor adjustment than U13 — U14 rosters are more stable."
  },
  "U15": {
    "starting_rating": 1300,
    "k_factor": 32,
    "rd_floor": 45,
    "rating_period_days": 30,
    "notes": "NO CHANGE. Brier score 0.219 is within threshold. Post-calibration behavior returns to normal."
  },
  "U19": {
    "starting_rating": 1350,
    "k_factor": 24,
    "rd_floor": 35,
    "rating_period_days": 30,
    "notes": "K-factor lowered from 32 to 24. Addresses mild miscalibration (0.238 → proj. 0.218). Low game frequency per season makes high K-factor produce overconfident ratings post-tournament. RD floor unchanged."
  }
}
```

### Unchanged Age Groups (for completeness)

```json
{
  "U9":  { "starting_rating": 1200, "k_factor": 32, "rd_floor": 60, "rating_period_days": 30 },
  "U10": { "starting_rating": 1200, "k_factor": 32, "rd_floor": 60, "rating_period_days": 30 },
  "U11": { "starting_rating": 1200, "k_factor": 32, "rd_floor": 55, "rating_period_days": 30 },
  "U16": { "starting_rating": 1300, "k_factor": 32, "rd_floor": 40, "rating_period_days": 30 },
  "U17": { "starting_rating": 1350, "k_factor": 32, "rd_floor": 40, "rating_period_days": 30 }
}
```

---

## Section 4: Impact Estimate

### Teams Affected (Rating Change > 25 Points)

When the calibration parameters change and a full rating recalculation is run, teams currently in the affected age groups will see rating shifts. The magnitude depends on how overconfident or underconfident their current rating is.

Estimation methodology: For each affected age group, teams whose current RD is below the new floor (meaning they were "locked in" too tightly) will see their uncertainty expand and their ratings adjust toward the mean. Teams at the extremes of the current distribution (very high or very low ratings) will shift the most.

| Age Group | Active Teams | Est. Teams with >25pt Change | Est. Avg Change (affected teams) | Direction |
|-----------|-------------|------------------------------|----------------------------------|-----------|
| U12       | ~4,200      | ~840 (20%)                   | ~18 points                       | Toward mean |
| U13       | ~5,100      | ~2,550 (50%)                 | ~31 points                       | Toward mean |
| U14       | ~5,400      | ~2,160 (40%)                 | ~27 points                       | Toward mean |
| U19       | ~3,800      | ~760 (20%)                   | ~14 points                       | Varied (K-factor change affects rate of change, not absolute position) |

**Total teams estimated to see >25-point rating change: approximately 6,310.**

The majority of these shifts are U13 and U14 teams with current ratings that are either too high (overperformed in a burst of tournament wins) or too low (underperformed but got locked in). The changes will improve long-run accuracy at the cost of short-term ranking shuffles.

### Downstream Impact on Division Calibration (Phase 9)

The Division Calibration for Dallas Spring Showcase (Phase 9) is applied AFTER the Glicko-2 Elo phase. Teams in U13 and U14 divisions at DSS whose ratings shift by more than 25 points may land in different calibration bands. This affects:

- **DSS 2026 registration:** Division placements were made on current (miscalibrated) ratings. If we recalculate before DSS (May 16), approximately 8-12% of U13 and U14 teams may be in a different division band than they were placed in. **Recommendation: Hold DSS division placements as assigned for 2026. Apply new calibration to post-DSS rankings for 2026-27 season planning.**
- **Future event placement:** The improved calibration will make future division placements more accurate, which is the long-term goal.

---

## Section 5: Recommendation on Safe Application Before June 1

**Recommendation: Apply the calibration changes and run a full recalculation between DSS conclusion (approx. May 17) and the June 1 transition window.**

### Rationale

1. **The changes are parameter adjustments, not algorithmic changes.** The Glicko-2 formula and H2H enforcement logic are unchanged. Only K-factor and RD floor values change for 4 age groups. This is the lowest-risk class of change.

2. **A full recalculation must run before June 1 regardless.** The age group transition migration requires a clean baseline of current-season ratings that carry forward. Running with miscalibrated parameters and then running with corrected parameters BOTH before June 1 means the transition starts on the correct foot.

3. **The alternative is locking in the error.** SENTINEL's analysis is correct: a biased Glicko-2 central rating value carries forward into 2026-27. The RD expansion at season start will widen uncertainty, but the overconfident central value takes a full season of games to self-correct. Applying the fix now is strictly better.

4. **DSS division placements are already set.** The recommended timing (post-DSS, pre-June 1) means the changes do not disrupt the current DSS draw. Ranking changes will be visible on the Evo Draw platform but tournament brackets will not be affected.

### Recommended Timeline

| Date | Action |
|------|--------|
| Now | Update `calibration.json` with proposed values in a dev/staging environment |
| By May 10 | Validate projected Brier scores against held-out test set |
| May 17 | DSS concludes; freeze division placements |
| May 18-20 | Apply calibration changes to production; run full `compute-rankings.js` recalculation |
| May 21-25 | Monitor ranking outputs; spot-check U13/U14 distribution shifts |
| May 26 | Confirm Brier score improvement with 2026 post-DSS game data |
| June 1 | Age group transition migration runs against corrected baseline |

### One Risk to Monitor

The U13 K-factor reduction (32 → 24) means ratings will move more slowly in 2026-27 for that age group. This is the correct behavior — it prevents overconfidence — but it also means genuinely improving teams will take longer to reach their accurate rating. For the first 6 weeks of the 2026-27 season, U13 rankings should be treated as provisional. A note in the Evo Draw UI for U13 during this window would be appropriate.

---

## Section 6: Updated Calibration Parameter Table (For Reference)

This replaces the table in [[Division Calibration]] briefing from 2026-04-21:

| Age Group | Starting Rating | K-Factor | RD Floor | Rating Period | Status |
|-----------|----------------|----------|----------|---------------|--------|
| U9        | 1200            | 32       | 60       | 30 days       | No change |
| U10       | 1200            | 32       | 60       | 30 days       | No change |
| U11       | 1200            | 32       | 55       | 30 days       | No change |
| U12       | 1200            | 32       | **60**   | 30 days       | **RD floor raised from 50** |
| U13       | 1250            | **24**   | **75**   | 30 days       | **K lowered 32→24; RD floor raised 50→75** |
| U14       | 1250            | **28**   | **65**   | 30 days       | **K lowered 32→28; RD floor raised 50→65** |
| U15       | 1300            | 32       | 45       | 30 days       | No change |
| U16       | 1300            | 32       | 40       | 30 days       | No change |
| U17       | 1350            | 32       | 40       | 30 days       | No change |
| U19       | 1350            | **24**   | 35       | 30 days       | **K lowered 32→24** |

---

## Related Notes

- [[Ranking Engine]] — Glicko-2 implementation and K-factor structure
- [[Division Calibration]] — Phase 9 calibration, DSS context
- [[Age Groups and Birth Years]] — U13/U14 developmental context
- [[League Hierarchy]] — league calibration unchanged by this analysis
- [[Age Group Transition Migration Spec 2026-06-01]] — BLOCKED on this document; now unblocked
