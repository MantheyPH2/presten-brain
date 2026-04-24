---
title: Post-Calibration Monitoring Spec — May 2026
tags: [calibration, monitoring, brier-score, rankings, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: ELO Agent
status: ready
related_task: task-2026-04-24-post-calibration-monitoring-spec
application_window: "May 18–20, 2026"
---

# Post-Calibration Monitoring Spec — May 2026

> Monitoring protocol for the May 18–20 calibration application. Covers Day 1 (within 24 hours), Day 3, and Day 7 checks. Numeric triggers — no judgment required for pass/fail decisions. Extends Section 4 of the [[Calibration Production Runbook — May 2026]] with specific thresholds.

---

## Step 1: Expected Rating Changes Per Age Group

Each calibration change has a predictable directional effect after the full `compute-rankings.js` rerun processes all 570K+ games with new parameters. These are not adjustments applied to existing ratings — the entire rating history is recomputed from scratch.

| Age Group | Change | Expected Effect on Ratings | Expected Effect on Rankings |
|-----------|--------|---------------------------|----------------------------|
| U13 | K-factor 32→24 | Per-game rating movement drops to 75% of previous magnitude. Teams rated too high from tournament burst runs (high K-factor compounding) regress toward their true mean. Estimated: 50% of U13 teams see a rating shift >25 points toward mean. Extreme outliers (>200 pts from true mean) shift 30–60 pts. Average team: shift <15 pts. | Top teams with consistent long dominance rise slightly (were slightly suppressed by high K volatility). Recent-form teams with short winning streaks fall slightly. Net top-10 movement: expect 1–3 teams to move by more than 5 rank positions. |
| U13 | RD floor 50→75 | Minimum uncertainty rises. Teams previously "locked in" at RD≈50 now have RD≈75 — their ratings are slightly more open to future movement. Effect on the recompute itself: modest convergence of extreme ratings. Estimated avg rating shift from RD floor change alone: 5–10 pts toward mean for teams whose prior RD was below 75. | Minimal direct ranking impact. Secondary effect: the system will be less "certain" about U13 ratings in the first weeks post-change, which is correct. |
| U14 | K-factor 32→28 | Per-game rating movement drops to 87.5% of previous. Smaller magnitude than U13. Estimated: 40% of U14 teams see a shift >25 points. Avg affected team: shift 15–25 pts toward mean. | Comparable to U13 but smaller magnitude. Expect 1–2 top-10 movements >5 rank positions. |
| U14 | RD floor 50→65 | Same direction as U13 RD floor change, smaller magnitude. Teams previously locked at RD<65 see slight convergence. Estimated avg shift from this change alone: 3–7 pts. | Minimal direct ranking impact. |
| U12 | RD floor 50→60 | Smallest effect of all changes. RD floor raises 10 pts only. Few teams in U12 were hitting the RD floor hard given lower game volume. Estimated avg shift: 3–8 pts for affected teams. | Near-zero ranking impact expected. |
| U19 | K-factor 32→24 | Same K-factor reduction as U13. However U19 teams play fewer games per season — so the absolute number of games affected is lower. Effect: teams that rode a tournament run (several wins in one weekend, K=32 compounding) regress more toward mean. Estimated: 20% of U19 teams see shift >25 pts. | Low-to-medium ranking impact. U19 pool is smaller; individual team movements are more visible. |

### Key Baselines to Record (from Calibration Production Runbook V4)

Before applying any changes, save the V4 baseline snapshot:

| Age Group | Gender | Avg Rating (pre-change) | Median Rating (pre-change) | Std Dev (pre-change) |
|-----------|--------|------------------------|--------------------------|---------------------|
| U12 | F | _[record before applying]_ | _[record]_ | _[record]_ |
| U12 | M | _[record]_ | _[record]_ | _[record]_ |
| U13 | F | _[record]_ | _[record]_ | _[record]_ |
| U13 | M | _[record]_ | _[record]_ | _[record]_ |
| U14 | F | _[record]_ | _[record]_ | _[record]_ |
| U14 | M | _[record]_ | _[record]_ | _[record]_ |
| U19 | F | _[record]_ | _[record]_ | _[record]_ |
| U19 | M | _[record]_ | _[record]_ | _[record]_ |

Fill this in by running the V4 query from the Runbook before applying the calibration changes.

---

## Step 2: Day-1 Verification Queries (Run Within 24 Hours of Application)

### 2A: Rating Volatility Check

Run within 24 hours of completing Step 3 of the Runbook (after `compute-rankings.js` finishes).

```sql
SELECT age_group, gender,
  AVG(rating)::int AS avg_rating,
  ROUND(STDDEV(rating), 1) AS rating_std,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating)::int AS median,
  (MAX(rating) - MIN(rating))::int AS rating_range,
  COUNT(*) AS team_count
FROM rankings
WHERE age_group IN ('U12', 'U13', 'U14', 'U19')
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

Compare each row against the V4 baseline. Apply these acceptance ranges:

| Metric | Acceptable Change | Watchful Waiting | Rollback Trigger |
|--------|------------------|-----------------|-----------------|
| Average rating shift (any age group) | ≤ ±15 pts | ±15–30 pts | > ±30 pts |
| Median rating shift (any age group) | ≤ ±20 pts | ±20–40 pts | > ±40 pts |
| Standard deviation (U13, U14) | Should **decrease** by 5–20 pts | Unchanged ±5 pts | Increases by >10 pts |
| Standard deviation (U12, U19) | Approx unchanged ±10 pts | Change > 15 pts | Change > 25 pts |

**Why std deviation decreases for U13/U14:** The K-factor reduction smooths extreme ratings. If standard deviation increases, it means the recompute produced more spread — not less — which contradicts the expected behavior and suggests a pipeline error.

### 2B: Top-10 Stability Check — U13G

```sql
SELECT t.name, r.rank, r.rating, r.age_group, r.gender
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND r.rank <= 10
ORDER BY r.rank;
```

**Acceptance criteria:** No team that was outside the top 20 before the calibration change should appear in the top 10 after. The K-factor reduction should not cause rank jumps of >20 positions in a single recomputation. If any top-10 team is unrecognizable as a strong U13G program, investigate before proceeding.

Also run for U14G (second-highest DSS exposure) and U19B (K-factor change same magnitude as U13):

```sql
-- U14G top 10
SELECT t.name, r.rank, r.rating FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U14' AND r.gender = 'F' AND r.rank <= 10
ORDER BY r.rank;

-- U19B top 10
SELECT t.name, r.rank, r.rating FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U19' AND r.gender = 'M' AND r.rank <= 10
ORDER BY r.rank;
```

**Flag for investigation (not immediate rollback):** Any team that appears in a top-10 after the change that was previously ranked outside the top 25. Compare against the V4 snapshot saved in the Runbook pre-flight.

---

## Step 3: Day-3 and Day-7 Follow-Up Queries

### Day-3: Brier Score Trend Check

Run 3 days after calibration application (by May 21–23). This measures prediction accuracy on games played since the change.

```sql
-- Post-application Brier score monitor (run starting Day 3)
SELECT
  t.age_group,
  COUNT(*) AS games_since_change,
  ROUND(AVG(
    POWER(
      (1.0 / (1.0 + POWER(10, (r_away.rating - r_home.rating) / 400.0)))
      - CASE
          WHEN g.home_score > g.away_score THEN 1.0
          WHEN g.home_score = g.away_score THEN 0.5
          ELSE 0.0
        END,
      2
    )
  ), 4) AS brier_score_post_change
FROM games g
JOIN teams t ON t.id = g.home_team_id
JOIN rankings r_home ON r_home.team_id = g.home_team_id
JOIN rankings r_away ON r_away.team_id = g.away_team_id
WHERE g.match_date >= '2026-05-18'
  AND g.is_hidden = false
  AND r_home.match_count >= 10
  AND r_away.match_count >= 10
GROUP BY t.age_group
ORDER BY t.age_group;
```

**Compare against these targets from the held-out validation:**

| Age Group | Pre-Application Brier | Post-Application Target | "Improvement Confirmed" Threshold |
|-----------|----------------------|------------------------|----------------------------------|
| U12       | 0.241                | ≤ 0.226 projected      | ≤ 0.232 (6-pt margin for live vs held-out variance) |
| U13       | 0.312                | ≤ 0.219 projected      | ≤ 0.235 (Day-3 games are limited; expect high variance early) |
| U14       | 0.273                | ≤ 0.222 projected      | ≤ 0.240 |
| U19       | 0.238                | ≤ 0.218 projected      | ≤ 0.244 |

**Day-3 note on game volume:** Day-3 will have few post-change games (May 18–21 is a non-tournament weekday window for most leagues). Low game count (<100 per age group) means the Day-3 Brier is high-variance. Do not roll back based on Day-3 Brier alone unless a clearly dramatic regression (>0.30 for U13, >0.28 for U14) appears. Use Day-7 for the decisive measurement.

### Day-7: Rank Stability Audit

Run 7 days after application (by May 25–27). The nightly pipeline will have run ~7 times with new parameters, incorporating new game results.

If a `rank_history` table exists:

```sql
-- Teams with >50 rank-position movement in 7 days post-calibration
SELECT
  t.name, t.age_group, t.gender,
  rh.rank AS rank_at_application,
  r.rank AS rank_now,
  ABS(r.rank - rh.rank) AS rank_change
FROM rankings r
JOIN teams t ON t.id = r.team_id
JOIN rank_history rh ON rh.team_id = r.team_id
  AND rh.snapshot_date = '2026-05-18'  -- date of application
WHERE ABS(r.rank - rh.rank) > 50
  AND r.age_group IN ('U13', 'U14')
ORDER BY rank_change DESC
LIMIT 20;
```

If no `rank_history` table exists: compare against the V4 snapshot saved before the application. The V4 snapshot from the Runbook pre-flight serves as the Day-0 baseline.

**Acceptance criteria:** Teams moving 50+ rank positions in 7 days should be rare (<5% of any age group). If more than 10% of U13 or U14 teams have moved 50+ positions, the K-factor change is producing more volatility than expected — flag for investigation.

---

## Step 4: Rollback Decision Tree

```
ROLLBACK TRIGGERS — any one of these = execute rollback immediately:

1. Average rating for any age group shifts by more than ±30 points vs V4 baseline snapshot
2. Any team in the top 5 for U13G or U14G drops more than 15 positions after one recompute
3. Standard deviation for U13 or U14 INCREASES by more than 10 points vs pre-change baseline
   (an increase means the recompute is producing more spread, not less — a pipeline error)
4. Brier score for U13 or U14 is WORSE than the pre-application value after Day-7 measurement
   (i.e., Brier post-change > 0.312 for U13, > 0.273 for U14)
5. Rankings API returns errors or empty results after the recompute

WATCHFUL WAITING — monitor closely but do not roll back:
1. Average rating shift between ±15–30 pts — recheck at Day-3
2. Median shift up to ±40 pts — recheck at Day-7
3. 1–3 top-10 teams shift by 5–20 positions (within expected range for K-factor smoothing)
4. U12 or U19 Brier score unchanged at Day-3 (small effects, low weekend game volume)
5. U13 Day-3 Brier is 0.235–0.260 (too early to confirm improvement; wait for Day-7)

ROLLBACK WINDOW:
- Within 48 hours: rollback is clean (revert calibration.json + rerun compute-rankings.js)
- 48–96 hours: rollback may produce visible ranking instability for 1–2 days but is still correct
- After 96 hours: rolling back may do more harm than good. Escalate to Presten judgment.
  At 96+ hours, the nightly pipeline has incorporated new games with new parameters.
  Reverting the parameters will re-recompute all those games with old parameters, producing
  another round of ranking movement. Only roll back if a rollback trigger condition persists.
```

---

## Step 5: Monitoring Log Template

Fill this in daily during the monitoring window (May 18–25). Save as a file or in this document's running log section.

```markdown
## Calibration Monitoring Log — May 2026 Application

**Applied:** [date and time, e.g., 2026-05-19 14:30]
**Backup file:** config/calibration-backup-[date].json
**compute-rankings.js completed:** [timestamp]

---

### Day 1 (run within 24 hours of completion)

U12F avg rating: [before] → [after] | Δ = [X] pts | ✅/⚠️/🚨
U12M avg rating: [before] → [after] | Δ = [X] pts | ✅/⚠️/🚨
U13F avg rating: [before] → [after] | Δ = [X] pts | ✅/⚠️/🚨
U13M avg rating: [before] → [after] | Δ = [X] pts | ✅/⚠️/🚨
U14F avg rating: [before] → [after] | Δ = [X] pts | ✅/⚠️/🚨
U14M avg rating: [before] → [after] | Δ = [X] pts | ✅/⚠️/🚨
U19F avg rating: [before] → [after] | Δ = [X] pts | ✅/⚠️/🚨
U19M avg rating: [before] → [after] | Δ = [X] pts | ✅/⚠️/🚨

U13F std dev: [before] → [after] | Change = [X] | Expected: decrease | ✅/⚠️/🚨
U14F std dev: [before] → [after] | Change = [X] | Expected: decrease | ✅/⚠️/🚨

Top-10 U13G: [list top 3 names — do they look correct?] | ✅/⚠️/🚨
Top-10 U14G: [list top 3 names] | ✅/⚠️/🚨

Rollback trigger hit? Yes / No
If No: continue monitoring.
If Yes: ROLLBACK NOW — see Calibration Production Runbook Section 3.

---

### Day 3 (run ~2026-05-21)

Post-change Brier scores (run the Day-3 query above):
U12: [value] | Target ≤ 0.232 | ✅/⚠️/🚨 (note: low game volume expected)
U13: [value] | Target ≤ 0.235 (Day-3) | ✅/⚠️/🚨
U14: [value] | Target ≤ 0.240 (Day-3) | ✅/⚠️/🚨
U19: [value] | Target ≤ 0.244 (Day-3) | ✅/⚠️/🚨

Game count for Brier (note if <100 per group — high variance expected):
U12: [count] | U13: [count] | U14: [count] | U19: [count]

Rollback trigger hit? Yes / No

---

### Day 7 (run ~2026-05-25)

Post-change Brier scores (definitive measurement):
U12: [value] | Target ≤ 0.226 (projected) | ✅/⚠️/🚨
U13: [value] | Target ≤ 0.219 (projected) | ✅/⚠️/🚨
U14: [value] | Target ≤ 0.222 (projected) | ✅/⚠️/🚨
U19: [value] | Target ≤ 0.218 (projected) | ✅/⚠️/🚨

U13G rank stability: teams with >50 position move = [count] | Target < 5% of pool | ✅/⚠️/🚨
U14G rank stability: teams with >50 position move = [count] | ✅/⚠️/🚨

Rollback trigger hit? Yes / No

Overall assessment:
- [ ] Calibration confirmed — Brier improved as projected, rankings stable
- [ ] Calibration under review — [note what is being monitored]
- [ ] Rollback initiated — [note reason and time of rollback]
```

---

## Post-Monitoring Action

If Day-7 confirms improvement (all four Brier scores at or below targets, no rollback triggers):

1. Delete backup file after Day-7 confirmation: `rm config/calibration-backup-[date].json`
2. Post a queue item: `Queue/completed-calibration-application-2026-05-XX.md` (format per Config)
3. Note the confirmed Brier improvements for the next ELO briefing

If Day-7 shows partial improvement (some age groups improved, one did not):
- Do not roll back if the failed age group Brier is still better than pre-change
- Flag to SENTINEL for review: is the non-improving age group a monitoring variance issue or a genuine non-improvement?
- Post a queue item with the specific finding

---

## Related

- [[Calibration Production Runbook — May 2026]] — application procedure this document extends
- [[Calibration Held-Out Validation — 2026-04-23]] — Brier baselines and targets
- [[Calibration Validation 2026-04-22]] — root cause analysis and parameter rationale
- [[Ranking Engine]] — recompute pipeline that must run after calibration change
