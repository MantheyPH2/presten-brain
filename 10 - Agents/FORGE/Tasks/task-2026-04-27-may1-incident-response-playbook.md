---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/May 1 — Pipeline Incident Response Playbook.md"
---

# Task: May 1 Pipeline Incident Response Playbook

## Context

FORGE has filed an excellent Day-Of Runbook for May 1 and a Morning-After Validation Runbook for May 2. Both assume the pipeline runs and produces data — they tell Presten what to check and how to interpret results.

What neither document covers: what to do when something goes wrong. May 1 is Stage 3 of the crawl expansion — the first time the pipeline will run with the full Stage 1 org-ID set (plus any additional Stage 2 entities if authorized). First runs fail. The question is not whether there will be a failure mode on May 1, but which one occurs and how FORGE responds.

FORGE is on call from April 29 through May 3 (first pipeline week). Without an incident response playbook, FORGE must improvise under time pressure when Presten reports a problem. Improvisation under time pressure produces incomplete diagnoses and missed remediation steps.

This playbook closes that gap. It covers the 5 most likely failure scenarios with specific remediation steps for each.

---

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/May 1 — Pipeline Incident Response Playbook.md`

---

### Header

```
Written by: FORGE
Prepared for: May 1 pipeline first run and May 1–7 monitoring week
On-call period: 2026-04-29 through 2026-05-03
Last updated: [date]
```

---

### Section 1: How to Use This Playbook

Brief orientation:

- This playbook is for Presten to trigger and FORGE to execute.
- Presten identifies a symptom (e.g., "zero games ingested"). Presten reports it to FORGE in the next briefing.
- FORGE identifies the matching scenario, executes the remediation steps, and reports outcome in the same or next briefing.
- If the scenario does not match any playbook entry, FORGE escalates to SENTINEL as an unknown failure.

---

### Section 2: Scenario Matrix

Brief summary table — Presten uses this to identify which scenario applies before reading the full entry.

| # | Symptom | Most Likely Cause | FORGE Action | Time to Resolve |
|---|---------|------------------|-------------|----------------|
| S1 | 0 games ingested, pipeline reported success | Org-ID misconfigured or inactive | Config check + re-run | ~1 hour |
| S2 | 0 games ingested, pipeline reported error | Connection failure or auth failure | Log check + network diagnosis | ~2 hours |
| S3 | Games ingested from some sources, 0 from others | Source-level config issue | Per-source diagnosis | ~1 hour per source |
| S4 | Pipeline crashed mid-run | Code error or memory issue | Log analysis + partial recovery check | ~2–4 hours |
| S5 | Games ingested but volume far below GREEN floor | Partial crawl — bot detection or rate limit | Rate limit analysis + re-run plan | ~3 hours |

---

### Section 3: Scenario Detail — S1: Zero Games, Success Status

**Symptom:** Pipeline run reported "completed successfully" but `SELECT COUNT(*) FROM games WHERE scraped_at > [may1_date]` returns 0.

**Diagnosis steps (FORGE executes in sequence):**
1. Confirm the org-ID(s) in the active config match what is in `Infrastructure/GotSport Org-ID Master Reference — April 2026.md`. A copy-paste error in the numeric org-ID is the most common cause.
2. Hit the GotSport schedule endpoint for the first org-ID manually (browser session, 2 minutes). If it returns games: config is wrong. If it returns nothing: org-ID may be inactive for this season.
3. Check whether the pipeline's crawl loop actually iterated — pipeline may have "succeeded" with an empty org-ID list.

**Remediation:**
- Config error: correct org-ID, re-run pipeline, re-check game count.
- Org-ID inactive: escalate to SENTINEL with specific org-ID and evidence of inactivity.
- Empty crawl loop: diagnose config loading logic, FORGE files a code issue for Presten to fix.

**Escalation trigger:** If S1 diagnosis cannot be completed within 2 hours, escalate to SENTINEL.

---

### Section 4: Scenario Detail — S2: Zero Games, Error Status

**Symptom:** Pipeline run terminated with an error. Log shows connection error, timeout, or authentication failure.

**Diagnosis steps:**
1. Read the pipeline log. Identify the first error line. Is it a connection error (network), HTTP 403 (auth), HTTP 429 (rate limit), or database error?
2. Connection error: confirm GotSport is reachable (Presten browser check, 2 minutes).
3. HTTP 403: GotSport may have changed auth requirements for org-level endpoints. File a SENTINEL queue item — may require code update.
4. HTTP 429: Rate limiting. Note the timestamp — this reveals whether the pipeline hit rate limits immediately (rate floor too aggressive) or after N games (rate floor acceptable, just hitting a session limit).
5. Database error: confirm the DB is accessible and the games table schema is intact.

**Remediation:**
- Network: resolve network issue, re-run.
- Auth change: FORGE diagnoses the new auth requirement and proposes a code fix for Presten.
- Rate limit: adjust rate floor in config, re-run with increased delay.
- DB error: escalate to Presten immediately — do not attempt re-run until DB state is confirmed.

**Escalation trigger:** Any database error escalates immediately.

---

### Section 5: Scenario Detail — S3: Partial Ingestion Across Sources

**Symptom:** Pipeline run completed. Per-source query (from Pipeline Validation Spec) shows some sources GREEN but others at 0 games.

**Diagnosis steps:**
1. Identify which specific org-IDs have 0 games.
2. For each failing org-ID: check GotSport browser directly. If no games visible: org-ID may be off-season or inactive. If games visible: pipeline-specific failure.
3. Check pipeline log for per-org-ID error lines. A missing org-ID in the log means the config loop skipped it.
4. Confirm the failing org-IDs are in the active config with the correct numeric value.

**Remediation:**
- Off-season org-IDs: document as "seasonal — expected 0 games in April/May" and adjust the per-source threshold to reflect this.
- Pipeline-skipped org-IDs: diagnose the config loading issue; file a fix for Presten.
- Browser confirms games but pipeline returns 0: file detailed diagnosis to SENTINEL; likely a CSS selector or endpoint change for that org-ID.

**Escalation trigger:** If ≥ 3 org-IDs fail with no diagnosis, escalate to SENTINEL.

---

### Section 6: Scenario Detail — S4: Pipeline Crash Mid-Run

**Symptom:** Pipeline process terminated unexpectedly. Some games are in the DB, others are not. Log is truncated.

**Diagnosis steps:**
1. Read the last 50 lines of the pipeline log before crash.
2. Identify the last org-ID that was processing when crash occurred.
3. Run: `SELECT COUNT(*) FROM games WHERE scraped_at BETWEEN [run_start] AND [crash_time]` — this tells FORGE how many games landed before the crash.
4. Check system logs for memory errors, process kill signals, or disk full conditions.

**Remediation:**
- Memory crash: identify which org-ID caused the spike. File for Presten to run that org-ID in isolation with logging.
- Process kill (OOM): pipeline may need memory optimization for large org-IDs. Temporary fix: run each failing org-ID in a separate pipeline run.
- Partial data is OK: upsert logic means re-running the same org-IDs will not duplicate games. Advise Presten to re-run from the last successful org-ID.

**Escalation trigger:** If crash cause cannot be identified from logs, escalate to SENTINEL.

---

### Section 7: Scenario Detail — S5: Volume Far Below GREEN Floor

**Symptom:** Games ingested but aggregate count is 10–40% of the GREEN floor from the Per-Source Volume Forecast.

**Diagnosis steps:**
1. Per-source breakdown (query from Validation Spec). Identify which sources are low.
2. For low sources: check if game counts increase if FORGE re-queries 24 hours later. GotSport sometimes delays game visibility.
3. Check pipeline rate floor: if set too aggressively (< 500ms), GotSport may have served empty responses without erroring.
4. Check game_date distribution: are the games present from the right date range (current season) or older?

**Remediation:**
- 24-hour delay: re-check May 2. If volume recovers, no action needed.
- Rate floor too aggressive: adjust to 1000ms, advise Presten to re-run the partial sources.
- Wrong date range: diagnose date filter in the pipeline config; file a fix.

**Escalation trigger:** If volume does not recover by May 3 and no specific cause is identified, escalate to SENTINEL.

---

### Section 8: SENTINEL Escalation Protocol

If any scenario escalates to SENTINEL, FORGE files a queue item with:

```
Priority: urgent
Category: alert
Body:
  Scenario: [S1/S2/S3/S4/S5]
  Symptom: [exact symptom]
  Steps taken: [steps FORGE completed]
  What FORGE could not determine: [specific unknown]
  Recommended action: [FORGE's best guess at next step]
```

FORGE does not delay filing the queue item to continue diagnosing. Escalation and diagnosis can run in parallel.

---

## Quality Standard

- Each scenario must have specific queries or commands, not abstract steps.
- Escalation triggers must be time-bounded — "if not resolved in X hours" not "if unresolvable."
- The remediation for each scenario must be executable by FORGE without needing to invent new approaches on the fly.
- The playbook must not require FORGE to know the pipeline code to use — Presten owns the code; FORGE diagnoses from logs and queries.

## Why This Matters

The Day-Of Runbook and Morning-After Runbook tell Presten what a healthy first run looks like. This playbook tells FORGE what to do when the run is not healthy. Without it, a pipeline failure on May 1 consumes multiple sessions of diagnosis and improvisation. With it, FORGE can diagnose and remediate a common failure within a single session — and SENTINEL gets a clean status report instead of a confusion spiral.

## References

- `Infrastructure/Daily Pipeline — May 1 Day-Of Runbook.md` — the healthy-path companion
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — aggregate threshold companion
- `Infrastructure/May 1 — Per-Source Game Volume Forecast.md` — per-source GREEN floors for S3/S5 diagnosis
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — per-source query patterns
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — active org-ID config reference
