---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-23
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Calibration Production Runbook — May 2026.md"
priority: high
due: 2026-04-26
---

# Task: Write Calibration Production Runbook for U12/U13/U14/U19 Changes

## Context

The calibration validation is complete (`Reports/calibration-held-out-validation-2026-04-23.md`). The staged config exists (`config/calibration-staging-2026-04-23.json`). Presten's approval via `pending-2026-04-22-calibration-approval.md` is the only blocker. The target application window is **May 18–20** (after DSS concludes on May 16).

That is 25 days away. When approval arrives, execution must be immediate and error-free — no time to reconstruct the plan in the moment.

This task: write the production runbook now, so Presten can approve AND execute in the same session. The runbook contains every command, every verification query, and every rollback step. No decisions left open. ELO's analysis work is done; this is documentation work.

## Proposed Changes (from Calibration Validation 2026-04-22.md, Section 3)

| Age Group | Parameter | Current | Proposed |
|-----------|-----------|---------|----------|
| U12 | RD floor | 50 | 60 |
| U13 | K-factor | 32 | 24 |
| U13 | RD floor | 50 | 75 |
| U14 | K-factor | 32 | 28 |
| U14 | RD floor | 50 | 65 |
| U19 | K-factor | 32 | 24 |

All other age groups unchanged.

## What to Produce

### Section 1: Pre-Application Verification (Run Before Changing Anything)

**V1. Confirm calibration staging file matches the approved spec:**
```bash
diff config/calibration.json config/calibration-staging-2026-04-23.json
```
Expected: only the 6 fields above differ. If more fields differ, stop and investigate.

**V2. Confirm held-out validation passed:**
State the Brier scores from `Reports/calibration-held-out-validation-2026-04-23.md`:
| Age Group | Held-out Brier (current) | Held-out Brier (proposed) | Pass? |
|-----------|--------------------------|---------------------------|-------|
| U12 | ? | ? | ? |
| U13 | ? | ? | ? |
| U14 | ? | ? | ? |
| U19 | ? | ? | ? |

All four must show improvement. If any showed regression in the held-out validation, note it here and flag it in the runbook header.

**V3. Confirm no pipeline run is in progress:**
```bash
# Check for active pipeline processes
ps aux | grep -E 'crawl-gotsport|run-overnight|ranking' | grep -v grep
```
Do not apply calibration during an active pipeline run.

**V4. Snapshot current ratings distribution:**
```sql
-- Save baseline before changes
SELECT age_group, gender, COUNT(*) as team_count, 
       AVG(rating) as avg_rating, 
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating) as median_rating
FROM rankings
WHERE age_group IN ('U12', 'U13', 'U14', 'U19')
GROUP BY age_group, gender
ORDER BY age_group, gender;
```
Save this output. You will compare it against post-application results.

### Section 2: Application Steps (In Order)

**Step 1: Back up current calibration config**
```bash
cp config/calibration.json config/calibration-backup-$(date +%Y%m%d).json
```
Verify: `ls -la config/calibration-backup-*.json`

**Step 2: Apply the staged config**
```bash
cp config/calibration-staging-2026-04-23.json config/calibration.json
```
Verify: `diff config/calibration.json config/calibration-staging-2026-04-23.json` — should show no differences.

**Step 3: Run the full ranking recompute for affected age groups**
Document the exact command to run the ranking pipeline limited to U12, U13, U14, U19. If a scoped recompute is not supported (pipeline always runs all age groups), run the full pipeline.
```bash
# Full pipeline (if scoped run not available):
node scripts/compute-rankings.js
# Or the appropriate command — confirm from existing pipeline docs
```
Expected runtime: [document from prior runs]

**Step 4: Post-application verification query**
Re-run the V4 snapshot query. Compare before/after:
- U13 rating variance should DECREASE (K-factor reduced → smoother ratings)
- U12, U13, U14 ratings should be less volatile overall (higher RD floor → slower convergence, more stable)
- U19 ratings should stabilize (K-factor reduced)

If median ratings shifted by more than ±30 points for any age group, flag it — this indicates an unexpected interaction.

**Step 5: Spot check 5 known teams**
For each of these teams (or equivalent well-known teams in your DB), compare before/after rating:
- 1 high-rated U13G team (expected: rating slightly lower, more stable)
- 1 low-rated U13G team (expected: rating slightly higher, more stable)
- 1 U14G team with many games (expected: K-factor change has larger effect on teams with many games)
- 1 U19 team (expected: rating more stable after K-factor reduction)
- 1 U12 team with few games (expected: RD floor change increases minimum uncertainty)

Document the before/after ratings for each. If a team's rating changed by more than 100 points, investigate before proceeding.

**Step 6: Confirm rankings API returns updated data**
```bash
curl "http://localhost:3000/api/rankings?age_group=U13&gender=F&limit=5"
```
Confirm ratings in the API response match the updated DB values.

### Section 3: Rollback Procedure

If any verification step fails:

```bash
# Restore backup
cp config/calibration-backup-YYYYMMDD.json config/calibration.json

# Re-run ranking recompute to restore prior state
node scripts/compute-rankings.js

# Verify rollback by re-running V4 snapshot and comparing to baseline
```

Rollback is safe as long as the backup file exists. Do not delete backup files until the new calibration has been running for 7+ days without issues.

### Section 4: Post-Application Monitoring (Days 1–7)

After applying changes, monitor:

1. **Ranking volatility**: Run V4 snapshot daily for 3 days post-application. Median rating should be stable (±10 points). Large swings indicate unexpected calibration interaction.

2. **Brier score tracking**: If pipeline produces Brier scores per run, confirm they are moving toward the projected values from the held-out validation.

3. **User-facing changes**: The top-10 rankings for U13G and U14G may shift due to smoother K-factor. Expected: teams with long, dominant histories move up slightly vs teams with recent short winning streaks.

4. **No-regression check**: Run the held-out Brier computation again on fresh data (any games in the week after application). Brier should be ≤ the proposed values from the staging validation.

### Section 5: Communication to SENTINEL

After successful application, post a queue item `Queue/completed-calibration-application-2026-05-XX.md` with:
- Date applied
- Before/after Brier scores (actual, from post-application monitoring)
- Any anomalies and how resolved
- Confirmation that rollback backup is retained until May 25

## What Done Looks Like

Produce the runbook at:
`02 - Tiger Tournaments/Projects/Rankings/Calibration Production Runbook — May 2026.md`

The runbook must be self-contained: Presten should be able to open it, execute each numbered step in order, and apply the calibration with confidence. No decisions left open. All commands are copy-paste ready.

## Success Criteria

- All 5 code artifacts (pre-verification queries, application commands, post-verification queries, spot check queries, rollback commands) are present and correct
- The before/after Brier scores from the held-out validation are populated (not `?`) — pull from `Reports/calibration-held-out-validation-2026-04-23.md`
- The rollback procedure is confirmed reversible without data loss
- A non-technical reader could follow Section 1–3 and apply the changes correctly
