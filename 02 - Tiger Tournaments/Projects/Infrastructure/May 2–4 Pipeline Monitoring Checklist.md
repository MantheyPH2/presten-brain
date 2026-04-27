---
title: May 2–4 Pipeline Monitoring Checklist
tags: [infrastructure, pipeline, monitoring, may1, weekly-check, forge]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: ready — activate May 3 if Day 1 (May 2 Morning-After) shows YELLOW or requires continued monitoring
task: task-2026-04-27-may2-may4-pipeline-monitoring-checklist
---

# May 2–4 Pipeline Monitoring Checklist

> **How to use:** Open this document after completing the May 2 Morning-After Runbook check. If that check shows GREEN with no concerns, proceed to Day 4 (May 5) for the first weekly summary. If it shows YELLOW or any flag, work through Days 2–4 here in sequence.

---

## Pre-Checklist: Fill Before Each Day's Check

```
Day being checked:                     [ ] May 3 (Day 2)  [ ] May 4 (Day 3)  [ ] May 5 (Day 4)
Aggregate game count from May 1 run:   ___________
May 2 Morning-After result:            GREEN / YELLOW / RED
Any active flags from prior day:       ___________
psql session open:                     [ ]
```

---

## Section 1: Day 2 Check — May 3 (Weekend Re-Check)

> **When to run:** Morning of May 3 (Saturday). This is the primary re-check for any YELLOW signal from May 2. Weekend volume is materially higher than Thursday (May 1); a May 1–2 YELLOW that recovers on May 3 was a normal weekday dip.

### May 3 Queries

**Q3-1: Aggregate games added since pipeline launch**
```sql
SELECT
  DATE(created_at) AS run_date,
  COUNT(*) AS games_added
FROM games
WHERE created_at >= '2026-05-01'
GROUP BY DATE(created_at)
ORDER BY run_date;
```
*Record:* games added per day since May 1. Compare against Per-Source Forecast Section 3 thresholds.

**Q3-2: Per-source delta May 1 → May 3**
```sql
SELECT
  source,
  COUNT(*) AS total_new_games,
  MAX(created_at) AS most_recent_ingest
FROM games
WHERE created_at >= '2026-05-01'
GROUP BY source
ORDER BY total_new_games DESC;
```
*Record:* per-source running total since May 1 launch. Compare each source against Section 2 thresholds in `Infrastructure/May 1 — Per-Source Game Volume Forecast.md`.

**Q3-3: New duplicates since May 1**
```sql
SELECT game_date, home_team_id, away_team_id, event_id, COUNT(*) AS instances
FROM games
WHERE created_at >= '2026-05-01'
GROUP BY game_date, home_team_id, away_team_id, event_id
HAVING COUNT(*) > 1
LIMIT 20;
```
*Record:* count of rows returned. Expected: 0. Any result requires investigation before Stage 2 is authorized.

**Q3-4: Date distribution of new games**
```sql
SELECT
  game_date,
  COUNT(*) AS new_games
FROM games
WHERE created_at >= '2026-05-01'
  AND game_date >= '2025-08-01'
GROUP BY game_date
ORDER BY game_date DESC
LIMIT 30;
```
*Record:* are new games landing in the current season window (fall 2025 – spring 2026)? Flag if the majority of new games have `game_date` older than 2 years — this suggests the crawl is pulling historical data only, not current events.

### May 3 Pass/Fail Assessment

| Check | Threshold | Actual | Status |
|-------|-----------|--------|--------|
| Aggregate games (2-day total) | GREEN: ≥ 300 / YELLOW: 50–299 / RED: < 50 | | |
| Any source individually RED (0 new games) | FLAG if any source = 0 after 2 days | | |
| Duplicate rows | 0 | | |
| New games in 2025–26 season window | Yes (> 50% of new games in current season) | | |

**May 3 Verdict:**
- **GREEN (≥ 300 aggregate, no source RED, 0 duplicates):** Normal weekday-into-weekend behavior confirmed. No escalation. Continue to Day 3 check on May 4.
- **YELLOW (50–299 aggregate, or 1 source at 0 new games):** Note specific source(s) lagging. Escalate to SENTINEL with 2-day comparison table. Do not wait for May 4.
- **RED (< 50 aggregate, or any multiple-source failure):** Immediate escalation per `Infrastructure/May 1 — Pipeline Red Signal Response Protocol.md` Section 2 aggregate RED diagnostic.

---

## Section 2: Day 3 Check — May 4 (First Weekend Run Complete)

> **When to run:** Morning of May 4 (Sunday). This is the first full weekend data point — highest expected volume of the first week. A healthy pipeline should show materially more games on May 4 than May 3.

### May 4 Queries

**Q4-1: Running aggregate since May 1**
```sql
SELECT
  DATE(created_at) AS run_date,
  COUNT(*) AS daily_new_games,
  SUM(COUNT(*)) OVER (ORDER BY DATE(created_at)) AS running_total
FROM games
WHERE created_at >= '2026-05-01'
GROUP BY DATE(created_at)
ORDER BY run_date;
```
*Record:* per-day new games and running total. Compare running total against Per-Source Forecast Section 3 thresholds.

**Q4-2: Per-source forecast vs. actual (3-day comparison)**
```sql
SELECT
  source,
  COUNT(*) AS games_since_may1,
  MAX(created_at) AS last_ingest_time,
  MIN(created_at) AS first_ingest_time
FROM games
WHERE created_at >= '2026-05-01'
GROUP BY source
ORDER BY games_since_may1 DESC;
```
*Record:* fill the May 4 column in the Section 4 comparison table below.

**Q4-3: gotsport_api weekend volume confirmation**
```sql
SELECT
  DATE(game_date) AS game_day,
  COUNT(*) AS games
FROM games
WHERE source = 'gotsport_api'
  AND created_at >= '2026-05-03'
  AND created_at < '2026-05-05'
GROUP BY DATE(game_date)
ORDER BY game_day;
```
*Record:* weekend (May 3–4) game ingestion for gotsport_api specifically. Expected: higher than May 1 Thursday volume. If May 3–4 gotsport_api count ≤ May 1 count, GotSport weekend events may not be flowing — flag for investigation.

**Q4-4: ecnl_verified write-time check (if fix deployed)**
```sql
SELECT
  COUNT(*) AS ecnl_games_since_may1,
  SUM(CASE WHEN ecnl_verified IS TRUE THEN 1 ELSE 0 END) AS verified_count,
  ROUND(100.0 * SUM(CASE WHEN ecnl_verified IS TRUE THEN 1 ELSE 0 END) / NULLIF(COUNT(*), 0), 1) AS pct_verified
FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier IN ('ecnl', 'ecnl_rl')
  AND created_at >= '2026-05-01';
```
*Record:* pct_verified for post-May-1 ECNL games. Expected: 100% if FM2 write-time fix was deployed. If 0%: FM2 fix was not deployed — flag to SENTINEL; supplemental backfill needed for May 1–4 ECNL records.
*(Run this only if the ecnl_verified write-time fix has been deployed. If fix not yet deployed, note: "FM2 fix pending — ecnl_verified write-time check not applicable yet.")*

### May 4 Pass/Fail Assessment

| Check | Threshold | Actual | Status |
|-------|-----------|--------|--------|
| 3-day aggregate total | GREEN: ≥ 500 / YELLOW: 200–499 / RED: < 200 | | |
| gotsport_api weekend volume | ≥ 200 new games May 3–4 combined | | |
| ecnl_verified rate (if fix deployed) | 100% | | |
| Any source: 0 games over 3 days | FLAG if any source = 0 | | |

**May 4 Verdict:**
- **GREEN (≥ 500 aggregate, gotsport_api weekend confirmed, no source at 0):** Pipeline healthy through first weekend. File first weekly health summary on May 5.
- **YELLOW (200–499 aggregate, or gotsport_api weekend low):** File escalation with 3-day trend table. If specific source lagging: include source-level breakdown. SENTINEL reviews before May 5.
- **RED (< 200 aggregate, or any source RED for 3 consecutive days):** Escalate immediately per Red Signal Protocol Section 2. Do not wait for May 5.

---

## Section 3: Day 4 Check — May 5 (4-Day Trend Complete)

> **When to run:** Morning of May 5 (Monday). Four data points are now available. This is the threshold for FORGE's first weekly health review filing.

### May 5 Queries

**Q5-1: Full 4-day trend**
```sql
SELECT
  DATE(created_at) AS run_date,
  COUNT(*) AS daily_games,
  SUM(COUNT(*)) OVER (ORDER BY DATE(created_at)) AS running_4day_total
FROM games
WHERE created_at >= '2026-05-01'
  AND created_at < '2026-05-06'
GROUP BY DATE(created_at)
ORDER BY run_date;
```
*Record:* fill the daily-games column in the Section 4 trend table below.

**Q5-2: Per-source 4-day totals**
```sql
SELECT
  source,
  COUNT(*) AS games_may1_may4
FROM games
WHERE created_at >= '2026-05-01'
  AND created_at < '2026-05-06'
GROUP BY source
ORDER BY games_may1_may4 DESC;
```
*Record:* fill the 4-day actual column in Section 4 comparison table.

**Q5-3: DLQ review (4-day aggregate)**
```sql
-- Adjust table/column names to match actual DLQ schema
SELECT error_type, COUNT(*) AS error_count
FROM dead_letter_queue
WHERE created_at >= '2026-05-01'
GROUP BY error_type
ORDER BY error_count DESC;
```
*Record:* error type and count. Classify each error type using `Infrastructure/Pipeline Error Classification Taxonomy.md`. Note any repeating error patterns that indicate a systemic scraper issue.

### May 5 Pass/Fail Assessment

| Metric | Threshold | Actual | Status |
|--------|-----------|--------|--------|
| 4-day aggregate (May 1–4) | GREEN: ≥ 700 / YELLOW: 300–699 / RED: < 300 | | |
| gotsport_api 4-day total | ≥ 600 games | | |
| tgs_athleteone 4-day total | ≥ 100 games | | |
| DLQ error count | 0–15 (log); 16+ (escalate per taxonomy) | | |

**May 5 Verdict:**
- **GREEN (4-day ≥ 700, all sources non-zero, DLQ < 16):** Pipeline viable for DSS-readiness claim. File first weekly health review.
- **YELLOW (300–699 4-day, or 1 source lagging):** File escalation package per Section 5. Flag for SENTINEL that DSS-readiness claim needs clarification.
- **RED (< 300 4-day, or systemic error pattern in DLQ):** Escalate immediately. DSS-readiness claim cannot be made. SENTINEL decision required.

---

## Section 4: Comparison Table (Fill As Each Day Completes)

Use this table across all 4 checks. Pre-populated with forecast figures from `Infrastructure/May 1 — Per-Source Game Volume Forecast.md`.

### Daily New Games

| Date | gotsport_api | tgs_athleteone | tgs | tgs_rl | gotsport_html | Daily Total |
|------|-------------|----------------|-----|--------|---------------|-------------|
| May 1 (launch day) | | | | | | |
| May 2 | | | | | | |
| May 3 | | | | | | |
| May 4 | | | | | | |
| **4-Day Total** | | | | | | |
| **Per-Source Forecast (7-day)** | ~2,100–8,400 | ~560–2,100 | ~140–560 | ~140–560 | ~70–350 | ~700/day (est.) |
| **4-Day Estimate (57% of 7-day)** | ~1,200–4,800 | ~320–1,200 | ~80–320 | ~80–320 | ~40–200 | ~400 (est.) |
| **Delta (Actual vs. 4-Day Est.)** | | | | | | |

*The 57% estimate reflects 4 days including 1 Thursday, 1 Friday, 1 Saturday, 1 Sunday — roughly 55–60% of weekly game volume given weekend density.*

---

## Section 5: Escalation Package Format

When FORGE escalates to SENTINEL during May 2–5, include:

```
ESCALATION — May [N] Pipeline [YELLOW / RED]

Signal:                [description of what triggered]
Day triggered:         May [N]
Days with this signal: [list dates]

3-day trend:
  May 1: [X] games | May 2: [X] games | May 3: [X] games (if available)

Per-source breakdown:
  gotsport_api:       [X] games — [GREEN / YELLOW / RED per May 1 Forecast]
  tgs_athleteone:     [X] games — [GREEN / YELLOW / RED]
  tgs:                [X] games — [GREEN / YELLOW / RED]
  tgs_rl:             [X] games — [GREEN / YELLOW / RED]
  gotsport_html:      [X] games — [GREEN / YELLOW / RED]

FORGE diagnosis:      [probable cause, or "no diagnosis yet"]
Diagnostic steps run: [which steps from Red Signal Protocol Section 2 were completed]
DLQ count:            [N errors, classified as: X]

FORGE recommendation:
  [ ] Escalate to Presten immediately — cron/deployment issue
  [ ] Hold — re-check May [N+1]; weekday/weekend pattern may explain YELLOW
  [ ] Resolved — [explain what was diagnosed and fixed]
```

---

## References

- `Infrastructure/May 1 — Day-Of Runbook.md` — May 1 launch execution
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — May 2 Day 1 check (precedes this checklist)
- `Infrastructure/May 1 — Pipeline Red Signal Response Protocol.md` — escalation decision tree
- `Infrastructure/May 1 — Per-Source Game Volume Forecast.md` — forecast thresholds used in Section 4
- `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md` — where 4-day actuals are filed for the record

*FORGE — 2026-04-27*
