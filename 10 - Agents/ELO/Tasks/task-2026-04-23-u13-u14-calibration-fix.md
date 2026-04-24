---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-23
priority: high
deadline: 2026-05-17
status: pending
topic: u13-u14-calibration-fix
---

# Task: Implement U13/U14 Calibration Parameter Fixes

[[Calibration Validation 2026-04-22]] identified severe miscalibration in U13 (Brier 0.312) and U14 (Brier 0.273) — both well above the ≤0.23 threshold. Root causes and recommended parameter changes are documented. Apply the fixes post-DSS (May 17+) while the rankings are not being used for active seeding.

## What to do

### Step 1 — Confirm the parameter changes

From [[Calibration Validation 2026-04-22]], document the exact recommended changes:
- U13: new RD floor value + new K-factor
- U14: new RD floor value + new K-factor
- U12: new RD floor value
- U19: new K-factor

Write these to a pre-deploy checklist at: `02 - Tiger Tournaments/Projects/Rankings/calibration-fix-pre-deploy-2026-04-23.md`

### Step 2 — Implement in `compute-rankings.js`

Apply the parameter changes. Changes touch two locations:
- RD floor: Glicko rating deviation initialization / floor enforcement
- K-factor: Phase 3 (Elo Computation) and Phase 4 (H2H Adjustments)

Make changes age-group-conditional (don't change parameters for age groups not listed above).

### Step 3 — Held-out validation

Before applying to production:
1. Reserve 10% of recent U13 and U14 games as held-out test set
2. Run ranking computation on training set
3. Measure Brier score on held-out set
4. Confirm improvement: U13 target ≤0.24, U14 target ≤0.24
5. Verify ~6,310 teams see >25-point rating changes as predicted

Document results at: `02 - Tiger Tournaments/Projects/Rankings/calibration-fix-validation-results-2026-04-23.md`

### Step 4 — Deploy post-DSS

Do NOT deploy before May 17. These are significant changes (6,310+ teams moving >25 points). They should be applied after DSS placements are finalized so they don't disrupt seeding.

Get Presten's sign-off before deploying — flag this as ready in your briefing.

## Acceptance criteria

- [ ] Parameter changes documented in pre-deploy checklist
- [ ] `compute-rankings.js` changes implemented (age-group-conditional)
- [ ] Held-out validation run: U13 Brier ≤0.24, U14 Brier ≤0.24
- [ ] Validation results note written
- [ ] Presten sign-off requested in briefing before deploy

## Why this matters

U13 Brier of 0.312 means the ranking engine's game outcome predictions are substantially wrong for this age group. This is not a minor calibration drift — it's a systematic error affecting accuracy for ~tens of thousands of teams. Fixing it post-DSS (May 17+, before June 1 calibration recalculation window) is the right timing.
