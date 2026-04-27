---
title: "May 1 — Pipeline Red Signal Response Protocol"
tags: [infrastructure, pipeline, monitoring, may1, red-signal, escalation, forge, evo-draw]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: ready — reference starting May 2 morning-after review
---

# May 1 — Pipeline Red Signal Response Protocol

> **Companion to:** [[May 1 — Per-Source Game Volume Forecast]] (defines thresholds) and [[Daily Pipeline — Morning-After Runbook]] (initial diagnostic tool).
>
> **Purpose:** After FORGE identifies a RED or YELLOW signal from the Morning-After Runbook, this protocol tells FORGE what to do — classify the signal, run the right diagnostic steps, and either resolve autonomously or escalate to Presten with a specific ask.
>
> **FORGE handles most signals autonomously.** Only true systemic failures require Presten.

---

## Section 1: Signal Classification Matrix

Enter with aggregate and per-source counts from the Morning-After Runbook.

| Signal | Aggregate Count | Per-Source Pattern | Classification | Response |
|--------|----------------|-------------------|---------------|----------|
| Full GREEN | ≥ 300 games | All sources ≥ GREEN floor | Healthy | No action. Log in briefing. |
| Aggregate YELLOW — weekday pattern | 50–299 games | Each source in YELLOW range (no source at RED) | Low-volume weekday | Log. No escalation. Re-check May 3–4 weekend before declaring a problem. |
| Aggregate YELLOW — one source RED | 50–299 games | 1 source below its RED floor; others normal | Source-specific issue | Investigate the RED source (Section 3). Do not pause pipeline. File issue report in briefing. |
| Aggregate RED | < 50 games | — | Pipeline not running OR systemic failure | Execute Section 2 diagnostic immediately. Do NOT wait for weekend data. |
| Aggregate GREEN, one source flat zero | ≥ 300 games | 1 source = 0 new games | Source silent failure | Investigate zero source (Section 3). File issue report within 24 hours. May mask future deterioration. |
| Aggregate GREEN, unusual distribution | ≥ 300 games | One source >> expected; others < expected | Possible counting artifact or config change | Note in briefing. Investigate distribution shift if it persists on May 3. |

**Weekday baseline note:** May 1 is a Thursday. The `gotsport_api` YELLOW range (50–199 games) on May 1 alone is **not an escalation trigger** — weekday volume is structurally lower than weekends. Re-check against the May 3–4 weekend run before calling it failed.

---

## Section 2: Aggregate RED Diagnostic Tree

**Enter here if:** aggregate new games < 50 after 48 hours (i.e., both May 1 and May 2 show < 50 games).

**Run sequentially. Stop at the first step that identifies the cause.**

---

**Step 1: Confirm cron is running**

Check the cron log or job runner output for a May 1 execution record.

```bash
# Check cron execution log (adapt path to actual cron log location)
grep "run-daily" /var/log/cron.log | grep "2026-05-01"
# Or check the job runner output file:
cat /path/to/pipeline/logs/may1-run.log | head -50
```

| Result | Action |
|--------|--------|
| No execution record for May 1 | **ESCALATE TO PRESTEN IMMEDIATELY.** "Daily pipeline cron did not execute on May 1. Deployment issue — cron may not be configured or the server was down. Action required: verify cron setup." |
| Execution record exists | Proceed to Step 2. |

---

**Step 2: Confirm pipeline completed without exception**

Check the pipeline exit code and error log for the May 1 run.

```bash
# Check for exceptions in pipeline log
grep -i "error\|exception\|failed\|uncaught" /path/to/pipeline/logs/may1-run.log | tail -20
```

| Result | Action |
|--------|--------|
| Exception in scraper code | Note specific exception. If exception is source-specific (one scraper failing), check per-source counts — other sources may be healthy. File issue report with exception text. |
| Exception in DB write path | **ESCALATE TO PRESTEN.** DB write exceptions are architecture-level failures, not source-level. |
| No exceptions logged | Proceed to Step 3. |

---

**Step 3: Confirm games are reaching the DB**

```sql
-- Check if any games were written on May 1
SELECT source, COUNT(*) AS games_written
FROM games
WHERE created_at >= '2026-05-01'
  AND created_at < '2026-05-02'
GROUP BY source
ORDER BY games_written DESC;
```

| Result | Action |
|--------|--------|
| Result set is empty | **ESCALATE TO PRESTEN.** "Pipeline ran on May 1 without exception but no games written to DB. DB write path may be broken. Please review." |
| Result has rows but aggregate < 50 | Scrapers are producing low output. Investigate per-source using Section 3. |
| Result matches expected per-source breakdown | Aggregate count < 50 is an artifact. Recheck the Morning-After Runbook query — possible timestamp filter issue. |

---

**Step 4 (if steps 1–3 do not isolate the cause):**

File an escalation briefing with all three diagnostic outputs. State: "Aggregate RED on May 1–2 with no clear source-specific cause. Steps 1–3 completed. SENTINEL review required before re-examining pipeline."

---

## Section 3: Source-Specific RED Response

For each source that hits its individual RED threshold, FORGE executes the following steps before escalating.

---

### gotsport_api RED (< 10 new games)

1. Confirm GotSport API credentials are valid:
   ```bash
   # Check scraper log for 401/403 responses
   grep -i "401\|403\|unauthorized\|forbidden" /path/to/logs/gotsport_api.log | tail -10
   ```
2. Check if GotSport had a known outage (manual check — no automated monitor available). Note this as a possibility in the issue report.
3. Check last successful run date:
   ```sql
   SELECT MAX(created_at) AS last_game_written
   FROM games
   WHERE source = 'gotsport'
     AND created_at < '2026-05-01';
   ```
4. | Result | Action |
   |--------|--------|
   | Auth error (401/403) | **ESCALATE TO PRESTEN.** "GotSport API credential failure on May 1. Scraper returning 401/403. Do not retry with stale credentials — credential rotation required." |
   | No auth error, last successful run > 3 days ago | Flag in briefing: "gotsport_api has not written new games since [date]. May indicate silent scraper failure before May 1 cron. Review." |
   | No auth error, recency is normal | May be genuine low-volume (no games processed in last 24 hours). Hold until May 3 weekend check. |

---

### tgs_athleteone RED (< 5 new games)

1. Confirm TGS scraper is targeting correct endpoints:
   ```bash
   grep -i "endpoint\|url\|target" /path/to/logs/tgs_athleteone.log | tail -5
   ```
2. Check for off-by-one date filter bug — TGS scrapers have historically had issues excluding current-day games:
   ```sql
   SELECT game_date, COUNT(*) AS count
   FROM games
   WHERE source = 'tgs_athleteone'
     AND created_at >= '2026-05-01'
   GROUP BY game_date
   ORDER BY game_date DESC
   LIMIT 10;
   ```
3. | Result | Action |
   |--------|--------|
   | Game dates stop 1 day before today | Likely off-by-one filter bug. File bug report for Presten: "TGS date filter excluding current-day games. Games appear in DB only after 1-day lag. May cause consistent undercount." |
   | Game dates are current | TGS scraper is working but low game volume. Normal if few TGS-sourced ECNL games were played today. Hold until May 3. |

---

### gotsport_html / tgs / tgs_rl RED (< 2 new games each)

1. Confirm these scrapers are in the May 1 cron deployment:
   ```bash
   # Check if these scrapers are included in run-daily.sh
   grep -i "gotsport_html\|tgs_rl" /path/to/run-daily.sh
   ```
2. | Result | Action |
   |--------|--------|
   | Scraper not in cron config | **Not a failure.** These sources may have been excluded from the initial May 1 deployment. Note in briefing: "[source] not in May 1 cron scope — expected zero output." |
   | Scraper in cron but zero output | Check for rate-limit blocks or format changes in scraper log. File issue report. Do not escalate unless data quality check (team match rate, etc.) degrades. |

---

## Section 4: Escalation Thresholds

### FORGE handles WITHOUT Presten

- Aggregate YELLOW with all sources in expected YELLOW range (May 1 weekday pattern)
- Single-source YELLOW when aggregate is GREEN — log and monitor
- gotsport_api YELLOW on May 1 (expected weekday behavior — check again May 3)
- Source not in cron scope returning zero — document, not an error
- Source-specific RED with a clear diagnostic cause (wrong date filter, log evidence of auth error) — file bug report

### FORGE escalates to Presten IMMEDIATELY for

- Cron not running on May 1 (deployment failure)
- Aggregate RED after 48 hours with no clear source-specific cause
- DB write path confirmed broken (Step 3 returns empty result after clean run)
- Auth credential failure on gotsport_api (do not retry without rotation)
- Data corruption: > 100 duplicate game insertions in a single run

**Escalation format — include in next briefing:**

```
ESCALATION — May 1 Pipeline [SEVERITY: CRITICAL / HIGH]
Signal: [aggregate count, specific source affected]
Diagnostic steps completed: [Step 1 / Step 2 / Step 3]
Evidence: [key log line or query output]
Probable cause: [FORGE's assessment]
Action needed from Presten: [specific ask — e.g., "check cron config", "rotate GotSport API credentials"]
```

---

## Section 5: Weekend Re-Check Protocol (May 3–4)

If May 1–2 are **aggregate YELLOW** (50–299 games) but no specific source failure is detected:

1. **Do not escalate to Presten between May 1 and May 3.** Weekday YELLOW is expected.
2. On May 3 (Saturday), re-run the Morning-After Runbook queries.
3. **Expected result:** aggregate ≥ 500 games (weekend volume is materially higher — more games scheduled on weekends).
4. | May 3 result | Action |
   |-------------|--------|
   | May 3 GREEN (≥ 300 games) | May 1–2 YELLOW was normal weekday behavior. No action. Note in briefing. |
   | May 3 also YELLOW (< 300 games) | This is two consecutive low-volume days including a weekend. **Escalate to Presten** with May 1, May 2, and May 3 counts. Something is structurally wrong. |
   | May 3 RED (< 50 games) | **ESCALATE IMMEDIATELY.** Three-day failure pattern. Run Section 2 diagnostic before filing escalation. |

5. File a "May 3–4 Weekend Check" note in the Monday briefing regardless of result, so SENTINEL has the full picture.

---

## Quick Reference: Per-Source Thresholds

| Source | GREEN Floor | YELLOW Range | RED Floor |
|--------|------------|-------------|----------|
| gotsport_api | ≥ 200 games | 50–199 games | < 10 games |
| tgs_athleteone | ≥ 50 games | 10–49 games | < 5 games |
| gotsport_html | ≥ 20 games | 5–19 games | < 2 games |
| tgs | ≥ 20 games | 5–19 games | < 2 games |
| tgs_rl | ≥ 10 games | 3–9 games | < 1 game |
| **Aggregate** | **≥ 300 games** | **50–299 games** | **< 50 games** |

*Source: [[May 1 — Per-Source Game Volume Forecast]] — refer to that document for full threshold rationale.*

---

## References

- [[May 1 — Per-Source Game Volume Forecast]] — per-source thresholds this protocol responds to
- [[Daily Pipeline — Morning-After Runbook]] — the initial diagnostic tool that feeds this protocol
- [[May 1 Deployment — Day-Of Runbook]] — deployment context and cron setup reference
- [[Pipeline Error Classification Taxonomy]] — error codes for classifying scraper exceptions
- [[May 1 First-Run Validation Checklist]] — post-first-run substantive validation (complements this protocol)

*FORGE — 2026-04-27*
