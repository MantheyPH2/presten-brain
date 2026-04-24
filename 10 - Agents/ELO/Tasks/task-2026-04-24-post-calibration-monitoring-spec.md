---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Post-Calibration Monitoring Spec — May 2026.md"
priority: high
due: 2026-04-30
---

# Task: Post-Calibration Monitoring Spec — May 18–20 Application Window

## Context

The Calibration Production Runbook (`02 - Tiger Tournaments/Projects/Rankings/Calibration Production Runbook — May 2026.md`) covers how to apply the calibration changes. It has a post-application monitoring section (Section 4), but that section is brief — it tells Presten to monitor rating volatility and run a daily snapshot for 3 days.

What Section 4 does not provide: specific thresholds. What counts as "too volatile"? What does a Brier score trend look like after a good calibration change vs a bad one? What is the trigger for rollback vs watchful waiting?

This task fills that gap. You have the Brier scores from the held-out validation (`Reports/calibration-held-out-validation-2026-04-23.md`). You know the expected magnitude of rating changes per age group. Write the monitoring spec with precise numbers, not narratives.

## What You Need to Do

### Step 1: Define Expected Rating Changes Per Age Group

For each age group being changed (U12, U13, U14, U19), document the expected direction and magnitude of rating changes from the calibration adjustment:

| Age Group | Change | Expected Effect on Ratings | Expected Effect on Rankings |
|-----------|--------|---------------------------|----------------------------|
| U13 | K-factor 32→24 | New games weighted less heavily → smoother convergence | Top teams (long dominant history) rise slightly; recent-form teams fall slightly |
| U13 | RD floor 50→75 | Higher minimum uncertainty → ratings spread narrows | All teams slightly closer to the mean |
| U14 | K-factor 32→28 | Similar to U13 but smaller magnitude | ... |
| U14 | RD floor 50→65 | ... | ... |
| U12 | RD floor 50→60 | ... | ... |
| U19 | K-factor 32→24 | Same as U13 K-factor | ... |

Fill in the expected effects based on your Brier score analysis. Be specific — "ratings spread narrows" is better than "things change."

### Step 2: Write Day-1 Verification Queries

Write the queries Presten runs within 24 hours of applying the calibration changes:

**2A: Rating volatility check — is any age group wildly different?**
```sql
SELECT age_group, gender,
  AVG(rating) AS avg_rating,
  STDDEV(rating) AS rating_std,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating) AS median,
  MAX(rating) - MIN(rating) AS rating_range
FROM rankings
WHERE age_group IN ('U12', 'U13', 'U14', 'U19')
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

Compare against the baseline from the Calibration Production Runbook (Section 1, V4). Document the acceptable range for each metric:
- Average rating should shift by no more than ±[X] points for any age group
- Standard deviation should [increase/decrease/stay similar] for U13 (based on K-factor reduction)
- Median should shift by no more than ±[Y] points

Fill in X and Y based on your analysis.

**2B: Top-10 stability check — do the top 10 teams still make sense?**
```sql
SELECT t.name, r.rank, r.rating, r.age_group, r.gender
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND r.rank <= 10
ORDER BY r.rank;
```

Define what "makes sense" means here: "If any team not in the top 20 before the calibration change appears in the top 10 after, this is a flag requiring investigation — K-factor reduction should not cause rank jumps of >20 positions in one recalculation."

Write the analogous check for U14G and U19B (or whatever the two highest-risk age groups are for the changes made).

### Step 3: Write Day-3 and Day-7 Follow-Up Queries

**Day-3: Brier score trend check**

Write the Brier score computation query from your held-out validation framework (adapted for live data). Compare to the baseline Brier scores:

| Age Group | Pre-application Brier | Post-application Brier (expected) | Threshold for "improvement confirmed" |
|-----------|----------------------|-----------------------------------|---------------------------------------|
| U13 | [from validation doc] | [projected] | ≤ [X] |
| U14 | [from validation doc] | [projected] | ≤ [Y] |

The "threshold for improvement confirmed" values should come from your held-out validation results — the projected Brier for the staged config.

**Day-7: Rank stability audit**

After 7 days, the nightly pipeline will have processed approximately 7 runs of new games. Write a query to check for teams whose rank has moved by more than 50 positions in the 7 days since calibration:

```sql
-- Significant rank movers post-calibration (requires a rank_history table or snapshots)
-- If no rank history: compare against the pre-calibration snapshot saved in the Runbook (V4)
```

Document the approach: if a rank_history table exists, use it. If not, Presten should have saved the V4 snapshot before application — compare current rankings against that.

### Step 4: Write the Rollback Decision Tree

Define the specific triggers that should initiate the rollback procedure (from Section 3 of the Calibration Production Runbook):

```
ROLLBACK TRIGGERS (any one of these = rollback):
1. Average rating for any age group shifts by more than [X] points vs baseline snapshot
2. Any team in the top 5 for any age group drops by more than 15 positions after one recompute
3. Brier score for U13 or U14 is WORSE than pre-calibration after Day-3 measurement
4. Rankings API returns errors or empty results after the recompute

WATCHFUL WAITING (monitor but do not roll back):
1. Average rating shifts by [X/2]–[X] points — recheck in 48 hours
2. Top-20 shuffle within expected range but 1–2 surprising teams
3. U12 or U19 Brier score unchanged (RD floor and K-factor changes have smaller effects)

ROLLBACK WINDOW:
- Within 48 hours of application: rollback is clean (revert calibration.json + recompute)
- 48–96 hours: rollback may introduce visible ranking instability but is still correct
- After 96 hours: rolling back may do more harm than good — escalate to Presten judgment
```

Fill in `[X]` from your analysis. Be specific.

### Step 5: Write the Monitoring Log Template

Write a simple template Presten fills in each day of monitoring:

```markdown
## Calibration Monitoring Log — May 18–20 Application

**Applied:** [date and time]
**Backup file:** config/calibration-backup-[date].json

### Day 1 (within 24 hours)
- U13G avg rating: [before] → [after] | Δ = [X] | Status: ✅/⚠️/🚨
- U14G avg rating: [before] → [after] | Δ = [X] | Status: ✅/⚠️/🚨
- U12G avg rating: [before] → [after] | Δ = [X] | Status: ✅/⚠️/🚨
- U19B avg rating: [before] → [after] | Δ = [X] | Status: ✅/⚠️/🚨
- Top-10 U13G: [names look correct?] | Status: ✅/⚠️/🚨
- Any rollback trigger hit? Yes / No

### Day 3
- U13 Brier score: [before] → [after] | Expected ≤ [threshold] | Status: ✅/⚠️/🚨
- U14 Brier score: [before] → [after] | Expected ≤ [threshold] | Status: ✅/⚠️/🚨
- Any rollback trigger hit? Yes / No

### Day 7
- U13G rank stability: [teams with >50 rank move = X] | Status: ✅/⚠️/🚨
- Overall assessment: Calibration confirmed / Calibration under review / Rollback initiated
```

## Deliverable

Write the monitoring spec to:
`02 - Tiger Tournaments/Projects/Rankings/Post-Calibration Monitoring Spec — May 2026.md`

The document must:
- Include the expected-change table (Step 1) with actual values, not narratives
- Include Day-1 queries with specific acceptable-range thresholds filled in
- Include Day-3 Brier comparison with projected values from the held-out validation
- Include the rollback decision tree with numeric triggers (not "significant change")
- Include the monitoring log template

## Definition of Done

- Expected rating changes are quantified (not "ratings may shift")
- Day-1 queries include explicit acceptable ranges (not "should be similar")
- Rollback triggers are numeric (not "if things look wrong")
- Monitoring log template is pre-filled with expected values
- Summary in your briefing: monitoring spec ready; rollback threshold for U13 is [X] points

## Why This Matters

Calibration changes will go live after DSS with minimal margin for error — the next major event after May 16 is June 1 (ECNL migration). If the calibration application causes a silent accuracy regression, Presten needs to catch it and reverse it within 48 hours. Without specific thresholds, "monitoring" means noticing something is wrong after it's too late. With specific thresholds, it means Presten runs three queries per day for a week and gets a clear red/yellow/green status.

## References

- `02 - Tiger Tournaments/Projects/Rankings/Calibration Production Runbook — May 2026.md` — Section 4 (current brief monitoring notes) + Section 3 (rollback procedure)
- `02 - Tiger Tournaments/Projects/Reports/calibration-held-out-validation-2026-04-23.md` — Brier score baselines for each age group
- `02 - Tiger Tournaments/Projects/Rankings/Calibration Validation 2026-04-22.md` — original validation findings
