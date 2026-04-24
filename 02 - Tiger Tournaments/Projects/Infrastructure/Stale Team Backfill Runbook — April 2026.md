---
title: Stale Team Backfill Runbook — April 2026
tags: [infrastructure, runbook, backfill, gotsport, stale-teams, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: FORGE
status: ready-for-execution
prerequisite: fetchWithBackoff() deployed in crawl-gotsport-api-parallel.js
---

# Stale Team Backfill Runbook — April 2026

**Prepared by:** FORGE  
**Date:** 2026-04-24  
**Task:** `task-2026-04-24-stale-team-backfill-runbook.md`  
**Prerequisite:** The exponential backoff fix (`fetchWithBackoff()`) must be deployed and confirmed before running any stage of this runbook. Do not proceed without it.  
**Source data:** `Reports/stale-teams-2026-04-21.csv` — 540 confirmed rate-limit-casualty team IDs

---

## Overview

The 847 stale teams from the April 21 audit break down as:
- ~540 confirmed rate-limit casualties (these are the targets of this runbook)
- ~180 de-prioritized tournament teams (archive workflow handles these separately)
- ~127 unexplained (investigated in `Reports/stale-teams-unexplained-2026-04-22.md`)

This runbook covers the **540 rate-limit casualties only**. Execute in three staged batches.

---

## Pre-Backfill: Confirm Prerequisites

Before running any stage:

- [ ] `fetchWithBackoff()` is confirmed deployed in `crawl-gotsport-api-parallel.js`
- [ ] `Reports/backoff-implementation-2026-04-24.md` confirms the deployment
- [ ] `Logs/dlq/` directory exists and is writable
- [ ] The 540-team ID list is available from `Reports/stale-teams-2026-04-21.csv` (rate-limit casualty subset)
- [ ] No other crawl job is currently running (check process list: `ps aux | grep crawl`)
- [ ] Daily pipeline cron is NOT active yet (do not run alongside the daily pipeline until Stage 3 is complete)

---

## Step 1 — Pre-Backfill Verification Queries

Run these before any backfill to establish the baseline. Record all outputs.

### Query A: Confirm the 540 team IDs exist in the DB

```sql
-- Replace <stale-team-id-list> with the actual comma-separated IDs from the CSV
SELECT COUNT(*) FROM teams WHERE id IN (<stale-team-id-list>);
```

**Expected:** 540 (or very close). If significantly fewer (e.g., <500), some IDs may have been merged or archived — investigate before proceeding.

### Query B: Check current last_sync_date for the pilot batch (10 IDs)

```sql
-- Use the first 10 IDs from the CSV as the pilot batch
SELECT id, name, last_sync_date, last_game_date
FROM teams
WHERE id IN (<pilot-10-ids>)
ORDER BY last_sync_date ASC;
```

**Record these dates.** This is your pre-backfill baseline. After Stage 1, all 10 should show a recent `last_sync_date`.

### Query C: Count total games for the pilot batch (before)

```sql
SELECT COUNT(*) FROM games 
WHERE home_team_id IN (<pilot-10-ids>) OR away_team_id IN (<pilot-10-ids>);
```

**Record this count.** The post-Stage 1 count must be ≥ this number (never lower).

---

## Stage 1 — Pilot (10 Teams)

**Purpose:** Confirm `fetchWithBackoff()` works correctly in production with a small, controlled run.  
**Team IDs:** First 10 rows from `Reports/stale-teams-2026-04-21.csv` (rate-limit casualty subset).  
**Expected run time:** 10–15 minutes.  
**Run during:** Daylight hours (not overnight). Monitor logs actively.

### Command

```bash
# Extract first 10 stale team IDs from the CSV (adjust column name as needed)
PILOT_IDS=$(head -11 Reports/stale-teams-2026-04-21.csv | tail -10 | cut -d',' -f1 | tr '\n' ',' | sed 's/,$//')

GSAPI_TEAM_IDS="$PILOT_IDS" node crawl-gotsport-api-parallel.js
```

Alternatively, if the scraper accepts a file:

```bash
head -11 Reports/stale-teams-2026-04-21.csv | tail -10 | cut -d',' -f1 > /tmp/pilot-ids.txt
GSAPI_TEAM_IDS_FILE=/tmp/pilot-ids.txt node crawl-gotsport-api-parallel.js
```

### Monitor Logs

During the run, tail the log for backoff events and errors:

```bash
tail -f Logs/gotsport-sync/$(date +%Y-%m-%d)-*.log | grep -E "BACKOFF|RETRY|429|ERROR|SYNC_RUN_SUMMARY"
```

### Post-Stage 1 Verification Queries

#### A: Confirm last_sync_date was updated

```sql
SELECT id, name, last_sync_date 
FROM teams 
WHERE id IN (<pilot-10-ids>)
ORDER BY last_sync_date DESC;
```

**Expected:** All 10 teams show `last_sync_date` within the last 1 hour.

#### B: Confirm game count increased

```sql
SELECT COUNT(*) FROM games 
WHERE home_team_id IN (<pilot-10-ids>) OR away_team_id IN (<pilot-10-ids>);
```

**Compare to the pre-run count from Step 1C.** Should be ≥ pre-run count (new games ingested).

#### C: Check for duplicate game records

```sql
SELECT home_team_id, away_team_id, match_date, COUNT(*) as duplicate_count
FROM games
WHERE (home_team_id IN (<pilot-10-ids>) OR away_team_id IN (<pilot-10-ids>))
GROUP BY home_team_id, away_team_id, match_date
HAVING COUNT(*) > 1
LIMIT 20;
```

**Expected: 0 rows.** Any duplicates = stop immediately and investigate.

### Stage 1 Go/No-Go Criteria

- ✅ **Proceed to Stage 2 if:**
  - `last_sync_date` updated for all 10 teams
  - 0 duplicate game records
  - `SYNC_RUN_SUMMARY` log line shows 0 post-max-retry failures
  - No `PERSISTENT_EMPTY_PAYLOAD` errors
- ⚠️ **Investigate before proceeding if:**
  - 1–2 teams still show old `last_sync_date` — may be an API timeout for specific teams, not a backoff failure. Re-run just those IDs individually to confirm.
- 🚨 **Stop if:**
  - >3 teams failed (exceeded max retries)
  - Any duplicate game records found
  - Logs show runaway 429 errors not resolving via backoff
  - A 403 response is logged (auth wall — stop the entire operation, alert Presten)

---

## Stage 2 — Batch 1 (50 Teams)

**Purpose:** Verify the backoff logic holds at moderate scale.  
**Team IDs:** Next 50 rows from the stale CSV (rows 11–60).  
**Expected run time:** 45–60 minutes.  
**Run during:** Daylight hours. Monitor the first 10 minutes closely.

### Command

```bash
# Extract IDs 11–60
BATCH1_IDS=$(sed -n '12,61p' Reports/stale-teams-2026-04-21.csv | cut -d',' -f1 | tr '\n' ',' | sed 's/,$//')
GSAPI_TEAM_IDS="$BATCH1_IDS" node crawl-gotsport-api-parallel.js
```

### Post-Stage 2 Verification Queries

Same three queries as Stage 1, substituting `<pilot-10-ids>` with `<batch1-50-ids>`.

Also check the **daily summary log** for backoff event counts:

```bash
grep "SYNC_RUN_SUMMARY" Logs/gotsport-sync/$(date +%Y-%m-%d)-*.log | tail -1
```

Review `teams_dlq` field in the summary. If >5% of the 50 teams ended in the DLQ (>2-3 teams), investigate before Stage 3.

### Stage 2 Go/No-Go Criteria

- ✅ **Proceed to Stage 3 if:**
  - >95% of teams updated (≥48 of 50 show recent `last_sync_date`)
  - 0 duplicate game records
  - `teams_dlq` ≤ 2 in the SYNC_RUN_SUMMARY
  - Backoff events logged but resolving (no team reaching max retries repeatedly)
- 🚨 **Stop if:**
  - >5% max-retry failures (>2-3 teams in DLQ that don't resolve on retry)
  - Any duplicate game records
  - Daily summary log shows escalating 429 rate (backoff wait times trending toward cap)
  - 403 response logged at any point

---

## Stage 3 — Remaining (480 Teams)

> [!warning] SEQUENCING CONSTRAINT — READ BEFORE RUNNING STAGE 3
> **Do NOT run Stage 3 on the same night as the first `run-daily.sh` pipeline run.**
>
> Stage 3 identifies stale teams using a snapshot of the `last_game_date` column. If the daily pipeline runs concurrently or immediately after Stage 3 begins, new game records may update `last_game_date` mid-scan, producing false negatives (teams incorrectly excluded from the backfill batch).
>
> **Correct sequence:** Complete the daily pipeline first. Confirm it succeeded (check `Logs/gotsport-sync/` for a `SYNC_RUN_SUMMARY` line and confirm `DAILY PIPELINE COMPLETED SUCCESSFULLY` in the daily log). Then run Stage 3 the following day.
>
> If you are unsure whether the daily pipeline has run for the first time yet, check: `logs/daily-pipeline-$(date +%Y-%m-%d).log` for the success marker, or use the verification command in `Infrastructure/Daily Pipeline Log Format Spec.md`.

**Purpose:** Complete the full backfill of all 540 rate-limit casualties.  
**Team IDs:** All remaining rows from the stale CSV (rows 61–540).  
**Expected run time:** 6–10 hours at 600ms/request with backoff events.  
**Run during:** Overnight (recommended start time: 11 PM).  
**IMPORTANT:** Do NOT run Stage 3 on the same night as the first daily pipeline run. Sequence: backoff fix → Stage 1 → Stage 2 → daily pipeline first run → Stage 3.

### Command

```bash
# Extract IDs 61 onward (480 teams)
REMAINING_IDS=$(tail -n +62 Reports/stale-teams-2026-04-21.csv | cut -d',' -f1 | tr '\n' ',' | sed 's/,$//')
GSAPI_TEAM_IDS="$REMAINING_IDS" node crawl-gotsport-api-parallel.js
```

Or split into two nights (240 each) if you prefer shorter individual runs:

```bash
# Night 1: IDs 61–300
NIGHT1_IDS=$(sed -n '62,301p' Reports/stale-teams-2026-04-21.csv | cut -d',' -f1 | tr '\n' ',' | sed 's/,$//')
GSAPI_TEAM_IDS="$NIGHT1_IDS" node crawl-gotsport-api-parallel.js

# Night 2: IDs 301–540
NIGHT2_IDS=$(sed -n '302,541p' Reports/stale-teams-2026-04-21.csv | cut -d',' -f1 | tr '\n' ',' | sed 's/,$//')
GSAPI_TEAM_IDS="$NIGHT2_IDS" node crawl-gotsport-api-parallel.js
```

### Post-Stage 3 Verification

```sql
-- Check overall completion: how many of the 540 now have recent last_sync_date?
SELECT 
  COUNT(*) FILTER (WHERE last_sync_date > NOW() - INTERVAL '48 hours') as recently_synced,
  COUNT(*) FILTER (WHERE last_sync_date <= NOW() - INTERVAL '48 hours' OR last_sync_date IS NULL) as still_stale,
  COUNT(*) as total
FROM teams
WHERE id IN (<all-540-stale-ids>);
```

**Expected:** recently_synced ≥ 510 (>95%). Remaining stale IDs should be investigated individually (DLQ file will have details).

---

## Rollback Plan

### Scenario A: Duplicate Game Records Found

Stop the backfill immediately. Run the cleanup query:

```sql
-- Delete duplicate games, keeping the most recently created record
DELETE FROM games WHERE id IN (
  SELECT id FROM (
    SELECT id,
      ROW_NUMBER() OVER (
        PARTITION BY home_team_id, away_team_id, match_date 
        ORDER BY created_at DESC
      ) as rn
    FROM games
    WHERE home_team_id IN (<stage-ids>) OR away_team_id IN (<stage-ids>)
  ) ranked
  WHERE rn > 1
);
```

Adapt column names to the actual schema (`match_date` may be `game_date` — check `Database Schema.md`). After cleanup, re-run the duplicate check query (Stage 1 Query C) to confirm 0 rows.

### Scenario B: Wrong Game Data Written (Score Errors)

If GotSport returned incorrect data during a 429-retry window, identify and spot-check:

```sql
-- Games added in the last 24h for the backfill teams
SELECT id, home_team_id, away_team_id, home_score, away_score, match_date, created_at
FROM games
WHERE created_at >= NOW() - INTERVAL '24 hours'
  AND (home_team_id IN (<stage-ids>) OR away_team_id IN (<stage-ids>))
ORDER BY created_at DESC
LIMIT 50;
```

Spot-check 5 games against GotSport website directly. If scores look wrong:
1. Note the `created_at` timestamp range
2. Delete newly-added games in that window
3. Re-examine the API response logs for error patterns
4. Re-run only the affected team IDs after investigating

### Scenario C: Rankings Corrupted Post-Backfill

If new games shift rankings unexpectedly after re-running `compute-rankings.js`:
1. **Do NOT re-run compute immediately** — analyze first
2. Check: did the backfill teams gain a disproportionate number of wins? Losses with no corresponding opponents in the DB?
3. If data looks correct (scores reasonable, opponents exist): re-run ranking computation for the affected age groups only
4. If data looks suspicious: roll back the affected games using Scenario B approach

---

## Scheduling Recommendation

| Action | Timing | Notes |
|---|---|---|
| Deploy backoff fix | ASAP — April 24 | First priority |
| **Stage 1 (10 teams)** | **Same day as backoff deployment, during daylight** | Confirms the fix works; 15 min run |
| **Stage 2 (50 teams)** | **Day 2 (April 25), during daylight** | Monitor logs actively; 45–60 min run |
| First daily pipeline run | Day 3 (April 26), 2 AM via cron or manual | Confirm pipeline stable before Stage 3 |
| **Stage 3 (480 teams)** | **April 26–27, overnight (11 PM start)** | After daily pipeline first run passes |

**Critical sequencing rule:** Stage 3 must NOT run on the same night as the first daily pipeline run. A failed overnight with both jobs competing for GotSport API calls would be very difficult to diagnose.

---

## Log Locations

| What | Path |
|---|---|
| Sync run logs | `Logs/gotsport-sync/YYYY-MM-DD-*.log` |
| Dead Letter Queue | `Logs/dlq/dlq-{runId}.jsonl` |
| SYNC_RUN_SUMMARY | Last line of each sync log — grep for `SYNC_RUN_SUMMARY` |

---

## References

- `Reports/stale-teams-2026-04-21.csv` — the 540 rate-limit casualty team IDs
- `Reports/backoff-implementation-2026-04-24.md` — backoff spec and deployment note
- `task-2026-04-24-implement-option-b-backoff.md` — backoff task (vault deliverable complete, deploy pending)
- `Infrastructure/Database Schema.md` — games + teams table schema (column name reference)
- `task-2026-04-24-stale-team-backfill-runbook.md` — assigned task

---

*FORGE — 2026-04-24*
