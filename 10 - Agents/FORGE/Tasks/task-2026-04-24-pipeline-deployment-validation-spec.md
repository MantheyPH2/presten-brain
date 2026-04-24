---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: medium
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Pipeline Deployment Validation Spec — April 2026.md"
---

# Task: Pipeline Deployment Validation Spec

## Context

Three pipeline deployments are pending Presten execution, each gating the next:

1. **Backoff fix** — `fetchWithBackoff()` deployed to `crawl-gotsport-api-parallel.js`
2. **TX org-ID test crawl** — Stage 1 of the GotSport org-ID expansion using TX org_ids
3. **Daily pipeline** — `run-daily.sh` + `get-active-team-ids.sh` + cron configuration

When Presten executes these, FORGE should have a validation spec ready. Currently, the vault has:
- `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — covers backfill after Stage 1 runs
- `Infrastructure/Daily Pipeline Log Format Spec.md` — covers log format
- `Product & Planning/Presten Session Plans — April 2026.md` — covers session sequencing

What does not exist: a document telling Presten what to check immediately after deploying each component to confirm it worked before moving to the next step.

FORGE has the specs for all three components. This task: write the deployment validation criteria.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Infrastructure/Pipeline Deployment Validation Spec — April 2026.md`

This document is a checklist Presten uses during the deployment session (estimated: 2–3 hours). It runs after Presten deploys code, not before.

---

### Section 1: Backoff Fix Validation

**After deploying `fetchWithBackoff()` to `crawl-gotsport-api-parallel.js`:**

```bash
# Run a minimal crawl against a known org_id to confirm backoff activates without crash
node crawl-gotsport-api-parallel.js --org [small-test-org-id] --dry-run 2>&1 | head -50
```

Pass criteria:
- Script runs without crashing on the first request
- If a 429 is returned: log shows backoff delay message (not crash)
- If no 429: script completes and outputs game data normally

**What to look for in the output:**
- `Retrying after [X]ms` — backoff activated correctly
- `Error: Too many retries` — backoff is activating but max retries is too low; not a crash, but escalate to FORGE for config review
- Any uncaught exception — backoff code has a bug; escalate to FORGE immediately

If the script crashes: do NOT proceed to Stage 1 backfill. File the error output in `Reports/backoff-deployment-error-[date].md` and post to FORGE.

---

### Section 2: TX Org-ID Test Crawl Validation (Stage 1)

**After running the TX test crawl:**

Per the GotSport org-ID expansion task, Stage 1 uses TX org_ids as the pilot cohort. After the crawl completes:

```sql
-- Confirm new games were ingested (check for games from TX-area org_ids)
SELECT COUNT(*) AS new_games, MAX(created_at) AS latest_ingested
FROM games
WHERE created_at > NOW() - INTERVAL '24 hours';
```

Pass criteria:
- `new_games > 0` — crawl returned data
- `latest_ingested` is within the last hour — freshness confirms correct DB write

```sql
-- Confirm no duplicate games from the test crawl
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*) AS count
FROM games
WHERE created_at > NOW() - INTERVAL '24 hours'
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1;
```

Pass criteria: `0 rows` — no duplicates introduced.

If duplicates exist: do NOT proceed to Stage 2 or Stage 3. File at `Reports/gotsport-org-expansion-test-[date].md` with the duplicate game IDs and the org_id that produced them. Escalate to FORGE.

**Also confirm Stage 1 exit condition from the Backfill Runbook:** Stage 1 is complete when game count for new org_ids stabilizes (no new games added on second crawl). Note the game count and org_id in the report.

---

### Section 3: Daily Pipeline Validation

**After deploying `run-daily.sh` and configuring cron:**

First, trigger a manual run (not the cron):
```bash
bash run-daily.sh 2>&1 | tee /tmp/daily-pipeline-test.log
```

Per `Infrastructure/Daily Pipeline Log Format Spec.md`, the last line of the log must be:
```
DAILY PIPELINE COMPLETED SUCCESSFULLY
```

Verify:
```bash
tail -1 /tmp/daily-pipeline-test.log
```

Pass: exact string match. Fail: investigate the preceding log lines for the step that failed.

**Cron verification:**
```bash
crontab -l | grep run-daily
```
Expected: an entry matching the configured schedule. If not found: cron was not saved correctly.

**Log file location check:**
```bash
ls -la logs/daily-pipeline-$(date +%Y-%m-%d).log
```
Expected: file exists with content. If missing: the log path in `run-daily.sh` does not match the spec.

---

### Section 4: Deployment Completion Checklist

```
□ Backoff fix: minimal crawl ran without crash (Section 1)
□ Backoff fix: 429 handling logged (or confirmed no 429 in test environment)
□ TX test crawl: new games ingested (> 0), no duplicates
□ TX test crawl: game count and org_id documented in Reports/
□ Daily pipeline: DAILY PIPELINE COMPLETED SUCCESSFULLY on manual run
□ Daily pipeline: cron entry confirmed (crontab -l)
□ Daily pipeline: log file created at expected path
□ All three: note completion time in Presten Session Plans log
```

If any box cannot be checked: stop, note the failure, and post to FORGE before proceeding.

---

## Definition of Done

- Validation spec written at the specified path
- All three deployments covered with concrete pass/fail criteria
- Each section has at least one copy-paste command Presten can run immediately after deploying
- Completion checklist at the end (4-minute review)
- Summary in briefing: "Pipeline deployment validation spec written. Covers: backoff fix, TX org-ID test crawl, daily pipeline. Presten can run this during the deployment session."

## Why This Matters

When Presten deploys code, there is currently no document that says "here is how you know it worked." The Backoff spec tells Presten what to build. The Backfill Runbook tells Presten how to run Stage 1. But neither tells Presten what a successful Stage 1 crawl looks like. This spec closes the gap between "I deployed it" and "I know it worked."

## References

- `02 - Tiger Tournaments/Projects/Infrastructure/Daily Pipeline Log Format Spec.md` — log format spec
- `02 - Tiger Tournaments/Projects/Infrastructure/Stale Team Backfill Runbook — April 2026.md` — Stage 1 exit conditions
- `10 - Agents/FORGE/Tasks/task-2026-04-24-implement-option-b-backoff.md` — backoff deployment spec
- `10 - Agents/FORGE/Tasks/task-2026-04-24-gotsport-org-id-expansion.md` — TX test crawl spec
- `10 - Agents/FORGE/Tasks/task-2026-04-24-implement-daily-pipeline.md` — daily pipeline deployment spec
