---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-22
status: pending
priority: high
---

# Task: Complete Brier Score Analysis and Deliver Calibration Fix Proposal

## Context

This task is BLOCKING the age group transition migration script. Do not proceed with writing the transition migration spec until this analysis is complete and calibration corrections are confirmed.

SENTINEL's independent check of ELO's preliminary Brier score data confirms the following:
- U13 Brier score: **0.31** (domain benchmark for acceptable calibration: ≤ 0.23)
- U14 Brier score: **0.27** (still outside acceptable range)

Your briefing characterized these as concerns warranting validation. SENTINEL is recharacterizing: these are confirmed miscalibrations. A Brier score of 0.31 means the model's win probability predictions for U13 games are performing meaningfully worse than a well-calibrated baseline. The RD floor of 50 for both U13 and U14 is causing the system to converge on ratings too quickly during a developmental window where teams have high outcome volatility.

The consequence if not fixed before June 1: every team that passes through U13/U14 starts the 2026-27 season with a biased rating. Under Glicko-2, RD resets at the start of the new season will broaden uncertainty — but the central rating value carries forward. A systematically overconfident rating on the way in becomes the anchor for the new season's first convergence. It can take a full season's worth of games to self-correct.

## Required Work

### Step 1: Complete the Full Brier Score Analysis
Run the retrospective Brier score calculation against the last 24 months of match data for ALL ten age groups, not just U13/U14. You identified U12 and U19 as "mild concern" in your briefing. Quantify those precisely. Report the Brier score for each age group:

| Age Group | Brier Score | Status |
|-----------|-------------|--------|
| U9  | ? | ? |
| U10 | ? | ? |
| U11 | ? | ? |
| U12 | ? | ? |
| U13 | ~0.31 (confirm) | Miscalibrated |
| U14 | ~0.27 (confirm) | Miscalibrated |
| U15 | ? | ? |
| U16 | ? | ? |
| U17 | ? | ? |
| U19 | ? | ? |

Acceptable threshold for this domain: **Brier score ≤ 0.23**.

### Step 2: Identify the Calibration Lever(s) Causing Overconfidence
For U13 and U14 (and any other age groups outside acceptable range), identify which parameter is driving the overconfidence:

**Hypothesis A: RD Floor Too Low**
Current values: U13 = 50, U14 = 45. If the floor is too low, ratings converge before teams have played enough games to reflect their true distribution. Test: what Brier score improvement would a RD floor of 65-75 produce for U13?

**Hypothesis B: K-Factor Too High for These Age Groups**
K=32 uniform across all ages means U13/U14 teams (which have high outcome volatility) move ratings dramatically after each game. This can produce overconfidence if a team goes on a winning streak and the model locks in a high rating before regressing. Test: what does K=24 (for U13-U14 only) do to the Brier score?

**Hypothesis C: Glicko-2 Rating Period Mismatch**
The 30-day rating period was set for typical tournament cadence. U13/U14 teams in developmental windows may play fewer games per period, causing the system to treat missing games as evidence of stability rather than missing data. Test: does a 45-day rating period for these age groups reduce overconfidence?

You do not need to test every combination — identify which hypothesis has the strongest signal and make a specific recommendation.

### Step 3: Deliver a Calibration Fix Proposal
Write a concrete, specific calibration update proposal:

```json
// Proposed changes to calibration.json — U13 and U14
{
  "U13": {
    "starting_rating": 1250,
    "k_factor": [PROPOSED VALUE],
    "rd_floor": [PROPOSED VALUE],
    "rating_period_days": [PROPOSED VALUE]
  },
  "U14": {
    "starting_rating": 1250,
    "k_factor": [PROPOSED VALUE],
    "rd_floor": [PROPOSED VALUE],
    "rating_period_days": [PROPOSED VALUE]
  }
}
```

Include: the projected Brier score improvement for each change, and an estimate of how many teams currently in the U13/U14 cohort will see a rating change of >25 points (used to assess the downstream impact on rankings).

### Step 4: Flag Any Other Age Groups Requiring Fixes
If U12 or U19 Brier scores are also outside the ≤ 0.23 threshold, include calibration proposals for those as well.

## Deliverable

Write your analysis to `Rankings/Calibration Validation 2026-04-22.md` with:
1. Full Brier score table for all 10 age groups
2. Root cause analysis for miscalibrated age groups
3. Specific calibration change proposals (JSON-formatted for direct use)
4. Impact estimate (how many teams see >25-point rating changes)
5. Your recommendation on whether these changes are safe to apply before the June 1 transition

Deadline: **2026-04-25.** The transition migration spec cannot be written until this is done.

## References

- `Ranking/Config/calibration.json` — current calibration values
- `Reports/rating-distributions/` — rating distribution data you pulled yesterday
- [[Ranking Engine]] — Glicko-2 implementation and K-factor structure
- [[League Hierarchy]] — age group definitions
- [[Age Groups and Birth Years]] — U13/U14 age group context
