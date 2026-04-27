---
title: May 1 — Per-Source Game Volume Forecast
tags: [infrastructure, pipeline, monitoring, forecast, may1, forge]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: active — update if Stage 1 TYSA results arrive before May 1
---

# May 1 — Per-Source Game Volume Forecast

```
Written by:           FORGE
Date:                 2026-04-27
Coverage:             Active and confirmed sources in crawl config as of this writing
Excludes:             Stage 1 TX (TYSA) — org-ID pending browser lookup
                      Stage 2 entities — none yet in config
Note:                 Pre-crawl estimates. Actual results may vary based on season timing,
                      GotSport data availability, and backlog since last sync.
                      These estimates will be superseded by actuals in the First-Week Monitoring Log.
```

---

## Section 1: Active Source Summary

As of 2026-04-27, the following sources are in the active crawl config. No new org-IDs have been confirmed (browser session pending; Stage 1 TX crawl pending). May 1 ingestion comes entirely from the EXISTING scraper scope against the known team ID set (~107,528 GotSport team IDs + TGS scraper schedule).

| Source | Entity / League Coverage | Estimated Games in DB | Est. New Games on May 1 | Confidence |
|--------|--------------------------|----------------------|------------------------|-----------|
| gotsport_api | MLS NEXT, GA Girls/Boys, GA ASPIRE (post-Apr-28), ECNL RL, NPL, DPL, Pre-ECNL, USYS national events | ~1.47M (90% of 1.64M total) | **300–1,200** | Medium — dependent on scraper backlog since last sync and April tournament schedule |
| tgs_athleteone | ECNL Girls, ECNL Boys | ~80K–120K (est., primary ECNL source) | **80–300** | Medium — ECNL spring season active; TGS sync cadence not yet daily |
| tgs | TGS main scraper (older ECNL records, some TGS-native events) | ~30K–60K (est.) | **20–80** | Low-medium — lower incremental volume; older data refreshed less frequently |
| tgs_rl | ECNL RL via TGS feed | ~15K–30K (est.) | **20–80** | Low — ingestion recency unverified (staleness check pending before May 9) |
| gotsport_html | GotSport HTML scraper (supplementary/event-centric fallback) | ~50K–80K (est.) | **10–50** | Low — HTML scraper is not team-discovery capable; event-centric coverage only |

**Note on DB estimates:** The Source Ingestion Baseline Section 1 query (target: April 29–30 psql session) will produce exact counts. The above estimates are derived from the Data Quality document (1.64M total visible games; GotSport ~90%) and the source distribution reflected in current schema. Replace with actuals from the Baseline once captured.

---

## Section 2: Per-Source GREEN/YELLOW/RED Thresholds

Thresholds are set per-source based on expected incremental volume during April peak season. A daily pipeline run during mid-season (April–May) should produce non-zero new games from every active source within 24 hours.

| Source | GREEN (healthy) | YELLOW (investigate) | RED (likely failed) |
|--------|----------------|---------------------|-------------------|
| gotsport_api | ≥ 200 new games | 10–199 new games (partial crawl or off-peak day) | 0–9 new games (crawl failure — expected 300+ during spring season) |
| tgs_athleteone | ≥ 50 new games | 5–49 new games | 0–4 new games |
| tgs | ≥ 10 new games | 1–9 new games | 0 new games |
| tgs_rl | ≥ 10 new games | 1–9 new games | 0 new games (also: run staleness verification query before declaring RED) |
| gotsport_html | ≥ 5 new games | 1–4 new games | 0 new games |

**Threshold rationale:**
- `gotsport_api` GREEN floor is set conservatively at 200 because April is peak spring season across MLS NEXT, ECNL RL, NPL/DPL, and GA leagues. On a weekday (May 1 = Thursday), tournament volume is lower than weekends, but still expect 300–600 new game records from ongoing spring event cycles.
- `tgs_athleteone` and `tgs` thresholds are lower because TGS sync cadence is not daily; a small number of new ECNL records per day is expected.
- `gotsport_html` has the lowest threshold because it is supplementary-only; it covers events the API scraper does not reach, and its output is expected to be sparse on any given day.

**May 1 is a Thursday (weekday):** Expect lower volume than a Saturday. Weekend runs (May 3–4) will show higher counts. Do not flag gotsport_api as RED on May 1 if count is 50–199 — re-check against the May 3 weekend run before escalating.

---

## Section 3: Aggregate Totals

Based on Section 1 per-source estimates:

```
Total estimated new games — May 1 first daily automated run:
  Conservative estimate (at YELLOW floor across all sources):    ~430 games
  Expected estimate (mid-range per Section 1):                  ~700–1,700 games
  Optimistic estimate (120% of expected mid-range):             ~840–2,040 games

GREEN floor (aggregate — first 24 hours):                       ~300 games
RED ceiling (aggregate — if below this, pipeline likely failed): < 50 games

Note: These aggregate figures assume NO Stage 1 TYSA incorporation.
      If TYSA org-ID confirmed and Stage 1 crawl runs before May 1:
      → Add 5,000–25,000 games to conservative/expected/optimistic estimates respectively
        (from Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md, GREEN floor 5,000 games)
```

**Aggregate RED interpretation:** If total new games < 50 after 48 hours of pipeline operation, the pipeline is almost certainly not running. A fully operational pipeline during April spring season should produce hundreds of new game records within hours of first launch.

---

## Section 4: Source Failure Diagnostic Tree

If the first run is aggregate YELLOW (partial ingestion — somewhere between 50–299 new games), use this tree to find the failing source:

```
Aggregate YELLOW — Diagnostic Steps:

1. Run per-source counts:
   SELECT source, COUNT(*) as new_games, MAX(created_at) as last_ingested
   FROM games
   WHERE created_at > '[May 1 pipeline start timestamp]'
   GROUP BY source ORDER BY source;

2. Compare each source count against Section 2 thresholds.
   → Any source individually RED? → Go to Step 3 for that source.
   → All sources individually GREEN or YELLOW? → Aggregate total is low but pipeline is functional.
     May be off-peak day or short backlog since last sync.

3. For each individually-RED source:
   a. Check pipeline log output for that source: connection error? auth failure? 429 rate limit?
   b. gotsport_api RED: check INTER_REQUEST_DELAY_MS is correctly set to 600ms in config.
      If 429 errors in log: backoff is firing — healthy behavior, run completed with partial coverage.
   c. tgs / tgs_athleteone RED: confirm TGS scraper ran (it is not always on the same schedule as API scraper).
      Check log for TGS-specific run timestamp.
   d. gotsport_html RED: html scraper is lowest priority — if all other sources are GREEN, html RED is acceptable.

4. Escalate RED sources to FORGE briefing; GREEN sources confirm partial success.
   → File per-source breakdown in May 1 monitoring log (First-Week Pipeline Monitoring Log).

5. If gotsport_api is RED and backoff logs show no 429s: pipeline may have crashed at startup.
   Check process logs — is cron/scheduler running? Did the pipeline job launch?
```

---

## Section 5: Stage 1 Placeholder (TYSA)

**As of 2026-04-27, Stage 1 TX (TYSA) has not run.** The TYSA org-ID is `pending-browser-lookup`.

**If Stage 1 results become available before May 1:**
- Incorporate TYSA into Section 1 with confirmed org-ID and observed game count from the Stage 1 run
- Update Section 3 aggregate totals to include TYSA volume
- Note: "Stage 1 results incorporated from `Infrastructure/Stage 1 Backfill Results Report.md`"

**If Stage 1 has not run by May 1 (current expectation):**
TYSA volume is not included in this forecast. If the TYSA org-ID is confirmed in the browser session and the Stage 1 config entry is staged before May 1, add:
- **5,000 games** to the aggregate GREEN floor (from Stage 1 Output Spec conservative floor)
- **15,000–25,000 games** to the aggregate expected estimate
These are one-time discovery games, not daily incremental — they will not repeat on May 2 unless new games are played.

---

## Baseline Update Note

The `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` (Section 1) query runs April 29–30 and will produce exact per-source game counts. Once that baseline is captured:
1. Compare the Section 1 "Estimated Games in DB" column above against the actual baseline counts.
2. If actuals differ materially from estimates, recalibrate the Section 2 thresholds before May 1.
3. The Section 3 aggregate totals assume a ~0.05–0.1% daily new-game rate against the existing DB (1.64M × 0.075% ≈ 1,230 games/day mid-season). If the baseline shows a significantly different DB size, scale accordingly.

---

## References

- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — active org-ID status (confirms no new org-IDs as of this writing)
- `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` — TYSA volume estimates for Section 5
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — aggregate GREEN threshold this doc supplements
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — per-source query patterns (Section 4 diagnostic)
- `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` — pre-launch game counts (fill April 29–30; use to refine this forecast)
- `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md` — where per-source actual counts are recorded

*FORGE — 2026-04-27*
