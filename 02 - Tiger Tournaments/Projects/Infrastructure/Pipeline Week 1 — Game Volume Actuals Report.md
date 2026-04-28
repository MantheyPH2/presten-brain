---
title: Pipeline Week 1 — Game Volume Actuals Report
tags: [infrastructure, pipeline, monitoring, week1, may1, gotsport, tysa, forge, sentinel]
created: 2026-04-28
updated: 2026-04-28
author: FORGE
status: template-filed — fill Section 2 daily May 1–7; fill Section 3 by May 7 EOD
task: task-2026-04-28-pipeline-week1-game-volume-actuals-template
---

# Pipeline Week 1 — Game Volume Actuals Report

> [!info] Purpose
> Authoritative record of per-source game volumes for the first week of daily pipeline operation (May 1–7). FORGE fills Section 2 daily within 2 hours of each run completing. Section 3 summarizes by May 7 EOD. Section 5 feeds the Stage 2 authorization decision. SENTINEL and Presten read this document — not the monitoring log — to assess Week 1 health at a glance.

```
Date range:     May 1–7, 2026
Prepared by:    FORGE
Template filed: 2026-04-28 (today)
Filled:         Daily May 1–7
Final summary:  May 7 EOD
```

---

## Section 1: Forecast Baseline

Forecast values drawn from `Infrastructure/May 1 — Per-Source Game Volume Forecast.md` and `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md`.

> [!note] TYSA weekly volume interpretation
> TYSA Run 1 (May 1) captures historical games plus current-season games — a discovery crawl. Volume is expected to be 5,000–25,000 on Day 1 alone. Days 2–7 add only games played since the previous run (incremental), estimated at 50–300/day during spring season. The 3,500–7,000 weekly range below represents a conservative forward-only volume estimate; total week 1 with discovery will be substantially higher.

| Source | Stage | Forecasted Weekly Volume | GREEN Floor | YELLOW Floor | RED Threshold |
|--------|-------|------------------------|-------------|--------------|---------------|
| TYSA (TX) | Stage 1 | 3,500–7,000 games (incremental); Run 1 discovery adds 5,000–25,000 one-time | ≥ 5,000 total Run 1 | 2,000–4,999 Run 1 | < 2,000 Run 1 |
| gotsport_api (existing orgs) | Existing | 2,100–8,400 games/week (300–1,200/run × 7 runs) | ≥ 1,400/wk (≥ 200/run) | 70–1,399/wk (10–199/run) | < 70/wk (< 10/run) |
| tgs_athleteone | Existing | 560–2,100 games/week (80–300/run) | ≥ 350/wk (≥ 50/run) | 35–349/wk (5–49/run) | < 35/wk (< 5/run) |
| tgs | Existing | 140–560 games/week (20–80/run) | ≥ 70/wk (≥ 10/run) | 7–69/wk (1–9/run) | < 7/wk (0/run) |
| tgs_rl | Existing | 140–560 games/week (20–80/run) | ≥ 70/wk (≥ 10/run) | 7–69/wk (1–9/run) | < 7/wk (0/run) |
| gotsport_html | Existing | 70–350 games/week (10–50/run) | ≥ 35/wk (≥ 5/run) | 3–34/wk (1–4/run) | < 3/wk (0/run) |

**Aggregate GREEN floor (all sources, week 1):** ≥ 5,000 from TYSA Run 1 + ≥ 1,400 from existing sources = **≥ 6,400 total**  
**Aggregate RED threshold:** Total new games < 2,000 after full week = pipeline likely impaired.

> [!note] May 1 is a Thursday (weekday)
> Weekday volumes are lower than weekend volumes. Do not flag gotsport_api as RED on May 1 if count is 50–199 — compare against the May 3 (Saturday) run before escalating. Reference: `Infrastructure/May 1 — Per-Source Game Volume Forecast.md` Section 2.

---

## Section 2: Daily Run Log (May 1–7)

FORGE fills within 2 hours of each daily pipeline run completing. Pull counts from the per-source monitoring query in `Infrastructure/May 1 Pipeline Monitoring Queries.md`.

| Date | Run # | Source | Games Ingested | Cumulative | Parse Errors | Status |
|------|--------|--------|----------------|-----------|--------------|--------|
| May 1 | 1 | TYSA | | | | |
| May 1 | 1 | gotsport_api (existing) | | | | |
| May 1 | 1 | tgs_athleteone | | | | |
| May 1 | 1 | tgs | | | | |
| May 1 | 1 | tgs_rl | | | | |
| May 1 | 1 | gotsport_html | | | | |
| May 2 | 2 | TYSA | | | | |
| May 2 | 2 | gotsport_api (existing) | | | | |
| May 2 | 2 | tgs_athleteone | | | | |
| May 2 | 2 | tgs | | | | |
| May 2 | 2 | tgs_rl | | | | |
| May 2 | 2 | gotsport_html | | | | |
| May 3 | 3 | TYSA | | | | |
| May 3 | 3 | gotsport_api (existing) | | | | |
| May 3 | 3 | tgs_athleteone | | | | |
| May 4 | 4 | TYSA | | | | |
| May 4 | 4 | gotsport_api (existing) | | | | |
| May 5 | 5 | TYSA | | | | |
| May 5 | 5 | gotsport_api (existing) | | | | |
| May 6 | 6 | TYSA | | | | |
| May 6 | 6 | gotsport_api (existing) | | | | |
| May 7 | 7 | TYSA | | | | |
| May 7 | 7 | gotsport_api (existing) | | | | |

**Status codes:**
- **GREEN** — volume at or above GREEN floor per Section 1
- **YELLOW** — volume between YELLOW and GREEN floor (investigate; do not escalate unless persists ≥ 2 runs)
- **RED** — volume below RED threshold (escalate to SENTINEL immediately; file queue item same day)
- **ERROR** — pipeline failed to complete run (document in Section 4; file SENTINEL queue item immediately)

**Per-run monitoring SQL** (run after each pipeline execution):
```sql
SELECT source, COUNT(*) AS games_ingested
FROM games
WHERE created_at > NOW() - INTERVAL '24 hours'
GROUP BY source
ORDER BY games_ingested DESC;
```

---

## Section 3: Week 1 Summary (Fill by May 7 EOD)

| Metric | Value | Status |
|--------|-------|--------|
| Total TYSA games ingested (May 1–7) | | GREEN / YELLOW / RED |
| Total games from all sources (May 1–7) | | |
| Daily run success rate (7 of 7 = 100%) | | |
| Parse error rate across all runs (target < 5%) | | |
| Largest single-day volume (TYSA, Run 1 expected) | | |
| Smallest single-day volume (TYSA, Days 2–7) | | |
| Volume trend Days 2–7 (growing / stable / declining) | | |
| gotsport_api existing sources — week 1 total | | GREEN / YELLOW / RED |
| TGS sources combined — week 1 total | | GREEN / YELLOW / RED |

**Week 1 Verdict:** GREEN / YELLOW / RED

> [!warning] Escalation trigger
> If YELLOW or RED at any point during the week: FORGE files SENTINEL queue item immediately — do not wait for May 7. Use `Infrastructure/May 1 — Pipeline Red Signal Response Protocol.md` for response steps.

---

## Section 4: Incident Log

| Date | Run # | Source | Issue | Resolution | Status |
|------|--------|--------|-------|------------|--------|
| | | | | | |

Incidents include: pipeline failures, parse error spikes > 10%, unexpected volume drops > 30% day-over-day, schema errors, dedup failures, null source_org_id entries.

**Incident SQL — dedup check (run after any RED signal):**
```sql
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*) AS count
FROM games
WHERE created_at > NOW() - INTERVAL '24 hours'
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1;
```
Expected: 0 rows. Any result here is a pipeline incident — stop and notify SENTINEL.

---

## Section 5: Stage 2 Authorization Input

FORGE completes this section on May 7 and submits to SENTINEL as input to the Stage 2 authorization decision. Cross-reference: `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md`.

Per Stage 2 authorization criteria: Stage 1 must achieve GREEN verdict before Stage 2 expansion is authorized.

| Criterion | Status | Evidence |
|-----------|--------|----------|
| 7 consecutive days of pipeline runs completed (May 1–7) | YES / NO | Section 2 Run # column |
| Week 1 TYSA volume verdict = GREEN (Run 1 ≥ 5,000 games) | YES / NO | Section 3 TYSA total |
| No unresolved RED incidents in Section 4 | YES / NO | Section 4 incident log |
| All-source pipeline success rate = 100% (7/7 runs) | YES / NO | Section 3 daily run rate |
| Parse error rate across all runs < 5% | YES / NO | Section 3 parse error rate |
| FORGE recommendation for Stage 2 | AUTHORIZE / DEFER | |

**FORGE Stage 2 recommendation (fill May 7):**

> _[FORGE writes one paragraph here: assessment of week 1 health, specific metrics supporting the recommendation, and whether Stage 2 should proceed on the target date or defer.]_

---

## References

- `Infrastructure/May 1 — Per-Source Game Volume Forecast.md` — Section 1 forecast source
- `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` — TYSA GREEN/YELLOW/RED thresholds
- `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md` — companion daily monitoring log (more granular than this report)
- `Infrastructure/May 1 Pipeline Monitoring Queries.md` — SQL queries for Section 2 fills
- `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` — Section 5 feeds Stage 2 decision
- `Infrastructure/May 1 — Pipeline Red Signal Response Protocol.md` — escalation protocol for RED signals
- `Infrastructure/May 9 — FORGE Infrastructure Readiness Assessment.md` — this document's Week 1 verdict is a key input

*FORGE — 2026-04-28*
