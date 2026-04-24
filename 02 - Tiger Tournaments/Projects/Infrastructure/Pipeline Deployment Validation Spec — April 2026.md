---
title: Pipeline Deployment Validation Spec — April 2026
tags: [infrastructure, deployment, validation, pipeline, evo-draw, forge]
created: 2026-04-24
updated: 2026-04-24
author: FORGE
status: ready
task: task-2026-04-24-pipeline-deployment-validation-spec
---

# Pipeline Deployment Validation Spec — April 2026

> [!info] Purpose
> Use this checklist **after** deploying each pipeline component to confirm it worked before moving to the next step. Run in sequence: Backoff → TX Org-ID Test → Daily Pipeline. Estimated session time: 2–3 hours.

---

## Section 1: Backoff Fix Validation

**Trigger:** After deploying `fetchWithBackoff()` to `crawl-gotsport-api-parallel.js`.

**Step 1: Confirm function is present in the code:**

```bash
grep -n "fetchWithBackoff" crawl-gotsport-api-parallel.js
```

Expected: function definition found (line number printed). If not found: backoff was not deployed — stop, do not proceed.

**Step 2: Run a minimal dry-run crawl to confirm no crash:**

```bash
node crawl-gotsport-api-parallel.js --org <small-test-org-id> --dry-run 2>&1 | head -50
```

**What to look for in the output:**

| Output | Meaning | Action |
|--------|---------|--------|
| `Retrying after Xms` | Backoff activated correctly — 429 received and handled | ✅ Pass |
| Script runs without crash, game data returned | No 429 in test; normal operation | ✅ Pass |
| `Error: Too many retries` | Backoff activating but max retries too low for this org | ⚠️ Not a crash — escalate to FORGE for config review before Stage 1 |
| Any uncaught exception / stack trace | Backoff code has a bug | 🚨 Stop — file error output at `Reports/backoff-deployment-error-YYYY-MM-DD.md`, notify FORGE |

**Step 3: Verify log format includes backoff events:**

After any run that encounters a 429, the log should include a line matching:

```
{ event: 'API_RETRY', team_id: ..., attempt: ..., delay_ms: ..., status_code: 429 }
```

And the post-run summary should include:

```
{ event: 'SYNC_RUN_SUMMARY', teams_discovered: ..., teams_fetched_success: ..., teams_dlq: ..., ... }
```

**Pass criteria for Section 1:**
- [ ] `fetchWithBackoff` found in source file
- [ ] Dry-run completes without uncaught exception
- [ ] If 429 occurred: backoff delay logged (not crash)

> [!warning] Gate
> If any criterion fails: **do NOT proceed to Section 2 (TX test crawl).** File the error and post to FORGE.

---

## Section 2: TX Org-ID Test Crawl Validation (Stage 1)

**Trigger:** After running the Texas USYS state association org-ID test crawl (Stage 1 per `Data Sources/GotSport USYS Org-ID Master List.md`).

**Step 1: Confirm new games were ingested:**

```sql
SELECT COUNT(*) AS new_games, MAX(created_at) AS latest_ingested
FROM games
WHERE created_at > NOW() - INTERVAL '24 hours';
```

**Pass criteria:**
- `new_games > 0` — crawl returned data (if 0: org_id may be wrong or no events in range)
- `latest_ingested` is within the last 2 hours — confirms data was written to DB

**Step 2: Confirm no duplicate games from the test crawl:**

```sql
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*) AS count
FROM games
WHERE created_at > NOW() - INTERVAL '24 hours'
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1;
```

**Pass criteria:** `0 rows` — no duplicates introduced.

> [!warning] If duplicates exist
> Do NOT proceed to Stage 2 or Stage 3 of the org-ID expansion. File the duplicate game IDs and the org_id that produced them at `Reports/gotsport-org-expansion-test-YYYY-MM-DD.md` and notify FORGE before any further crawl runs.

**Step 3: Document the Stage 1 results:**

Note in `Reports/gotsport-org-expansion-test-2026-04-24.md`:
- The TX org_id used
- Number of new team IDs discovered (not already in `teams` table)
- Number of new games ingested in this run
- Spot-check: review 5 new teams — confirm they are real USYS state league/cup teams

**Step 4: Spot-check new teams for legitimacy:**

```sql
SELECT id, name, source, created_at
FROM teams
WHERE created_at > NOW() - INTERVAL '24 hours'
ORDER BY created_at DESC
LIMIT 10;
```

Review names manually — confirm they look like youth soccer club teams (not test data or duplicates of existing teams under different names).

**Pass criteria for Section 2:**
- [ ] `new_games > 0` from the 24h query
- [ ] `0` duplicate rows from the duplicate check
- [ ] TX org_id and game count documented in `Reports/gotsport-org-expansion-test-2026-04-24.md`
- [ ] 5 new teams spot-checked and look legitimate

> [!warning] Gate
> If any criterion fails: **do NOT add additional state org IDs to config.** Fix the issue with TX first.

---

## Section 3: Daily Pipeline Validation

**Trigger:** After deploying `run-daily.sh`, `get-active-team-ids.sh`, and configuring cron.

**Step 1: Trigger a manual run (not the cron) and capture the log:**

```bash
bash run-daily.sh 2>&1 | tee /tmp/daily-pipeline-test.log
```

Wait for completion (expected: up to 6 hours for full active-team crawl; shorter for a test subset).

**Step 2: Check the last line of the log for the success marker:**

```bash
tail -1 /tmp/daily-pipeline-test.log
```

Expected (exact string):
```
[YYYY-MM-DD HH:MM:SS] DAILY PIPELINE COMPLETED SUCCESSFULLY
```

Per `Infrastructure/Daily Pipeline Log Format Spec.md`, this is the definitive pass indicator. If the last line is anything else (especially `DAILY PIPELINE FAILED`), investigate the preceding log lines to identify which step failed.

**Step 3: Confirm the cron entry is saved:**

```bash
crontab -l | grep run-daily
```

Expected output: an entry matching the configured schedule (e.g., `0 2 * * 1-6 /path/to/run-daily.sh`). If empty: cron was not saved — re-add the cron entry.

**Step 4: Confirm the log file was written to the expected path:**

```bash
ls -la logs/daily-pipeline-$(date +%Y-%m-%d).log
```

Expected: file exists with content. If missing: the log redirect in `run-daily.sh` does not match the spec in `Infrastructure/Daily Pipeline Log Format Spec.md` — fix the redirect path.

**Step 5: Confirm active team ID count is in the expected range:**

Check the STEP 1 log line from the run:

```
[YYYY-MM-DD HH:MM:SS] STEP 1: Crawl started (team IDs: N)
```

Expected: `N` in the range 15,000–40,000. If N > 100,000: the active team filter (`get-active-team-ids.sh`) is not working — the script is defaulting to all team IDs. Stop, fix the filter query, and re-run.

**Pass criteria for Section 3:**
- [ ] Last line of test log is `DAILY PIPELINE COMPLETED SUCCESSFULLY`
- [ ] Cron entry confirmed via `crontab -l`
- [ ] Log file exists at `logs/daily-pipeline-YYYY-MM-DD.log`
- [ ] Active team ID count in range 15,000–40,000

---

## Section 4: Deployment Completion Checklist

Run this checklist after completing all three sections. Takes approximately 4 minutes.

```
BACKOFF FIX:
□ fetchWithBackoff present in crawl-gotsport-api-parallel.js (Section 1, Step 1)
□ Dry-run completed without uncaught exception (Section 1, Step 2)
□ 429 handling logged correctly (or confirmed no 429 in test environment)

TX ORG-ID TEST CRAWL (Stage 1):
□ New games ingested (> 0 from 24h query) (Section 2, Step 1)
□ Zero duplicate games from test crawl (Section 2, Step 2)
□ TX org_id and game count documented in Reports/gotsport-org-expansion-test-2026-04-24.md
□ 5 new teams spot-checked and look legitimate

DAILY PIPELINE:
□ Manual run completed: last log line = DAILY PIPELINE COMPLETED SUCCESSFULLY
□ Cron entry confirmed (crontab -l shows entry)
□ Log file created at expected path (logs/daily-pipeline-YYYY-MM-DD.log)
□ Active team ID count in range 15,000–40,000

SEQUENCING:
□ Note completion time in Presten Session Plans log
□ Stage 3 backfill (480 teams) NOT run same night as first daily pipeline — see sequencing warning in Stale Team Backfill Runbook
```

> [!warning] If any box cannot be checked
> Stop at the failed item. Note the failure mode. Post to FORGE before proceeding to the next component or the next stage of the backfill.

---

## Failure Filing Template

If a section fails, file a brief error note at:
`Reports/deployment-error-[component]-YYYY-MM-DD.md`

Contents:
1. Which section failed (Backoff / TX Crawl / Daily Pipeline)
2. Exact error message or unexpected output
3. Log file path or SQL output that showed the failure
4. Timestamp of failure

Post to FORGE in the next briefing cycle. Do not attempt to fix pipeline code without FORGE's input — the fix may require spec changes.

---

## References

- `Infrastructure/Daily Pipeline Log Format Spec.md` — log format spec (Section 3 depends on this)
- `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — Stage 1 exit conditions and sequencing constraint
- `Data Sources/GotSport USYS Org-ID Master List.md` — TX org_id source for Section 2
- `Reports/backoff-implementation-2026-04-24.md` — backoff code spec (what `fetchWithBackoff` does)
- `Reports/gotsport-org-expansion-test-2026-04-24.md` — where to file TX test results
- `Product & Planning/Presten Session Plans — April 2026.md` — session sequencing context

*FORGE — 2026-04-24*
