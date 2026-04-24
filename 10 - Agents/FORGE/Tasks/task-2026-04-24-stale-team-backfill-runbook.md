---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: completed
priority: high
due: 2026-04-26
completed: 2026-04-24
deliverable: Infrastructure/Stale Team Backfill Runbook — April 2026.md
---

# Task: Stale Team Backfill Runbook — Staged Execution Plan for 540 Rate-Limit Casualties

## Context

The backoff fix (`task-2026-04-24-implement-option-b-backoff.md`) unblocks the backfill of 540 confirmed rate-limit-casualty teams from the April 21 audit. Once Presten deploys `fetchWithBackoff()`, the backfill must be executed — but it cannot be run naively:

- 540 teams at 600ms/request + exponential backoff = potentially 6–10+ hours of run time
- Running all 540 in one batch risks hitting rate limits again before the backoff logic stabilizes
- If the backfill produces data errors (stale scores written over correct ones, duplicate game records), we need to be able to stop and diagnose before it affects all 540 teams
- The backfill must not disrupt the active daily pipeline once that's running

The vault deliverable from `task-2026-04-24-implement-option-b-backoff.md` confirmed the backoff spec. This task: write the execution runbook Presten follows once the code is deployed.

## What You Need to Do

### Step 1: Define Batch Sizes and Sequencing

The 540 teams are not all equal. Some may have dozens of missing games; others may have just a handful. Recommend a staged execution approach:

**Stage 1 — Pilot (10 teams)**
- Run 10 teams from the stale list
- Purpose: confirm `fetchWithBackoff()` is working correctly and logs are clean
- Expected run time: ~10–15 minutes
- Pass criteria: 10 teams updated, no 429 errors after retry, no duplicate game records

**Stage 2 — Batch 1 (50 teams)**
- Run 50 teams
- Expected run time: ~45–60 minutes
- Pass criteria: similar to Stage 1; check daily summary log for backoff event count

**Stage 3 — Remaining (480 teams)**
- If Stages 1 and 2 pass: proceed with the remaining 480 teams, either as a single overnight run or split into 2 × 240-team batches
- Recommended: run overnight to avoid rate pressure during active hours

For each stage, document:
- The command to run (e.g., `node crawl-gotsport-api-parallel.js --team-ids=<list>`)
- How to generate the team ID list for that stage (first 10 from `Reports/stale-teams-2026-04-21.csv`, etc.)
- Where to find the run logs

### Step 2: Write Pre-Backfill Verification Queries

Before running any backfill, confirm the baseline state of the 540 teams in the DB. Write these SQL queries:

**A: Confirm team IDs are in the DB**
```sql
-- Check how many of the 540 stale team IDs exist in the teams table
SELECT COUNT(*) FROM teams WHERE id IN (<stale-team-id-list>);
```
Expected: 540 (or close). If significantly fewer, some IDs may have been merged or removed.

**B: Check current last_sync_date for the pilot batch**
```sql
SELECT id, name, last_sync_date, last_game_date
FROM teams
WHERE id IN (<pilot-10-ids>)
ORDER BY last_sync_date ASC;
```
Record these dates — this is the baseline to compare against after the backfill runs.

**C: Count total games in DB for the pilot batch (before)**
```sql
SELECT COUNT(*) FROM games 
WHERE home_team_id IN (<pilot-10-ids>) OR away_team_id IN (<pilot-10-ids>);
```
Record this count — the post-backfill count should be higher.

### Step 3: Write Post-Stage Verification Queries

After each stage, Presten runs these to confirm the backfill worked correctly:

**A: Confirm last_sync_date was updated**
```sql
SELECT id, name, last_sync_date 
FROM teams 
WHERE id IN (<stage-ids>)
ORDER BY last_sync_date DESC;
```
Expected: all teams now show a recent `last_sync_date` (within the last hour).

**B: Confirm game count increased**
```sql
SELECT COUNT(*) FROM games 
WHERE home_team_id IN (<stage-ids>) OR away_team_id IN (<stage-ids>);
```
Compare to the before-count from Step 2C. An increase confirms new games were ingested.

**C: Check for duplicate game records**
```sql
SELECT home_team_id, away_team_id, game_date, COUNT(*) as duplicate_count
FROM games
WHERE (home_team_id IN (<stage-ids>) OR away_team_id IN (<stage-ids>))
GROUP BY home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1
LIMIT 20;
```
Expected: 0 rows. Any duplicates = data integrity issue, pause backfill.

### Step 4: Define Go/No-Go Criteria Between Stages

After Stage 1 (10 teams):
- ✅ Proceed if: last_sync_date updated for all 10, no duplicate games, log shows 0 post-max-retry failures
- ⚠️ Investigate if: 1–2 teams still show old last_sync_date (API timeout, not a backoff failure — retry individually)
- 🚨 Stop if: >3 teams failed, any duplicate games found, or logs show runaway 429s not resolved by backoff

After Stage 2 (50 teams):
- ✅ Proceed to Stage 3 if: >95% of teams updated, 0 duplicate games, backoff events logged but resolving (no max-retry hits)
- 🚨 Stop if: >5% max-retry failures, duplicate games found, or daily summary log shows escalating 429 rate

### Step 5: Rollback Plan

If the backfill produces bad data:

**Scenario A: Duplicate game records**
Write the cleanup query:
```sql
-- Delete duplicate games keeping the most recent record
DELETE FROM games WHERE id IN (
  SELECT id FROM (
    SELECT id,
      ROW_NUMBER() OVER (PARTITION BY home_team_id, away_team_id, game_date ORDER BY created_at DESC) as rn
    FROM games
    WHERE home_team_id IN (<stage-ids>) OR away_team_id IN (<stage-ids>)
  ) duplicates
  WHERE rn > 1
);
```
Note: this is a template — adapt column names to the actual schema.

**Scenario B: Wrong game data written (score errors)**
If GotSport API returned incorrect data during a 429 retry:
```sql
-- Identify games added in the last 24h for the backfill teams
SELECT id, home_team_id, away_team_id, home_score, away_score, game_date, created_at
FROM games
WHERE created_at >= NOW() - INTERVAL '24 hours'
  AND (home_team_id IN (<stage-ids>) OR away_team_id IN (<stage-ids>))
ORDER BY created_at DESC;
```
Spot-check 5 games. If scores look wrong, delete newly-added games and re-examine the API response logs.

**Scenario C: Rankings corrupted post-backfill**
If new games shift rankings unexpectedly:
- Do NOT re-run compute immediately — analyze first
- Check: did the backfill teams gain a disproportionate number of wins or losses?
- If data looks correct: re-run `compute-rankings.js` for the affected age groups only

### Step 6: Scheduling Recommendation

Document the recommended timing:

1. **Deploy backoff fix:** As early as Presten can (first priority, April 24)
2. **Run pilot (10 teams):** Same day as backoff deployment, during daylight hours — confirms the fix works
3. **Run Stage 2 (50 teams):** Day 2, during daylight — monitor logs actively
4. **Run Stage 3 (480 teams):** Overnight (e.g., 11 PM start via manual run or overnight script), after Stage 2 passes

Note: do NOT run the Stage 3 backfill on the same night as the first daily pipeline run. Sequence: backoff → Stage 1 → Stage 2 → daily pipeline first run → Stage 3.

## Deliverable

Write the runbook to:
`02 - Tiger Tournaments/Projects/Infrastructure/Stale Team Backfill Runbook — April 2026.md`

Structure as a linear checklist. Presten should be able to follow it top-to-bottom, run each query, interpret the output, and make the go/no-go call at each stage.

## Definition of Done

- All three stages defined with exact commands and expected output
- Pre-backfill verification queries complete (3 queries)
- Post-stage verification queries complete (3 queries)
- Go/no-go decision criteria are numeric and explicit (not "looks good")
- Rollback plan covers 3 failure scenarios
- Scheduling recommendation is explicit (which day, what time)
- Summary in your briefing: runbook ready; pilot can run same day as backoff deployment

## References

- `Reports/stale-teams-2026-04-21.csv` — the 540 rate-limit casualty team IDs
- `task-2026-04-24-implement-option-b-backoff.md` — backoff spec (code spec complete, deploy pending)
- `Reports/backoff-implementation-2026-04-24.md` — FORGE's backoff implementation note
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — games + teams table schema
