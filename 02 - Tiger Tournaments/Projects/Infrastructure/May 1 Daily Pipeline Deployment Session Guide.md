---
title: May 1 Daily Pipeline Deployment Session Guide
tags: [infrastructure, pipeline, deployment, daily-pipeline, session-guide, evo-draw]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: ready — use on May 1, 2026
task: task-2026-04-25-may1-pipeline-deployment-session-plan
companion: "Product & Planning/Daily Pipeline Cadence Scope.md"
---

# May 1 Daily Pipeline Deployment Session Guide

---

## Section 1: Session Overview

| Item | Value |
|------|-------|
| **Session name** | May 1 Daily Pipeline Deployment |
| **Hard deadline** | May 1, 2026 |
| **Estimated time** | 2.5–3.5 hours (script writing + cron config + validation + first manual test run) |
| **Prerequisites** | April 29 pipeline health check is GREEN; no active backfill runs; server access confirmed |
| **Deliverables** | `get-active-team-ids.sh` and `run-daily.sh` live on server; cron configured and tested; first manual run passes |
| **Fallback** | `run-overnight.sh` (existing full crawl) remains untouched and functional as rollback at all times |

---

## Section 2: Pre-Session Checklist

**Complete every item before writing a single line of code. If any item cannot be checked: stop and flag to SENTINEL.**

- [ ] FORGE's April 29 briefing shows **GREEN** pipeline health check status (all checks from `Infrastructure/April 29 Pipeline Health Check — Post-Fix Protocol.md` passed)
- [ ] No active backfill runs in progress: `ps aux | grep crawl-gotsport | grep -v grep` returns nothing
- [ ] No active pipeline runs: `ps aux | grep run-pipeline | grep -v grep` returns nothing
- [ ] Server access confirmed (can SSH to the production server)
- [ ] `logs/` directory exists in the project root (or create it: `mkdir -p logs`)
- [ ] `Product & Planning/Daily Pipeline Cadence Scope.md` Section 6 open in a second tab
- [ ] `run-overnight.sh` is confirmed functional and untouched (this is your fallback — do NOT modify it)

---

## Section 3: Script Creation — Step by Step

### Script 1: `get-active-team-ids.sh`

**File path on server:** `[project-root]/get-active-team-ids.sh`  
**Purpose:** Queries the DB for active GotSport team IDs (played in last 90 days or scheduled in next 30 days); outputs comma-separated IDs to stdout for use as `GSAPI_TEAM_IDS`.

**Complete script — copy-paste verbatim:**

```bash
#!/bin/bash
# get-active-team-ids.sh
# Outputs comma-separated active GotSport team IDs to stdout

psql youth_soccer -t -A -c "
SELECT string_agg(DISTINCT gotsport_id, ',')
FROM (
  SELECT unnest(ARRAY[home_team_id, away_team_id]) AS gotsport_id
  FROM games
  WHERE source IN ('gotsport_api','gotsport')
    AND is_hidden = false
    AND (
      (game_status = 'completed' AND match_date > NOW() - INTERVAL '90 days')
      OR (game_status = 'scheduled' AND match_date BETWEEN NOW() AND NOW() + INTERVAL '30 days')
    )
    AND home_team_id IS NOT NULL
    AND away_team_id IS NOT NULL
) t
"
```

**Make executable after writing:**
```bash
chmod +x get-active-team-ids.sh
```

**Verify after writing:**
```bash
bash get-active-team-ids.sh 2>&1 | head -c 200
```

**Expected output:** A single line of comma-separated numeric IDs (e.g., `12345,23456,34567,...`). No SQL headers, no blank lines.

**Expected ID count:** Run `bash get-active-team-ids.sh | tr ',' '\n' | wc -l` — should output a number between 15,000 and 40,000 during spring peak season. If below 5,000: the query is not returning results — check the DB connection and column names before proceeding. If above 100,000: the filter is not working; do NOT proceed — debug the WHERE clause.

---

### Script 2: `run-daily.sh`

**File path on server:** `[project-root]/run-daily.sh`  
**Purpose:** Daily pipeline — queries active team IDs, crawls only those teams via GotSport API, then runs the full normalization + ranking pipeline (Steps 1–9).

**Complete script — copy-paste verbatim:**

```bash
#!/bin/bash
# run-daily.sh
# Daily pipeline: active teams only + full normalization + ranking recompute
# Replaces run-overnight.sh for nightly execution (Mon-Sat)
# run-overnight.sh preserved as fallback (weekly full crawl, Sunday)

set -e

LOG_FILE="logs/daily-pipeline-$(date +%Y-%m-%d).log"
mkdir -p logs
exec >> "$LOG_FILE" 2>&1

log() {
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

trap 'log "DAILY PIPELINE FAILED (exit code: $?)"' ERR

log "DAILY PIPELINE STARTED"

# Step 0a: Get active team IDs
log "Fetching active team IDs..."
ACTIVE_IDS=$(bash get-active-team-ids.sh)
ID_COUNT=$(echo "$ACTIVE_IDS" | tr ',' '\n' | wc -l | tr -d ' ')
log "STEP 1: Crawl started (team IDs: $ID_COUNT)"

# Step 0b: Crawl active teams only
GSAPI_TEAM_IDS="$ACTIVE_IDS" GSAPI_SKIP_RANKINGS=true \
  node crawl-gotsport-api-parallel.js
log "STEP 1: Crawl completed (games fetched: see SYNC_RUN_SUMMARY above, errors: see DLQ)"

# Steps 1–9: Full pipeline (normalization, dedup, rankings)
log "STEP 2: Rankings compute started"
bash run-pipeline.sh
log "STEP 2: Rankings compute completed (teams ranked: see compute-rankings output above)"

log "DAILY PIPELINE COMPLETED SUCCESSFULLY"
```

**Make executable after writing:**
```bash
chmod +x run-daily.sh
```

**Verify script syntax before running:**
```bash
bash -n run-daily.sh && echo "Syntax OK"
```

Expected: `Syntax OK`. If syntax errors appear: fix before proceeding.

---

## Section 4: Cron Configuration

**First: confirm no existing entries for these scripts (avoid duplicates):**
```bash
crontab -l | grep -E "run-daily|get-active-team"
```
If any existing entries appear: remove them with `crontab -e` before adding the new entries below.

**Exact cron entries to add** (run `crontab -e` to edit):

```crontab
# Evo Draw — Daily Pipeline (active teams only, Mon–Sat, 2 AM)
0 2 * * 1-6 cd /path/to/evo-draw && bash run-daily.sh

# Evo Draw — Weekly full crawl (all 107K IDs, Sunday 11 PM)
0 23 * * 0 cd /path/to/evo-draw && bash run-overnight.sh

# Evo Draw — Health check (8:30 AM daily, after pipeline expected to finish)
30 8 * * * bash /path/to/evo-draw/check-pipeline-health.sh
```

> [!warning] Replace `/path/to/evo-draw`
> Substitute the actual absolute path to the project root on the server. Using a relative path in cron will fail because cron does not inherit the working directory.

**Verify cron was saved:**
```bash
crontab -l | grep run-daily
```
Expected output: the Mon–Sat cron entry is visible. If empty: cron was not saved — re-run `crontab -e` and try again.

---

## Section 5: Deployment Validation

Run these checks from `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` Section 3 before relying on the cron.

**Trigger a manual test run (do NOT wait for cron):**
```bash
bash run-daily.sh 2>&1 | tee /tmp/daily-pipeline-test.log
```

Wait for completion. Expected duration: 2.5–5 hours during spring peak season (active team crawl) + 1–2 hours pipeline = 3.5–7 hours total. During off-peak or on first run with a smaller active ID set, may complete faster.

**After run completes, check each item:**

- [ ] **Last log line is success marker:**
  ```bash
  tail -1 logs/daily-pipeline-$(date +%Y-%m-%d).log
  ```
  Expected exact string: `[YYYY-MM-DD HH:MM:SS] DAILY PIPELINE COMPLETED SUCCESSFULLY`

- [ ] **Cron entry confirmed:**
  ```bash
  crontab -l | grep run-daily
  ```
  Expected: Mon–Sat entry visible.

- [ ] **Log file exists at expected path:**
  ```bash
  ls -lh logs/daily-pipeline-$(date +%Y-%m-%d).log
  ```
  Expected: file exists with non-zero size.

- [ ] **Active team ID count in valid range:**
  Check the STEP 1 log line:
  ```bash
  grep "STEP 1: Crawl started" logs/daily-pipeline-$(date +%Y-%m-%d).log
  ```
  Expected: `team IDs: N` where N is between 15,000 and 40,000.  
  If N > 100,000: active filter not working — remove cron immediately, debug `get-active-team-ids.sh`.

- [ ] **Rankings updated after run:**
  ```sql
  SELECT MAX(updated_at) AS last_rankings_update FROM rankings;
  ```
  Expected: timestamp is within the last 1–2 hours.

**Decision framework:**

| Result | Action |
|--------|--------|
| **GREEN** — all 5 checks pass | Deployment complete. Cron is live. Proceed to Section 6. |
| **YELLOW** — one check fails (e.g., log path mismatch) | Do not kill cron. Investigate the specific failure. Fix the log redirect or path issue and re-verify. |
| **RED** — last log line is `DAILY PIPELINE FAILED` or active IDs > 100K | Remove cron entries immediately. File error at `Reports/deployment-error-daily-pipeline-[date].md`. Post queue item to FORGE before retrying. Do not retry without FORGE's corrected spec. |

---

## Section 6: Post-Deploy Actions

After GREEN confirmation:

1. **Note in FORGE's next briefing:**
   > "May 1 daily pipeline deployment complete. Cron live: `0 2 * * 1-6`. First automated run scheduled for [next weekday] at 2:00 AM. Manual test run: GREEN — active teams: [N], log line confirmed, rankings updated."

2. **Mark task completed:**
   Update `10 - Agents/FORGE/Tasks/task-2026-04-24-implement-daily-pipeline.md` status from `in_progress` to `completed`.

3. **Watch the first cron-triggered run:**
   At 2:00 AM on the next weekday, check the log at `logs/daily-pipeline-[tomorrow's date].log`:
   ```bash
   tail -5 logs/daily-pipeline-$(date -v+1d '+%Y-%m-%d' 2>/dev/null || date -d 'tomorrow' '+%Y-%m-%d').log
   ```
   Confirm `DAILY PIPELINE COMPLETED SUCCESSFULLY` appears. If not: check the health check output from the 8:30 AM cron.

4. **File first-run results in next FORGE briefing:**
   Note actual vs. expected: active team count, crawl duration, game count delta since last run, rankings updated_at timestamp. These become the baseline for future health checks.

5. **Stage 3 backfill sequencing:**
   Per `Infrastructure/Stale Team Backfill Runbook — April 2026.md` Section "Stage 3 Sequencing Constraint": Stage 3 (480 teams) must NOT run on the same night as the first daily pipeline run. Confirm first daily cron run completes successfully before scheduling Stage 3.

---

## Rollback (if cron causes issues)

If the daily cron causes unexpected problems (ranking regressions, excessive API load, data issues):

```bash
# 1. Disable daily cron — comment out the Mon-Sat line
crontab -e
# Change: 0 2 * * 1-6 cd /path/to/evo-draw && bash run-daily.sh
# To:    # 0 2 * * 1-6 cd /path/to/evo-draw && bash run-daily.sh

# 2. Re-enable overnight batch manually if rankings are stale
bash run-overnight.sh &

# Rankings will be refreshed with full 107K ID crawl within 12 hours
```

Rollback triggers: rankings for known active teams not updating day-over-day; active ID query returning <5,000 IDs; pipeline log shows completion without errors but `MAX(updated_at)` in rankings is >36 hours old.

---

## References

- `Product & Planning/Daily Pipeline Cadence Scope.md` — Option A spec; complete script text in Section 6
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — Section 3: daily pipeline validation checks
- `Infrastructure/Daily Pipeline Log Format Spec.md` — log format, success/failure markers, verification commands
- `Infrastructure/April 29 Pipeline Health Check — Post-Fix Protocol.md` — the GREEN pre-condition for this session
- `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — Stage 3 sequencing constraint
- `Product & Planning/April 28 Escalation Reference Card.md` — format reference for this document
- `task-2026-04-24-implement-daily-pipeline.md` — the task this session closes

*FORGE — 2026-04-25*
