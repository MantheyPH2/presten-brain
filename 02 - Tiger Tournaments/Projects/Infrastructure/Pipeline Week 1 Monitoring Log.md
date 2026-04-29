---
type: pipeline-monitoring-log
coverage_period: 2026-05-01 to 2026-05-07
pipeline_stage: Stage 1
status: in-progress
sentinel_review_due: 2026-05-07
created: 2026-04-29
author: FORGE
task: task-2026-04-29-post-may1-pipeline-week1-monitoring-log-template
---

# Pipeline Week 1 Monitoring Log — May 1–7

> [!info] Live Document
> FORGE or Presten fills data cells after each pipeline run. All date rows and source names are pre-populated. All baseline thresholds are drawn from `Infrastructure/May 1 — Per-Source Game Volume Forecast.md`. Anomaly log is blank — fill as issues arise.

**Baseline thresholds source:** `Infrastructure/May 1 — Per-Source Game Volume Forecast.md`
**Monitoring queries:** `Infrastructure/May 1 Pipeline Monitoring Queries.md`
**GREEN/YELLOW/RED criteria:** `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md`

---

## Daily Log Table

Pre-populated with dates. Data columns filled post-run.

**Run Status values:** SUCCESS / PARTIAL / FAILED / DID-NOT-RUN

| Date | Run Status | Total Games Ingested | New Teams Added | Errors/Warnings | Sources Active | Notes |
|------|-----------|---------------------|-----------------|-----------------|---------------|-------|
| May 1 (Thu) | — | — | — | — | — | First production run — manual launch per Day-Of Runbook. Weekday: expect lower volume than weekends. |
| May 2 (Fri) | — | — | — | — | — | First automated cron run (2 AM) |
| May 3 (Sat) | — | — | — | — | — | Weekend — expect higher gotsport_api volume |
| May 4 (Sun) | — | — | — | — | — | Weekend |
| May 5 (Mon) | — | — | — | — | — | Stage 2 authorization window opens if GREEN |
| May 6 (Tue) | — | — | — | — | — | |
| May 7 (Wed) | — | — | — | — | — | End of first-week window — complete Summary section |

---

## Per-Source Game Volume Log

Pre-populated with all Stage 1 active sources and org-ID notes. Fill game volumes after each run.

**Query (run after each pipeline run):**
```sql
SELECT source, COUNT(*) AS new_games
FROM games
WHERE created_at > NOW() - INTERVAL '24 hours'
GROUP BY source ORDER BY new_games DESC;
```

| Source | Org-ID / Notes | May 1 Games | May 2 Games | May 3 Games | May 4 Games | May 5 Games | May 6 Games | May 7 Games |
|--------|----------------|-------------|-------------|-------------|-------------|-------------|-------------|-------------|
| gotsport_api | Stage 1 TX org-IDs (TYSA pending — see note) | — | — | — | — | — | — | — |
| tgs_athleteone | ECNL Girls + Boys direct feed | — | — | — | — | — | — | — |
| tgs | TGS main scraper (legacy ECNL records) | — | — | — | — | — | — | — |
| tgs_rl | ECNL RL via TGS feed | — | — | — | — | — | — | — |
| gotsport_html | GotSport HTML scraper (supplementary) | — | — | — | — | — | — | — |
| **TYSA (gotsport_api)** | **Pending org-ID — add row if confirmed before May 1** | — | — | — | — | — | — | — |

**TYSA note:** If TYSA org-ID is confirmed and staged before May 1, expect a one-time first-run discovery volume of 5,000–25,000 games (from `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md`). This will not repeat on May 2+ at that scale. Mark as "Stage 1 TX first-run: [N] games" in the May 1 Notes column.

---

## Week 1 Baseline Thresholds

Pre-populated from `Infrastructure/May 1 — Per-Source Game Volume Forecast.md`. Use to flag anomalies daily.

### Per-Source Thresholds

| Source | Expected Range (per run) | GREEN | YELLOW (investigate) | RED (likely failed) |
|--------|--------------------------|-------|---------------------|---------------------|
| gotsport_api | 300–1,200 games/day | ≥ 200 | 10–199 | 0–9 |
| tgs_athleteone | 80–300 games/day | ≥ 50 | 5–49 | 0–4 |
| tgs | 20–80 games/day | ≥ 10 | 1–9 | 0 |
| tgs_rl | 20–80 games/day | ≥ 10 | 1–9 | 0 |
| gotsport_html | 10–50 games/day | ≥ 5 | 1–4 | 0 |

**Weekend adjustment:** gotsport_api GREEN floor may be lower on weekdays (May 1 = Thursday). Do not flag as RED on May 1 if count is 50–199 — re-check against May 3–4 weekend runs before escalating.

### Aggregate Thresholds

| Metric | Expected | GREEN Floor | RED Ceiling |
|--------|----------|-------------|-------------|
| Total new games / day (excluding TYSA first-run) | 700–1,700 | ≥ 300 | < 50 |
| Total new games / week (May 1–7, excluding TYSA first-run) | 4,900–11,900 | ≥ 2,100 | — |
| Pipeline error rate (per run) | < 5% | < 5% | ≥ 15% |
| New teams added (first run) | Varies | Any new teams | 0 with 0 new games = RED |

**Aggregate RED interpretation:** If total new games < 50 after 48 hours of pipeline operation, the pipeline is almost certainly not running. A fully operational pipeline during April–May spring season should produce hundreds of new game records within hours of first launch.

---

## Anomaly Log

Blank — fill when any pipeline run deviates from baseline thresholds.

| Date | Anomaly Description | Source Affected | Severity (1=low, 3=critical) | Investigation Status | Resolution |
|------|--------------------|-----------------|-----------------------------|---------------------|------------|
| | | | | | |

**Severity guide:**
- 1 — minor anomaly, pipeline functional, no data loss
- 2 — partial failure, some games missed, source dark, recoverable
- 3 — pipeline crash, data integrity issue, or source dark ≥ 2 consecutive days

Any Severity 3 item: flag to SENTINEL same day. Do not wait for next briefing.

**Diagnostic tree for aggregate YELLOW (50–299 games):** See `Infrastructure/May 1 — Per-Source Game Volume Forecast.md` Section 4.

---

## Week 1 Summary

> FORGE completes May 7 after final run data is received.

- **Total games ingested May 1–7:** __
- **Sources performing at GREEN or better:** __ / 5 (excluding TYSA if not yet active)
- **Sources in YELLOW this week:** __ (list: ___)
- **Sources in RED this week:** __ (list: ___)
- **Open anomalies (unresolved):** __
- **FORGE Stage 2 authorization recommendation:** YES / NO / CONDITIONAL
- **SENTINEL review request:** YES / NO

**FORGE authorization signal feeds into:**
`Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md` Section 6 and
`Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` Section 1

---

## References

- `Infrastructure/May 1 — Per-Source Game Volume Forecast.md` — baseline threshold source
- `Infrastructure/May 1 Pipeline Monitoring Queries.md` — SQL queries for data collection
- `Infrastructure/Stage 1 Config Final Review.md` — active source list and config confirmation
- `Infrastructure/May 1 Stage 1 — Launch Confidence Brief.md` — launch context and confidence summary
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — GREEN/YELLOW/RED criteria
- `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md` — companion detailed log
- `Infrastructure/Stage 2 Authorization — Week 1 Monitoring Rubric.md` — SENTINEL's Stage 2 authorization thresholds

*FORGE — 2026-04-29*
