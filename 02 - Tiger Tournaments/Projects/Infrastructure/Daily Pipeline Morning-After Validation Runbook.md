---
title: Daily Pipeline Morning-After Validation Runbook
tags: [infrastructure, pipeline, validation, runbook, daily-pipeline, evo-draw]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: ready — use on May 2, 2026 (morning after first automated cron run)
---

# Daily Pipeline Morning-After Validation Runbook

---

## When to Run

**May 2, 2026 morning — before any other pipeline or backfill work.**

This runbook verifies the first unattended cron execution of `run-daily.sh` (scheduled `0 2 * * 1-6`). The first automated run is the highest-risk moment. Do not proceed to Stage 3 backfill until all 5 steps below pass.

---

## Prerequisites

- May 1 deployment completed and confirmed GREEN (all Section 5 checks in `Infrastructure/May 1 Daily Pipeline Deployment Session Guide.md` passed)
- You have SSH or terminal access to the production server

---

## 5-Step Verification Sequence

### Step 1 — Confirm the Cron Fired at 2 AM

```bash
grep "DAILY PIPELINE STARTED" logs/daily-pipeline-$(date -v-1d '+%Y-%m-%d' 2>/dev/null || date -d 'yesterday' '+%Y-%m-%d').log
```

**Expected output:**
```
[2026-05-02 02:00:XX] DAILY PIPELINE STARTED
```

**PASS:** Line appears with timestamp between `02:00:00` and `02:05:00`.
**FAIL:** File does not exist OR no `DAILY PIPELINE STARTED` line → cron did not fire. **STOP. See RED decision below.**

> [!tip] Log filename
> The log file is at `logs/daily-pipeline-YYYY-MM-DD.log` where the date is the run date (May 2, which is the date the 2 AM run fires). If the server is UTC and your local is different, check `logs/` with `ls -lt logs/ | head -5` to find the most recent file.

---

### Step 2 — Confirm the Pipeline Produced Output

```bash
grep "STEP 1: Crawl started" logs/daily-pipeline-$(date -v-1d '+%Y-%m-%d' 2>/dev/null || date -d 'yesterday' '+%Y-%m-%d').log
```

**Expected output:**
```
[2026-05-02 02:00:XX] STEP 1: Crawl started (team IDs: N)
```

Where `N` is a number between **15,000 and 40,000**.

**PASS:** Line present AND `N` is between 15,000 and 40,000.
**FAIL (too low):** `N < 1,000` → `get-active-team-ids.sh` failed silently or returned empty. **STOP. RED.**
**FAIL (too high):** `N > 100,000` → active team filter not working; full 107K crawl ran unintended. **STOP. RED.**

---

### Step 3 — Confirm No Error-Level Failures

```bash
grep -E "DAILY PIPELINE FAILED|ERROR|UNHANDLED_REJECTION" logs/daily-pipeline-$(date -v-1d '+%Y-%m-%d' 2>/dev/null || date -d 'yesterday' '+%Y-%m-%d').log | head -20
```

**Expected output:** No lines returned (empty).

**PASS:** Zero lines returned.
**YELLOW:** Lines returned but include only `UNHANDLED_REJECTION` or `WARN`-level events (not `DAILY PIPELINE FAILED`) → investigate but do not halt.
**FAIL:** Any line containing `DAILY PIPELINE FAILED` → **STOP. RED.**

---

### Step 4 — Confirm DB Ingestion Advanced

```sql
SELECT MAX(created_at) AS last_game_inserted
FROM games
WHERE is_hidden = false
  AND source IN ('gotsport_api', 'gotsport');
```

**Expected output:** `last_game_inserted` timestamp is within the last 24 hours (i.e., later than `2026-05-01 02:00:00`).

**PASS:** Timestamp is after May 1 2 AM.
**YELLOW:** Timestamp is older — no new games inserted. This may be legitimate if no games were played overnight. Check Step 4 note below before deciding.
**FAIL:** Cannot connect to DB, or `MAX(created_at)` predates May 1 deployment. **STOP. RED.**

> [!note] Step 4 YELLOW investigation
> Before escalating a YELLOW on Step 4, check whether games were expected overnight. Spring season games typically run weekends (Sat/Sun). If the first cron fired on a Monday night with no games scheduled, zero new rows is correct. Run: `SELECT COUNT(*) FROM games WHERE match_date = CURRENT_DATE - 1 AND is_hidden = false AND source IN ('gotsport_api','gotsport');` — if 0, that's the answer, not a pipeline failure.

---

### Step 5 — Stage 3 Backfill Gate

**Before starting any Stage 3 backfill work, confirm:**

- [ ] Step 1 PASS
- [ ] Step 2 PASS (N between 15,000–40,000)
- [ ] Step 3 PASS (no ERROR lines)
- [ ] Step 4 PASS or YELLOW-explained

If all four boxes are checked: Stage 3 backfill may proceed this session.

**If any box is unchecked: STOP. Do not run Stage 3. Follow the RED procedure below.**

---

## GREEN / YELLOW / RED Decision

### GREEN — Proceed

**Criteria:** Steps 1–3 all PASS, Step 4 PASS or YELLOW-explained.

**Next action:** Stage 3 backfill may proceed. Note first-run results in a message to FORGE: "Daily pipeline first automated run confirmed GREEN. Active IDs: [N]. New games: [Y or 0 — explained]. Proceeding to Stage 3."

---

### YELLOW — Investigate Before Proceeding

**Criteria:** Steps 1–3 pass, Step 4 shows no new games.

**Next action:** Run the Step 4 YELLOW investigation query above. If zero games played overnight is confirmed (game calendar check passes), upgrade to GREEN and proceed. If game calendar shows games should have been ingested but weren't: **hold Stage 3 and contact FORGE.**

---

### RED — Halt

**Criteria:** Any of the following:
- Step 1: `DAILY PIPELINE STARTED` line absent (cron did not fire)
- Step 2: `N < 1,000` or `N > 100,000`
- Step 3: `DAILY PIPELINE FAILED` present in log
- Step 4: DB unreachable or `MAX(created_at)` predates deployment

**Next action:**
1. Do not run Stage 3 backfill
2. Do not manually re-run `run-daily.sh` without FORGE review
3. Send FORGE the log output using the template below

---

## If RED: What to Send FORGE

Copy and paste this template with actual values filled in:

```
DAILY PIPELINE FIRST RUN — RED STATUS

Step 1 result: [PASS / FAIL — cron did/did not fire]
Step 2 result: [PASS / FAIL — team IDs: N]
Step 3 result: [PASS / FAIL — error lines below]
Step 4 result: [PASS / FAIL / YELLOW — last ingestion: timestamp]

Relevant log lines:
[paste the output of: grep -E "PIPELINE|ERROR|FAILED|STEP" logs/daily-pipeline-[date].log | tail -30]

Crontab (current):
[paste: crontab -l]
```

FORGE will respond with a corrected spec or next diagnostic step before any retry.

---

## References

- `Infrastructure/May 1 Daily Pipeline Deployment Session Guide.md` — deployment guide this runbook follows
- `Infrastructure/April 29 Pipeline Health Check — Post-Fix Protocol.md` — pre-deployment green-light check
- `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — Stage 3 sequencing constraint

*FORGE — 2026-04-25*
