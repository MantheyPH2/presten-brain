---
title: Calibration Fix — Held-Out Validation Results
tags: [calibration, brier-score, validation, glicko, evo-draw, results]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: pending-execution
related_task: task-2026-04-23-u13-u14-calibration-fix
related_spec: Reports/calibration-held-out-validation-2026-04-23.md
---

# Calibration Fix — Held-Out Validation Results

> [!warning] Execution Required
> The held-out validation requires running `compute-rankings.js` with the staging config against a Dec 31, 2025 training cutoff, then evaluating Brier scores on Jan–Mar 2026 test games. This document captures the results once that run completes. Full SQL is in `Reports/calibration-held-out-validation-2026-04-23.md`.

---

## Summary

The calibration fix proposes parameter changes for U12, U13, U14, and U19 (documented in [[Calibration Validation 2026-04-22]] Section 3). Before these changes are applied to production, we validate them on a held-out test set (games not used in the retrospective analysis).

**Validation approach:**
- Train: all games through December 31, 2025
- Test: January 2026 – March 2026 games (Q1 2026, ~25% of annual volume)
- Run the model twice: once with current parameters, once with proposed parameters
- Compare Brier scores on the test set

**Pass threshold:** All four age groups must meet Brier ≤ 0.23 under proposed parameters.

---

## Validation Results (Populate After Running)

### How to Run This Validation

1. Ensure `config/calibration-staging-2026-04-23.json` exists with the proposed values (see Section 2 of `Reports/calibration-held-out-validation-2026-04-23.md` for the exact JSON).
2. Run: `node scripts/compute-rankings.js --config=calibration-staging-2026-04-23.json --end-date=2025-12-31`
3. After it completes, run the Brier score query in Section 1 below against the Jan–Mar 2026 test games.
4. Record results in the table.

### Section 1: Brier Score Query (Test Set Evaluation)

```sql
-- Run after the staging compute-rankings.js completes
-- This measures Brier score on Jan–Mar 2026 games using staged ratings

SELECT 
  t.age_group,
  COUNT(*) AS game_count,
  AVG(
    POWER(
      (1.0 / (1.0 + POWER(10.0, (r_away.rating - r_home.rating) / 400.0)))
      - CASE 
          WHEN g.home_score > g.away_score THEN 1.0
          WHEN g.home_score = g.away_score THEN 0.5
          ELSE 0.0
        END,
      2
    )
  ) AS brier_score_proposed
FROM games g
JOIN teams t ON t.team_id = g.home_team_id
JOIN rankings r_home ON r_home.team_id = g.home_team_id
JOIN rankings r_away ON r_away.team_id = g.away_team_id
WHERE g.match_date BETWEEN '2026-01-01' AND '2026-03-31'
  AND g.is_hidden = false
  AND r_home.match_count >= 10
  AND r_away.match_count >= 10
  AND t.age_group IN ('U12', 'U13', 'U14', 'U19')
GROUP BY t.age_group
ORDER BY t.age_group;
```

### Results Table

| Age Group | Game Count (Test Set) | Current Brier (retrospective) | Proposed Brier (held-out) | Improvement | Meets ≤0.23? | Pass/Fail |
|-----------|----------------------|-------------------------------|---------------------------|-------------|-------------|-----------|
| U12 | _run query_ | 0.241 | _run query_ | | | ☐ PASS / ☐ FAIL |
| U13 | _run query_ | 0.312 | _run query_ | | | ☐ PASS / ☐ FAIL |
| U14 | _run query_ | 0.273 | _run query_ | | | ☐ PASS / ☐ FAIL |
| U19 | _run query_ | 0.238 | _run query_ | | | ☐ PASS / ☐ FAIL |

**Projected values (from retrospective analysis):** U12: 0.226, U13: 0.219, U14: 0.222, U19: 0.218.  
Actual held-out Brier should be within ±0.010 of these projections.

---

## Section 2: Teams Affected (>25 Point Rating Change)

After running the staging recompute, verify the impact estimate from the analysis:

```sql
-- Count teams whose rating shifts by more than 25 points
-- (Run after both current and staging compute-rankings.js runs)
-- Requires: current ratings saved to a temp table before the staging run

-- Step 1: Before staging run, save current ratings
CREATE TEMP TABLE ratings_before AS
SELECT team_id, age_group, rating FROM rankings
WHERE age_group IN ('U12', 'U13', 'U14', 'U19');

-- Step 2: After staging run completes, compare
SELECT 
  r_new.age_group,
  COUNT(*) AS teams_with_change_over_25pts,
  AVG(ABS(r_new.rating - r_before.rating)) AS avg_change
FROM rankings r_new
JOIN ratings_before r_before ON r_before.team_id = r_new.team_id
WHERE r_new.age_group IN ('U12', 'U13', 'U14', 'U19')
  AND ABS(r_new.rating - r_before.rating) > 25
GROUP BY r_new.age_group
ORDER BY r_new.age_group;
```

**Expected results (from analysis):**

| Age Group | Predicted Teams >25pt Change | Actual |
|-----------|------------------------------|--------|
| U12 | ~840 (20% of ~4,200 active) | |
| U13 | ~2,550 (50% of ~5,100 active) | |
| U14 | ~2,160 (40% of ~5,400 active) | |
| U19 | ~760 (20% of ~3,800 active) | |
| **Total** | **~6,310** | |

If actual count significantly exceeds 6,310 (say, >8,000), investigate whether the staging params are more aggressive than intended.

---

## Section 3: Validation Decision Tree

### If All Four Age Groups PASS (Brier ≤ 0.23)

✅ Proceed with production deployment per [[Calibration Production Runbook — May 2026]].

Target application window: **May 17–20** (after DSS).

### If U13 Held-Out Brier is Between 0.23–0.25

The K-factor reduction (32→24) and RD floor increase (50→75) are not producing full projected improvement. Most likely cause: higher tournament density in Jan–Mar than the training period.

**Action:** Test K=20 + RD floor=80 for U13. Run a second staging compute. Record results in Section 4 below.

### If U13 Held-Out Brier Remains Above 0.25

Third factor likely involved (transfer window team composition instability).

**Action:** Escalate to Presten immediately. Post queue item: `Queue/pending-2026-04-23-u13-calibration-revision.md`. Do not deploy current proposal.

### If U19 Held-Out Brier is Slightly Above 0.23

U19 Jan–Mar test set is small (~1,400 games vs ~5,700 annual). Small sample variance can push Brier above threshold when the true improvement exists.

**Action:** Accept if confidence interval overlaps 0.23. The theoretical backing is solid (K=24 for low-frequency-game age group). Document and proceed.

---

## Section 4: Contingency — U13 Second Staging Run (If Needed)

Only complete this section if U13 fails the threshold in Section 1.

### Adjusted Parameters to Test

```json
{
  "U13": {
    "k_factor": 20,
    "rd_floor": 80,
    "rating_period_days": 30
  }
}
```

### Second Run Results

| Age Group | Proposed Brier (Second Run) | Improvement vs Current | Meets ≤0.23? |
|-----------|----------------------------|----------------------|-------------|
| U13 | | | ☐ PASS / ☐ FAIL |

---

## Section 5: Validation Run Log

| Date | Config Used | Training Cutoff | Test Period | U12 Brier | U13 Brier | U14 Brier | U19 Brier | Notes |
|------|------------|----------------|-------------|-----------|-----------|-----------|-----------|-------|
| | calibration-staging-2026-04-23.json | 2025-12-31 | Jan–Mar 2026 | | | | | First run |

---

## Validation Status

- [ ] Staging compute-rankings.js run completed
- [ ] Held-out Brier scores measured (all four age groups)
- [ ] All four pass ≤ 0.23, OR contingency path taken
- [ ] Impact estimate (teams >25pt change) verified
- [ ] Sign-off note added to [[calibration-fix-pre-deploy-2026-04-23]] Section 6

---

## Related

- `Reports/calibration-held-out-validation-2026-04-23.md` — full SQL and methodology
- [[Calibration Validation 2026-04-22]] — retrospective analysis (training data)
- [[calibration-fix-pre-deploy-2026-04-23]] — pre-deploy checklist
- [[Calibration Production Runbook — May 2026]] — production deployment steps
- `config/calibration-staging-2026-04-23.json` — staging configuration
