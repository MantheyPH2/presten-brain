---
type: agent-task
assigned_by: SENTINEL
assigned_to: FORGE
date: 2026-04-29
priority: high
due: "2026-05-01 within 2 hours of pipeline run"
status: pending
topic: may1-pipeline-launch-day-monitoring
---

# Task: May 1 Pipeline Launch Day — Real-Time Monitoring Brief

## Objective

When the May 1 pipeline runs, FORGE monitors all Stage 1 sources in real-time and files a post-run monitoring brief within 2 hours of completion. SENTINEL uses this brief to determine if Stage 2 conditions are met and to update the Stage 2 Trigger Document Section 1.

## Background

The pipeline is scheduled to run May 1. FORGE has filed:
- `Infrastructure/May 1 — Official Source Baseline.md` (forecast volumes per source)
- `Infrastructure/Pipeline Week 1 Monitoring Log.md` (pre-populated with thresholds)

This task operationalizes both documents into a real-time monitoring action when the pipeline actually runs.

## During the Pipeline Run

Monitor the following per source against the Official Source Baseline forecasts:

| Source | Expected Range | Flag if |
|--------|---------------|---------|
| gotsport_api | 300–1,200 new games | < 100 or > 1,500 |
| tgs_athleteone | 80–300 new games | < 30 or > 400 |
| tgs | 20–80 new games | < 10 or > 150 |
| tgs_rl | 20–80 new games | < 5 or > 150 |
| gotsport_html | 10–50 new games | < 5 or > 100 |
| TYSA | Per org-ID (if active) | Any zero-result run |

Note any sources that error, timeout, or return zero results — these require immediate SENTINEL notification, not just briefing mention.

## Post-Run Deliverable

File `Infrastructure/May 1 Pipeline Launch — Post-Run Actuals.md` within 2 hours of pipeline completion. Structure:

### Section 1 — Actual Counts vs Forecast

| Source | Forecast Range | Actual Count | Status |
|--------|---------------|--------------|--------|
| gotsport_api | 300–1,200 | [FILL] | GREEN / YELLOW / RED |
| ... | ... | ... | ... |

### Section 2 — Anomalies and Flags

List any sources that fell outside forecast range with a one-sentence hypothesis for why.

### Section 3 — SENTINEL Recommendation

One of:
- **GO for Stage 2 assessment** — all sources GREEN
- **CONDITIONAL** — some sources YELLOW, explain which
- **HOLD** — any source RED or errored, explain

### Section 4 — ELO Handoff Signal

One sentence FORGE sends to ELO confirming pipeline ran and ratings updates are available for ELO's May 2 stability check.

## Escalation

If **any** source errors or returns zero games: SENTINEL notification in briefing immediately (do not wait 2 hours). The Stage 2 trigger timeline depends on May 1 being a clean run.

## Notes

- If the pipeline does not run May 1 due to Presten session timing, FORGE files a one-paragraph "launch deferred" note and notifies SENTINEL.
- This task does NOT require FORGE to fix pipeline failures on May 1 — only to report them clearly.
