---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/May 1 — Pipeline Red Signal Response Protocol.md"
---

# Task: May 1 — Pipeline Red Signal Response Protocol

## Context

FORGE filed the May 1 Per-Source Game Volume Forecast this session, defining per-source GREEN/YELLOW/RED thresholds and a diagnostic tree for YELLOW aggregate signals. The diagnostic tree tells FORGE how to diagnose the source of low volume. What it does not specify is: **after FORGE diagnoses the problem, what are the decision rules for escalation, pause, or rollback?**

The May 1 first production run is the most uncertain pipeline event in the plan. The pipeline code has been specified and tested in limited scope, but it has never run as a daily automated cron. Failure modes range from benign (low-volume weekday) to critical (cron not running, scraper exception swallowed, DB write path broken). FORGE needs a protocol that distinguishes these cases and tells FORGE — without requiring Presten's time — what to do for each.

This is the companion to the Per-Source Forecast: the forecast defines the signals, this document defines the responses.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/May 1 — Pipeline Red Signal Response Protocol.md`

---

### Section 1: Signal Classification Matrix

FORGE enters this matrix with the aggregate and per-source counts from the Morning-After Runbook.

| Signal | Aggregate | Per-Source Pattern | Classification | Response |
|--------|-----------|-------------------|---------------|----------|
| Full GREEN | ≥ 300 games | All sources ≥ GREEN floor | Healthy | No action. Log and continue. |
| Aggregate YELLOW, all sources YELLOW | 50–299 games | Each source in YELLOW range | Low-volume weekday | Log, note as expected for weekday. Re-check May 3–4 weekend run before escalating. |
| Aggregate YELLOW, one source RED | 50–299 games | 1 source below RED floor | Source-specific issue | Investigate the RED source. Do not pause pipeline. File issue report. |
| Aggregate RED | < 50 games | N/A | Pipeline not running OR systemic failure | Execute Section 2 diagnostic. Do NOT wait for weekend data. |
| Aggregate GREEN, one source flat zero | ≥ 300 games | 1 source = 0 | Source failure masked by aggregate | Investigate the zero source. File issue report within 24 hours. |

---

### Section 2: Aggregate RED Diagnostic Tree

If aggregate < 50 games after 48 hours (i.e., May 1 and May 2 both RED):

**Step 1: Confirm cron is running**
- Check cron log or job runner output for May 1 execution record
- If no execution record: cron is not running. This is a deployment failure, not a data failure. Escalate to Presten immediately with: "Daily pipeline cron did not execute on May 1. Deployment issue. Action required."
- If execution record exists: proceed to Step 2.

**Step 2: Confirm pipeline completed without exception**
- Check pipeline exit code / error log for May 1 run
- If exception in scraper or DB write: note the specific exception. File issue report with exception message. If exception is in a source-specific scraper, that source may be failing while others succeed — check per-source counts.
- If no exception: proceed to Step 3.

**Step 3: Confirm games are reaching the DB**
```sql
-- Check if any games were written on May 1
SELECT source, COUNT(*) AS games_written
FROM games
WHERE ingested_at >= '2026-05-01'
  AND ingested_at < '2026-05-02'
GROUP BY source;
```
- If result is empty: games are not reaching the DB despite a clean pipeline run. This is a write-path failure. Escalate to Presten with: "Pipeline ran without exception but no games written to DB on May 1. DB write path may be broken."
- If result has rows but aggregate is still < 50: scraper is producing low output. Investigate per-source with the source-specific diagnostic queries from the Per-Source Forecast document.

---

### Section 3: Source-Specific RED Response

For each source that hits its individual RED threshold, FORGE executes:

**gotsport_api RED (< 10 new games):**
1. Confirm GotSport API credentials are valid (check for 401/403 responses in scraper log)
2. Check if GotSport had a known outage (no automated check available — FORGE notes the possibility)
3. If credentials issue: escalate to Presten. Do not retry with stale credentials.
4. If no auth issue: check last successful run date. If > 3 days ago, flag for Presten review.

**tgs_athleteone RED (< 5 new games):**
1. Confirm TGS scraper is targeting correct endpoints
2. Check if `game_date` filter in TGS scraper is excluding current-day games (off-by-one date bug)
3. If filter issue: FORGE files a bug report; Presten deploys fix.

**gotsport_html / tgs / tgs_rl RED (< 2 new games each):**
1. Confirm these scrapers are included in the May 1 cron (they may have been excluded from the initial deployment scope)
2. If excluded: not a failure — note which scrapers are in-scope for May 1 vs. later activations
3. If included but zero output: check for format changes or rate-limit blocks in scraper log

---

### Section 4: Escalation Thresholds — What Requires Presten Action

FORGE handles the following WITHOUT Presten:
- Aggregate YELLOW with all sources in expected YELLOW range (weekday volume) — log and monitor
- Single-source YELLOW when aggregate is GREEN — log and file issue report
- Source-specific RED with clear diagnostic (wrong credentials, format change) — file bug report for Presten to deploy fix

FORGE escalates to Presten IMMEDIATELY for:
- Cron not running on May 1 — pipeline deployment failure
- Aggregate RED after 48 hours with no clear source-specific cause — systemic issue
- DB write path confirmed broken — architecture issue
- Any exception involving data corruption or duplicate game insertion at scale (> 100 duplicates)

**Escalation format:**
When escalating, FORGE files in the next briefing:
```
ESCALATION — May 1 Pipeline [SEVERITY]
Signal: [description]
Diagnostic step completed: [which steps from Section 2]
Probable cause: [FORGE's assessment]
Action needed from Presten: [specific ask]
```

---

### Section 5: Weekend Re-Check Protocol (May 3–4)

If May 1–2 are YELLOW aggregate but no specific failure is detected:

1. Do not escalate to Presten between May 1 and May 3.
2. On May 3 (Saturday), re-run the same Morning-After Runbook queries.
3. Expected: aggregate ≥ 500 games (weekends have materially higher volume than weekdays).
4. If May 3 is also YELLOW (< 300 games): escalate to Presten with weekday + weekend data.
5. If May 3 is GREEN: May 1–2 YELLOW was expected weekday behavior. No action.

---

## Why This Matters

The Per-Source Forecast tells FORGE what numbers to expect. This protocol tells FORGE what to do when the numbers are wrong. Without it, FORGE faces a real-time decision under ambiguity on May 1: is a 200-game aggregate an emergency or a normal weekday? The protocol answers that question in advance, prevents over-escalation (Presten spends time on a weekday dip), and prevents under-escalation (a cron failure treated as low volume). SENTINEL's job is to make sure FORGE has everything needed to run the first pipeline week without requiring Presten's attention except for genuine failures.

## Definition of Done

- File at deliverable path
- Section 1 signal classification matrix covers all main scenarios
- Section 2 diagnostic tree is actionable without additional design — FORGE follows it linearly
- Section 3 specifies responses for each source-specific RED
- Section 4 clearly draws the FORGE-handles vs. escalate-to-Presten line
- Section 5 covers the weekday/weekend volume ambiguity
- FORGE briefing note: "May 1 Red Signal Response Protocol filed. Escalation threshold: [aggregate RED after 48 hrs + no source-specific cause]. Weekend re-check: May 3."

## References

- `Infrastructure/May 1 — Per-Source Game Volume Forecast.md` — defines signal thresholds this protocol responds to
- `Infrastructure/Daily Pipeline — Morning-After Runbook.md` — the initial diagnostic tool this protocol extends
- `Infrastructure/May 1 — Day-Of Runbook.md` — deployment context for May 1 execution
