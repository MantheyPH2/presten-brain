---
title: May 1 Deployment — Day-Of Runbook
tags: [infrastructure, pipeline, deployment, runbook, daily-pipeline, evo-draw]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: ready — use on May 1, 2026 after Pre-Authorization Checklist is complete
companion: "Infrastructure/May 1 Daily Pipeline Deployment Session Guide.md"
---

# May 1 Deployment — Day-Of Runbook

> This runbook covers the deployment session itself: from "Pre-Auth Checklist complete" to "first run launched and confirmed healthy." Follow it linearly, top to bottom.

---

## Pre-Runbook Gate

Before beginning Step 1, confirm all 5 sections of `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` are complete:

- Section 1 (April 28 DB Changes): all 4 boxes checked ✅
- Section 2 (April 29 Pipeline Health Check): GREEN confirmed ✅
- Section 3 (Scripts in place): `run-daily.sh` and `get-active-team-ids.sh` confirmed ✅
- Section 4 (Stage 1 status): confirmed (PASS / PENDING / FAIL) ✅
- Section 5 (SENTINEL Authorization): signed ✅

**If any Section 1–3 item is unchecked: do not proceed. Contact SENTINEL.**

---

## Step 1: Launch `run-daily.sh` — First Manual Test Run

> The first run is a manual test, not a cron-triggered run. This confirms the pipeline works end-to-end before handing off to the scheduled cron.

**Command:**

```bash
bash run-daily.sh 2>&1 | tee /tmp/daily-pipeline-test.log
```

This command runs `run-daily.sh` in the foreground and sends output to both the terminal and a temp log file.

**What you should see in the first 60 seconds:**

```
[2026-05-01 HH:MM:SS] DAILY PIPELINE STARTED
[2026-05-01 HH:MM:SS] Fetching active team IDs...
[2026-05-01 HH:MM:SS] STEP 1: Crawl started (team IDs: N)
```

Where `N` is between **15,000 and 40,000** (spring peak season).

**Startup failure signals (act immediately, do not wait):**

| Signal | Meaning | Action |
|--------|---------|--------|
| `No such file or directory: get-active-team-ids.sh` | Missing script | Check file exists: `ls -lh get-active-team-ids.sh` |
| `psql: error: connection to server failed` | DB connection failed | Confirm DB is accessible before proceeding |
| `N > 100,000` in team IDs line | Active filter not working | Kill run with Ctrl+C. Debug `get-active-team-ids.sh` WHERE clause. |
| `N < 1,000` in team IDs line | Query returned too few IDs | Kill run with Ctrl+C. Run `bash get-active-team-ids.sh \| tr ',' '\n' \| wc -l` to debug. |
| `SyntaxError` or `Cannot find module` | Script error | Kill run. Fix the error per the error message. |

**If startup looks healthy (N between 15K–40K):** Let the run proceed. Do not interrupt unless a startup failure signal appears.

---

## Step 2: Monitor Initial Crawl Progress

**Expected run duration:**

| Condition | Estimated duration |
|-----------|--------------------|
| Spring peak season, ~20,000 active IDs | 2.5–5 hours (crawl) + 1–2 hours (pipeline) = **3.5–7 hours total** |
| Off-peak, <10,000 active IDs | 1–2 hours (crawl) + 1 hour (pipeline) = **2–3 hours total** |
| First run with restricted ID set (testing) | ~1 hour |

**Something is wrong threshold:** If no output has appeared in the terminal for **30 minutes** after the `STEP 1: Crawl started` line, the crawl may be hung.

**How to confirm the crawl is actively running:**

In a separate terminal:

```bash
tail -f logs/daily-pipeline-$(date +%Y-%m-%d).log
```

You should see new lines appearing every few minutes with game sync updates. Example healthy output pattern:

```
[HH:MM:SS] SYNC_RUN_SUMMARY: games_fetched=12 errors=0 org_id=12345
[HH:MM:SS] SYNC_RUN_SUMMARY: games_fetched=8 errors=0 org_id=12346
```

**If no new log lines for 30+ minutes:**

```bash
ps aux | grep crawl-gotsport | grep -v grep
```

If the process is listed: it's still running but slow (normal for large orgs). If nothing is listed: the crawl exited — check the log for `DAILY PIPELINE FAILED`.

**Spot-check game ingestion is happening** (run mid-crawl):

```sql
SELECT MAX(created_at) AS last_inserted, COUNT(*) AS games_today
FROM games
WHERE created_at > NOW() - INTERVAL '3 hours'
  AND source IN ('gotsport_api', 'gotsport')
  AND is_hidden = false;
```

Expected during an active crawl: `games_today` > 0 and `last_inserted` is recent.

---

## Step 3: First Run Completion Check

After the run completes (terminal returns to prompt, or you see the success marker in the log):

**Confirm the run completed successfully:**

```bash
tail -1 logs/daily-pipeline-$(date +%Y-%m-%d).log
```

**Expected exact string:**

```
[2026-05-01 HH:MM:SS] DAILY PIPELINE COMPLETED SUCCESSFULLY
```

**If instead you see:**

```
[2026-05-01 HH:MM:SS] DAILY PIPELINE FAILED (exit code: N)
```

**→ This is RED. Do not proceed. See Step 4 RED action.**

**Quick health query — active team count ingested this run:**

```bash
grep "STEP 1: Crawl started" logs/daily-pipeline-$(date +%Y-%m-%d).log
```

Record the team ID count. This is your baseline for future runs.

**Rankings updated check:**

```sql
SELECT MAX(updated_at) AS last_rankings_update FROM rankings;
```

Expected: timestamp within the last 1–2 hours.

**Pass threshold:** Last log line = `DAILY PIPELINE COMPLETED SUCCESSFULLY`, team IDs 15,000–40,000, rankings updated within last 2 hours.

**Fail threshold:** Any of — `DAILY PIPELINE FAILED` in log, team IDs > 100,000, rankings timestamp predates this run.

> [!note] Expected game count on first run
> Stage 1 Results Report will provide the expected count from the TX test cohort. If Stage 1 is still PENDING at the time of this run, use the Morning-After Runbook's thresholds: any non-zero `games_today` count from the spot-check query confirms ingestion is working. A definitive baseline is established after the first full overnight cron run.

---

## Step 4: Post-Run Decision

| Result | Action |
|--------|--------|
| **GREEN** — completion marker confirmed, team IDs 15K–40K, rankings updated | Proceed to Step 5. Deployment is complete. |
| **YELLOW** — completion marker confirmed, but rankings updated_at is >2 hours or minor log warnings | Do NOT kill cron. Check `grep WARN logs/daily-pipeline-$(date +%Y-%m-%d).log`. If only warnings (not errors), pipeline is healthy. Note in FORGE briefing. Stage 3 authorization deferred until YELLOW is explained. |
| **RED** — `DAILY PIPELINE FAILED` in log, OR team IDs >100,000, OR completion marker absent | Execute RED protocol below. Do NOT proceed to Stage 2 or Stage 3. |

**RED Protocol:**

1. Do NOT configure cron with a broken script — if you haven't added cron entries yet, hold. If you have, run `crontab -e` and comment out the `run-daily.sh` line immediately.
2. File error report at `Reports/deployment-error-daily-pipeline-[date].md` — paste the last 30 log lines.
3. Post a queue item to FORGE (file at `10 - Agents/FORGE/Queue/`) with:
   - Team ID count from Step 1 log line
   - Last 5 log lines
   - The failure exit code (from `DAILY PIPELINE FAILED (exit code: N)`)
4. Do not retry without FORGE's corrected spec.
5. `run-overnight.sh` remains your fallback for nightly data refresh — it is untouched and functional.

---

## Step 5: Cron Configuration and Deployment Completion

After GREEN from Step 4:

**Confirm cron entries are in place** (should have been set earlier per Session Guide Section 4):

```bash
crontab -l | grep -E "run-daily|run-overnight"
```

Expected: Mon–Sat entry and Sunday entry both visible.

If cron is not yet configured, add it now:

```bash
crontab -e
```

Add:

```crontab
# Evo Draw — Daily Pipeline (active teams only, Mon–Sat, 2 AM)
0 2 * * 1-6 cd /path/to/evo-draw && bash run-daily.sh

# Evo Draw — Weekly full crawl (all IDs, Sunday 11 PM)
0 23 * * 0 cd /path/to/evo-draw && bash run-overnight.sh
```

Replace `/path/to/evo-draw` with the actual absolute project path.

**Verify cron saved:**

```bash
crontab -l | grep run-daily
```

Expected: the Mon–Sat entry is visible.

**Post-deployment actions:**

1. Note in FORGE's next briefing: "May 1 deployment GREEN. Cron live: `0 2 * * 1-6`. First automated run: May 2 at 2:00 AM. Manual test: GREEN — active teams [N], completion marker confirmed, rankings updated."
2. Mark `10 - Agents/FORGE/Tasks/task-2026-04-24-implement-daily-pipeline.md` status: `completed`.
3. Open `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — run it on May 2 morning.
4. **Do NOT run Stage 3 backfill tonight** — per `Infrastructure/Stale Team Backfill Runbook — April 2026.md` Section "Stage 3 Sequencing Constraint": Stage 3 must not run same night as first daily pipeline run. Confirm first overnight cron run is GREEN (May 2 morning) before scheduling Stage 3.

---

## Expected First-Run Numbers

| Metric | Expected Range | Source |
|--------|---------------|--------|
| Active team IDs | 15,000–40,000 | Spring peak season estimate |
| Total run duration | 3.5–7 hours | Crawl (2.5–5h) + pipeline (1–2h) |
| GotSport games ingested | Pending Stage 1 results | See `task-2026-04-25-stage1-backfill-results-report.md` when available |
| Rankings updated_at delta | ≤ 2 hours after run completion | Normal pipeline timing |

> [!note]
> If Stage 1 Results Report is not yet available when you run this, record the actual game count and active team count from this run. These become the baseline for the Morning-After Runbook's GREEN threshold.

---

## Rollback Protocol

If the first run or subsequent cron runs cause unexpected issues (ranking regressions, excessive API load, data anomalies):

**Halt daily cron:**

```bash
crontab -e
# Comment out the Mon-Sat run-daily.sh line:
# 0 2 * * 1-6 cd /path/to/evo-draw && bash run-daily.sh
```

**Re-enable overnight batch as fallback:**

```bash
bash run-overnight.sh &
```

Rankings will refresh with the full crawl within 12 hours.

**Rollback triggers:**
- Rankings for known active teams not updating day-over-day
- Active ID count dropping below 5,000 on successive runs
- `DAILY PIPELINE FAILED` on two consecutive scheduled runs
- `MAX(updated_at)` in rankings > 36 hours old

**After rollback:** File a queue item to FORGE with the failure pattern before attempting any re-deployment.

**Partial run cleanup:** A partial first run does not corrupt the database — `set -e` in `run-daily.sh` exits cleanly on first error, and no partial state is written to the rankings table until `compute-rankings.js` completes. A re-run after fixing the failure is safe.

---

## References

- `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` — pre-conditions this runbook follows
- `Infrastructure/May 1 Daily Pipeline Deployment Session Guide.md` — full session context and script contents
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — post-run health check (May 2)
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — validation query patterns
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — config context for adding new org IDs
- `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — Stage 3 sequencing constraint
- `task-2026-04-25-stage1-backfill-results-report.md` — Stage 1 results informing expected game counts

*FORGE — 2026-04-25*
