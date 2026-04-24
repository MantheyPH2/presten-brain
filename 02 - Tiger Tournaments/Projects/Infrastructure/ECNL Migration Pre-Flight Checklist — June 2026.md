---
title: ECNL Migration Pre-Flight Checklist — June 2026
tags: [ecnl, migration, pre-flight, infrastructure, june-2026, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: FORGE
status: ready — run before June 1, 2026
task: task-2026-04-24-ecnl-migration-pre-flight-checklist
due: 2026-05-05
companion: "ELO task: task-2026-04-24-ecnl-rating-continuity-spec"
---

# ECNL Migration Pre-Flight Checklist — June 2026

**Purpose:** Confirm FORGE infrastructure readiness before ECNL migrates from TGS AthleteOne to GotSport (June 2026). Run this checklist in the week before the migration window (target: June 7–14, 2026 based on ECNL org timeline).

**Who runs this:** Presten (with FORGE support).

**Companion document:** ELO is writing `Rankings/ECNL Rating Continuity Spec — June 2026.md` covering rating carry-through. When that spec is complete, review it and add any FORGE-side requirements to Section 3 of this checklist.

---

## Section 1: Code Prerequisites

### 1.1 — ECNL Dedup P0 Fix: `verified = true` in All team_merges INSERTs

**Background:** When `ecnl-transition-dedup.js` runs, it must insert `verified = true` on every `team_merges` row. If `verified` is absent or `false`, the Ranking Engine ignores all new merge records and every affected ECNL team resets to ~1000 rating overnight. Estimated scope: 500–2,000 teams.

**Current status (as of 2026-04-24):** The script `scripts/ecnl-transition-dedup.js` does not yet exist. FORGE writes it starting May 1. The mandatory INSERT pattern is documented in `Reports/ecnl-dedup-verified-fix-2026-04-24.md`.

**Confirm by running after the script exists and after it has been executed:**

```sql
-- Confirm all ECNL_TRANSITION_2026 merges are verified
SELECT verified, COUNT(*) as count
FROM team_merges
WHERE verified_by = 'ECNL_TRANSITION_2026'
GROUP BY verified;
```

**Expected result:** One row only — `verified = true`, count = N (number of ECNL teams migrated). If any row shows `verified = false`: **do NOT run `compute-rankings.js`** — escalate to FORGE/SENTINEL immediately.

**Alternate confirmation** (to check for any unintended unverified transition rows):

```sql
SELECT COUNT(*) FROM team_merges 
WHERE merge_type = 'ecnl_transition' 
  AND (verified IS NULL OR verified = false);
```

**Expected result:** 0. If > 0: fix is not applied correctly.

---

### 1.2 — Exponential Backoff Deployed in Sync Runner

**Why this matters:** If the sync runner is still on a fixed delay with no retry logic, any GotSport rate-limit event in the June migration window leaves newly-discovered ECNL teams stale without a recovery path.

**Confirm by running in the project directory:**

```bash
grep -n "fetchWithBackoff" crawl-gotsport-api-parallel.js
```

**Expected result:** Line number(s) returned showing `fetchWithBackoff` function definition and call sites. If not found: the sync runner is still on the old fixed-delay implementation — deploy the backoff fix from `Reports/backoff-implementation-2026-04-24.md` before the migration.

---

### 1.3 — Daily Pipeline Tested and Stable

**Why this matters:** The first post-migration pipeline run must complete successfully to incorporate ECNL teams into GotSport-sourced rankings. A pipeline that has been running stably for 3+ weeks before June 1 is dramatically lower risk than one first deployed in late May.

**Target:** At least 3 consecutive successful daily pipeline runs before June 1.

**Log file location:** `logs/daily-pipeline-YYYY-MM-DD.log` (project root). See `Infrastructure/Daily Pipeline Log Format Spec.md` for the full format spec.

**A "successful" run is defined as:** the final line of the log file is `DAILY PIPELINE COMPLETED SUCCESSFULLY`.

**Confirm by running this verification command from the project root:**

```bash
for i in 1 2 3; do
  D=$(date -v-${i}d '+%Y-%m-%d' 2>/dev/null || date -d "${i} days ago" '+%Y-%m-%d');
  grep -q "DAILY PIPELINE COMPLETED SUCCESSFULLY" "logs/daily-pipeline-${D}.log" 2>/dev/null \
    && echo "${D}: PASS" || echo "${D}: FAIL"
done
```

**Expected output (pre-flight cleared):**
```
2026-MM-DD: PASS
2026-MM-DD: PASS
2026-MM-DD: PASS
```

**If any line shows FAIL:** the pre-flight is NOT cleared. Check the failed log file for the `DAILY PIPELINE FAILED` line and the `ERROR:` line preceding it.

**Additional crawl health check** (optional but recommended):
```bash
grep "SYNC_RUN_SUMMARY" Logs/gotsport-sync/$(date '+%Y-%m-%d')-*.log | tail -3
```
Review `teams_fetched_success` > 0 and `teams_dlq` is documented (not necessarily 0).

---

## Section 2: Data Prerequisites

### 2.1 — Stale Team Backfill Complete (All 3 Stages)

**Why this matters:** Stale teams interacting with the ECNL migration may produce orphaned records if their last-sync state is inconsistent. The backfill clears the 540 rate-limit-casualty teams before migration adds new complexity.

**Confirm by checking the Stage 3 completion log:**

```bash
grep "SYNC_RUN_SUMMARY" Logs/gotsport-sync/$(ls Logs/gotsport-sync/ | sort | tail -1) | tail -1
```

Also run all 3 post-backfill verification queries from `Infrastructure/Stale Team Backfill Runbook — April 2026.md`. All three must pass:

```sql
-- Post-Stage 3: check overall completion (expects ≥95% of 540 recently synced)
SELECT 
  COUNT(*) FILTER (WHERE last_sync_date > NOW() - INTERVAL '48 hours') as recently_synced,
  COUNT(*) FILTER (WHERE last_sync_date <= NOW() - INTERVAL '48 hours' OR last_sync_date IS NULL) as still_stale,
  COUNT(*) as total
FROM teams
WHERE id IN (<all-540-stale-ids>);
```

**Expected:** `recently_synced` ≥ 510 (>95%). Remaining stale IDs should have a DLQ file explaining why.

**If Stage 3 did not complete:** Proceed with awareness — the 480-team batch is still stale. Newly-discovered ECNL teams from this group may carry partial data into the migration. Log this as a known risk in the pre-flight report.

---

### 2.2 — No New Wave of Stale Teams in the 7 Days Before Migration

**Why this matters:** A new rate-limit event or backoff failure in the weeks before June 1 would create a second wave of stale teams entering the migration window.

**Run this query in the week before the migration:**

```sql
SELECT COUNT(*) 
FROM teams 
WHERE last_game_date < NOW() - INTERVAL '90 days' 
  AND is_active = true;
```

**Document the count** as of migration day. If the count is more than 50 new stale teams since the backfill completed (or more than ~100 total), investigate before proceeding. Likely causes: backoff fix not working, GotSport rate limit tightened again, or new API behavior. **This is recommended, not blocking.** Document the count and proceed.

---

### 2.3 — GotSport Org-ID Expansion Deployed

**Why this matters:** ECNL state-association teams that are registered under USYS state org IDs but not in our ranked team discovery feed will not be found by the crawler — meaning newly-GotSport-hosted ECNL teams might not be discovered until a full sweep.

**Confirm at least TX and FL org IDs are in the production discovery config:**

Check the file updated by the Org-ID Expansion task (`Data Sources/GotSport USYS Org-ID Master List.md`) and confirm the config file referenced there contains at least 2 confirmed state org IDs.

**This is recommended, not blocking.** If neither org ID is confirmed, note: "Org-ID expansion not deployed — ECNL state-association teams may not be auto-discovered after migration. Full discovery sweep may be needed manually after June 1."

---

## Section 3: Migration Script Prerequisites

### 3.1 — Migration Script Location

**Script status as of 2026-04-24:** `scripts/ecnl-transition-dedup.js` **DOES NOT EXIST YET.**

This is a known and planned gap. FORGE begins writing it on **May 1, 2026**, per the critical path in `Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md`.

**⚠️ CRITICAL GAP — SENTINEL FLAGGED**

The migration script not existing on April 24 is expected per the schedule. It becomes a blocking gap if not written by **May 15** (one day before DSS, two weeks before June 1). SENTINEL must confirm by May 5 that the May 1 development session is on Presten's calendar.

**Once written, document the file path here:**

```
Migration script file path: scripts/ecnl-transition-dedup.js
Confirmed written: [DATE]
Confirmed reviewed by Presten: [DATE]
```

**To verify after the script is written:**

```bash
ls -la scripts/ecnl-transition-dedup.js
grep -c "verified = true" scripts/ecnl-transition-dedup.js
```

**Expected:** File exists, and `verified = true` appears in at least 1 INSERT statement. See `Reports/ecnl-dedup-verified-fix-2026-04-24.md` for the mandatory INSERT pattern.

---

### 3.2 — Script Tested on a Sample Before Production Run

**Required:** At least 10 ECNL teams tested (mix of merged and unmerged cases) before the script runs on the full ECNL team population.

**What a passing test looks like:**
- No duplicate `team_merges` entries created (check: `SELECT COUNT(*) FROM team_merges WHERE verified_by = 'ECNL_TRANSITION_2026'` before and after test — should increase by exactly the number of test teams, no more)
- No orphaned game records (check: `SELECT COUNT(*) FROM games WHERE source IN ('tgs', 'tgs_athleteone') AND is_hidden = false` — should be unchanged if TGS records are not being deleted, only linked)

**Document the test date and sample size when run.**

This is recommended. If the script is not tested on a sample before June 1, run with heightened monitoring and have the rollback plan ready.

---

### 3.3 — Pre-Migration Database Backup

**A backup must exist within 24 hours of the migration run. This is non-negotiable.**

**Backup command:**

```bash
pg_dump -U p -h localhost -p 5432 youth_soccer > \
  /path/to/backups/youth_soccer-pre-ecnl-migration-$(date +%Y-%m-%d).sql
```

Adapt the output path to wherever backups are stored on the server. The backup file is the rollback target if the migration produces bad data that cannot be reversed in-place.

**Confirm backup exists:**

```bash
ls -lh /path/to/backups/youth_soccer-pre-ecnl-migration-*.sql | tail -3
```

**Expected:** At least one `.sql` file with a date within 24 hours of the planned migration run.

---

## Section 4: Pre-Flight Go/No-Go Rule

Run through this checklist the day before the migration. Do not proceed unless all blocking items are cleared.

```
CLEARED TO PROCEED if ALL of the following are true:

□ Section 1.1: verified=true confirmed in team_merges
  (SQL returns 0 unverified ecnl_transition rows)

□ Section 1.2: fetchWithBackoff() present in crawl-gotsport-api-parallel.js
  (grep returns at least one match)

□ Section 1.3: At least 3 successful daily pipeline runs logged
  (SYNC_RUN_SUMMARY present in last 3 daily logs)

□ Section 2.1: Stale team backfill Stages 1–3 complete and verified
  (Post-Stage 3 query shows ≥95% recently synced)

□ Section 3.1: Migration script file path identified and script exists
  (ls -la scripts/ecnl-transition-dedup.js returns a file)

□ Section 3.3: Pre-migration database backup taken within 24 hours
  (Backup file exists with today's date)


NOT CLEARED — escalate to SENTINEL if ANY of the following:

□ Section 1.1: ecnl_transition merges with verified = NULL or false > 0
  → Do NOT run compute-rankings.js. Stop and notify.

□ Section 3.1: Migration script does not exist as of June 1
  → CRITICAL — migration cannot proceed without it. Stop.

□ Section 2.1: Stage 3 backfill did not complete
  → Log as known risk; proceed with awareness (not blocking but high risk).

□ Section 3.3: No backup exists within 24 hours of migration
  → Do NOT proceed. Create the backup first.
```

**Recommended but not blocking:**
- Section 1.2: Backoff deployed (recommended; not blocking if daily pipeline has been stable)
- Section 2.2: New stale team count documented (not blocking)
- Section 2.3: Org-ID expansion deployed (not blocking; affects discovery, not migration correctness)
- Section 3.2: Script tested on sample (strongly recommended but cannot be blocking if the pre-migration window is tight)

---

## Section 5: Post-Migration Handoff

After the migration run completes, immediately check:

1. Run Section 1.1 verification query — confirm all ECNL_TRANSITION_2026 merges have `verified = true`
2. Run `compute-rankings.js` only after Section 1.1 passes
3. Run ELO's June 2 post-migration verification spec (`Rankings/June 2 ECNL Migration Verification Spec.md`) the morning after the run

If any blockers are tripped: do NOT run `compute-rankings.js`. The Ranking Engine will use the pre-migration state until the issue is resolved.

---

## Coordination Notes

| Agent | Companion Document | FORGE Action |
|---|---|---|
| ELO | `Rankings/ECNL Rating Continuity Spec — June 2026.md` (in progress) | When complete: review and add any FORGE-side requirements to Section 3 of this checklist |
| SENTINEL | SENTINEL Briefing ECNL section | SENTINEL will surface pre-flight checklist status to Presten as part of May status reporting |

---

## References

- `Reports/ecnl-dedup-verified-fix-2026-04-24.md` — P0 INSERT spec (Section 1.1)
- `Reports/backoff-implementation-2026-04-24.md` — backoff spec (Section 1.2)
- `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — backfill verification queries (Section 2.1)
- `Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md` — critical path schedule
- `Rankings/June 2 ECNL Migration Verification Spec.md` — ELO's post-migration check (companion)
- `Infrastructure/Database Schema.md` — `team_merges` table definition (verified column reference)

---

*FORGE — 2026-04-24*
