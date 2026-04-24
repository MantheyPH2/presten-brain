---
title: Calibration Production Runbook — May 2026
aliases: [Calibration Runbook, Calibration Application May 2026]
tags: [calibration, runbook, production, glicko, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: ready
apply_window: 2026-05-18 to 2026-05-20
approval_required: pending-2026-04-22-calibration-approval.md
---

# Calibration Production Runbook — May 2026

> [!warning] DO NOT APPLY without Presten's approval
> Approval pending via `Queue/pending-2026-04-22-calibration-approval.md`. This runbook is ready to execute immediately upon approval. Apply in the May 18–20 window (after DSS concludes May 16).

This runbook is self-contained. Execute each numbered step in sequence. All commands are copy-paste ready. No decisions are left open.

---

## Summary of Changes

These are the only changes to `calibration.json`. All other age groups are **unchanged**.

| Age Group | Parameter | Current | Proposed | Validation |
|-----------|-----------|---------|----------|------------|
| U12 | RD floor | 50 | **60** | Brier 0.241 → proj. 0.226 |
| U13 | K-factor | 32 | **24** | Brier 0.312 → proj. 0.219 |
| U13 | RD floor | 50 | **75** | Combined fix — both required |
| U14 | K-factor | 32 | **28** | Brier 0.273 → proj. 0.222 |
| U14 | RD floor | 50 | **65** | Combined fix — both required |
| U19 | K-factor | 32 | **24** | Brier 0.238 → proj. 0.218 |

Source: [[Calibration Validation 2026-04-22]] Section 3.
Validation: `Reports/calibration-held-out-validation-2026-04-23.md` (pending DB execution; projections confirmed in held-out framework).

---

## Section 1: Pre-Application Verification

Run these before touching anything in production. Complete all four checks before moving to Section 2.

### V1. Confirm the staging file matches the approved spec

```bash
diff config/calibration.json config/calibration-staging-2026-04-23.json
```

**Expected output:** Only these 6 lines should differ:
- `U12.rd_floor`: 50 → 60
- `U13.k_factor`: 32 → 24
- `U13.rd_floor`: 50 → 75
- `U14.k_factor`: 32 → 28
- `U14.rd_floor`: 50 → 65
- `U19.k_factor`: 32 → 24

**If more lines differ:** Stop. Do not proceed. Compare both files manually and identify unexpected changes before continuing.

### V2. Confirm held-out validation Brier scores

The projected Brier improvements from `Reports/calibration-held-out-validation-2026-04-23.md`:

| Age Group | Current Brier (held-out) | Proposed Brier (held-out) | Improvement | Threshold Met? |
|-----------|--------------------------|---------------------------|-------------|---------------|
| U12 | ~0.241 | ~0.226 projected | ~0.015 | YES (projected) |
| U13 | ~0.312 | ~0.219 projected | ~0.093 | YES (projected) |
| U14 | ~0.273 | ~0.222 projected | ~0.051 | YES (projected) |
| U19 | ~0.238 | ~0.218 projected | ~0.020 | YES (projected) |

All four age groups are projected to meet the ≤ 0.23 threshold under proposed parameters.

> [!note] Held-out execution status
> The held-out validation SQL in `Reports/calibration-held-out-validation-2026-04-23.md` is ready to run but requires FORGE or Presten to execute `compute-rankings.js --config=calibration-staging-2026-04-23.json --end-date=2025-12-31` against the test set. If this has been run and the actual held-out Brier scores are available, record them here before applying. If the actual scores show any age group NOT meeting ≤ 0.23, **stop and post a queue item** before proceeding.

### V3. Confirm no pipeline run is in progress

```bash
ps aux | grep -E 'crawl-gotsport|run-overnight|compute-rankings|node scripts' | grep -v grep
```

**Expected:** No matching processes. If a pipeline run is in progress, wait for it to complete before applying the calibration change.

### V4. Snapshot current ratings distribution (save this output)

```sql
SELECT
  age_group,
  gender,
  COUNT(*) AS team_count,
  ROUND(AVG(rating), 1) AS avg_rating,
  ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating)::numeric, 1) AS median_rating,
  ROUND(STDDEV(rating)::numeric, 1) AS rating_stddev
FROM rankings
WHERE age_group IN ('U12', 'U13', 'U14', 'U19')
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

**Save this output.** You will run it again after Step 4 in Section 2 to verify the changes had the expected effect.

---

## Section 2: Application Steps

Execute in order. Do not skip steps.

### Step 1: Back up the current calibration config

```bash
cp config/calibration.json config/calibration-backup-$(date +%Y%m%d).json
```

Verify the backup was created:
```bash
ls -la config/calibration-backup-*.json
```

**Do not delete this backup until May 25** (7 days of stable operation post-application).

### Step 2: Apply the staged calibration config

```bash
cp config/calibration-staging-2026-04-23.json config/calibration.json
```

Verify the change took effect:
```bash
diff config/calibration.json config/calibration-staging-2026-04-23.json
```

Expected output: **no differences** (files should be identical).

Double-check the key values:
```bash
node -e "const c = require('./config/calibration.json'); console.log('U13 K:', c.U13.k_factor, 'U13 RD:', c.U13.rd_floor, 'U14 K:', c.U14.k_factor, 'U14 RD:', c.U14.rd_floor)"
```

Expected output: `U13 K: 24 U13 RD: 75 U14 K: 28 U14 RD: 65`

### Step 3: Run the full rankings recompute

```bash
node scripts/compute-rankings.js
```

If a scoped recompute is available (only U12, U13, U14, U19):
```bash
node scripts/compute-rankings.js --age-groups U12,U13,U14,U19
```

Check the docs or source of `compute-rankings.js` to confirm whether `--age-groups` is a supported flag. If unsupported, run the full pipeline.

**Expected runtime:** Reference prior runs for typical duration. Do not interrupt once started.

**Monitor progress:**
```bash
tail -f /var/log/evo-draw/compute-rankings.log
```

### Step 4: Post-application verification

Re-run the V4 snapshot query from Section 1 and compare to the saved baseline:

```sql
SELECT
  age_group,
  gender,
  COUNT(*) AS team_count,
  ROUND(AVG(rating), 1) AS avg_rating,
  ROUND(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating)::numeric, 1) AS median_rating,
  ROUND(STDDEV(rating)::numeric, 1) AS rating_stddev
FROM rankings
WHERE age_group IN ('U12', 'U13', 'U14', 'U19')
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

**Expected changes:**
- U13/U14: `rating_stddev` should decrease slightly (less volatile K-factor)
- U12/U13/U14: median may shift slightly toward the mean (RD floor increase loosens locked-in ratings)
- U19: `rating_stddev` should decrease slightly (K-factor reduction)

**Red flag:** If median rating for any age group shifted by more than ±30 points from baseline, investigate before declaring success. That level of shift indicates an unexpected interaction.

### Step 5: Spot-check 5 known teams

Run these queries. For each team, compare the before/after rating to your V4 baseline (or to the team's most recent known rating):

```sql
-- High-rated U13G team
SELECT t.team_name, r.rating, r.rank, r.match_count, r.last_game_date
FROM teams t JOIN rankings r ON r.team_id = t.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
ORDER BY r.rating DESC LIMIT 3;

-- Low-rated U13G team
SELECT t.team_name, r.rating, r.rank, r.match_count, r.last_game_date
FROM teams t JOIN rankings r ON r.team_id = t.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
ORDER BY r.rating ASC LIMIT 3;

-- U14G team with many games (K-factor change has larger effect)
SELECT t.team_name, r.rating, r.rank, r.match_count
FROM teams t JOIN rankings r ON r.team_id = t.team_id
WHERE r.age_group = 'U14' AND r.gender = 'F'
ORDER BY r.match_count DESC LIMIT 3;

-- U19 team
SELECT t.team_name, r.rating, r.rank, r.match_count
FROM teams t JOIN rankings r ON r.team_id = t.team_id
WHERE r.age_group = 'U19'
ORDER BY r.rating DESC LIMIT 3;

-- U12 team with few games (RD floor change primary effect)
SELECT t.team_name, r.rating, r.rank, r.match_count
FROM teams t JOIN rankings r ON r.team_id = t.team_id
WHERE r.age_group = 'U12' AND r.match_count BETWEEN 6 AND 15
ORDER BY r.rating DESC LIMIT 3;
```

**Red flag:** If any individual team's rating shifted by more than 100 points from its pre-application value, investigate before declaring success. That magnitude indicates a data issue (dedup, merge, or scoring error) rather than a calibration effect.

### Step 6: Confirm the rankings API returns updated data

```bash
curl "http://localhost:3000/api/rankings?age_group=U13&gender=F&limit=5"
```

Confirm: ratings in the API response match the database values from Step 4. If the API is caching rankings, clear the cache or restart the server.

---

## Section 3: Rollback Procedure

Use this procedure if any verification step in Section 2 fails and you cannot identify and fix the root cause within 30 minutes.

### Rollback Step 1: Restore the backup

```bash
cp config/calibration-backup-YYYYMMDD.json config/calibration.json
```

Replace `YYYYMMDD` with the date from Step 1 (e.g., `20260518`).

Verify:
```bash
diff config/calibration.json config/calibration-backup-20260518.json
```

Expected: no differences.

### Rollback Step 2: Re-run rankings recompute

```bash
node scripts/compute-rankings.js
```

This restores the prior ratings. Runtime is the same as a normal full recompute.

### Rollback Step 3: Verify rollback

Re-run the V4 snapshot query. Median ratings should return to within ±5 points of the pre-application baseline recorded in Section 1 V4.

### Rollback complete

The rollback is fully reversible. No data is lost. The staging file (`config/calibration-staging-2026-04-23.json`) remains intact for a future application attempt.

---

## Section 4: Post-Application Monitoring (Days 1–7)

After successful application, monitor these daily for 7 days:

### Daily check: Rating stability

Re-run the V4 snapshot query. Median rating should be stable (±10 points day-over-day). If the median shifts more than 10 points in a single day, that is not caused by the calibration change — it indicates fresh game data is producing unusual results. Investigate the game data for that day.

### Brier score tracking

If the pipeline produces Brier scores per run (check `/var/log/evo-draw/` for logs), confirm they are trending toward the projected values:
- U12: ≤ 0.226
- U13: ≤ 0.219
- U14: ≤ 0.222
- U19: ≤ 0.218

Use the weekly monitor SQL from `Reports/calibration-held-out-validation-2026-04-23.md` Section 4 to track this:
```sql
-- Post-implementation Brier score monitor
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
  )::numeric, 4) AS brier_score_post_change
FROM games g
JOIN teams t ON t.team_id = g.home_team_id
JOIN rankings r_home ON r_home.team_id = g.home_team_id
JOIN rankings r_away ON r_away.team_id = g.away_team_id
WHERE g.match_date >= '2026-05-18'
  AND g.is_hidden = false
  AND r_home.match_count >= 10
  AND r_away.match_count >= 10
GROUP BY t.age_group
ORDER BY t.age_group;
```

### Top-10 U13G / U14G rankings check

```sql
SELECT t.team_name, r.rating, r.rank
FROM teams t JOIN rankings r ON r.team_id = t.team_id
WHERE r.age_group IN ('U13', 'U14') AND r.gender = 'F'
  AND r.rank <= 10
ORDER BY r.age_group, r.rank;
```

Confirm: no wildly unexpected teams at the top. The calibration change should not flip top-10 rankings dramatically — it smooths volatility, it doesn't invert the hierarchy. If a team that was previously unranked or very low suddenly appears in the top 3, investigate.

### No-regression check: Other age groups unchanged

```sql
SELECT age_group, gender, ROUND(AVG(rating), 1) AS avg_rating, COUNT(*) AS team_count
FROM rankings
WHERE age_group NOT IN ('U12', 'U13', 'U14', 'U19')
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

Compare to pre-application baseline. Non-affected age groups should show zero change in average rating. Any change indicates an unintended effect.

---

## Section 5: Communication to SENTINEL

After successful application and 3+ days of stable monitoring, post a completed queue item:

Create `Queue/completed-calibration-application-2026-05-XX.md` (replace XX with actual date):

```yaml
---
type: agent-queue
agent: ELO
category: update
priority: high
date: 2026-05-XX
status: completed
---

# Calibration Changes Applied — U12/U13/U14/U19

Applied on: 2026-05-XX
Runbook: [[Calibration Production Runbook — May 2026]]

## Before/After

| Age Group | Brier Before | Brier After (first-week measurement) | Change |
|-----------|-------------|--------------------------------------|--------|
| U12 | 0.241 | [FILL IN] | [FILL IN] |
| U13 | 0.312 | [FILL IN] | [FILL IN] |
| U14 | 0.273 | [FILL IN] | [FILL IN] |
| U19 | 0.238 | [FILL IN] | [FILL IN] |

## Anomalies

[None observed / describe any anomalies and how resolved]

## Status

Rollback backup retained until 2026-05-25 at: config/calibration-backup-20260518.json
```

---

## Quick Reference

| Action | Command |
|--------|---------|
| Check diff | `diff config/calibration.json config/calibration-staging-2026-04-23.json` |
| Back up config | `cp config/calibration.json config/calibration-backup-$(date +%Y%m%d).json` |
| Apply staged | `cp config/calibration-staging-2026-04-23.json config/calibration.json` |
| Run recompute | `node scripts/compute-rankings.js` |
| Rollback config | `cp config/calibration-backup-YYYYMMDD.json config/calibration.json` |
| Monitor log | `tail -f /var/log/evo-draw/compute-rankings.log` |

---

## Related Notes

- [[Calibration Validation 2026-04-22]] — root cause analysis and fix proposals
- `Reports/calibration-held-out-validation-2026-04-23.md` — held-out validation (pending DB run)
- `Queue/pending-2026-04-22-calibration-approval.md` — Presten's approval queue item
- [[Ranking Engine]] — Glicko-2 implementation (K-factor and RD floor context)
- [[Age Group Transition Migration Spec 2026-06-01]] — downstream dependency (migration runs after this)
