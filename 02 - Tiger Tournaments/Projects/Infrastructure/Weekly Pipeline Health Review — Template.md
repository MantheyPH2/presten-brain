---
title: Weekly Pipeline Health Review — Template
tags: [infrastructure, pipeline, monitoring, weekly-review, evo-draw, forge]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: template — first use due 2026-05-08
cadence: Every Friday or first pipeline session of the week
first_review_due: 2026-05-08
---

# Weekly Pipeline Health Review — Template

> **Purpose:** Weekly structured health check. FORGE fills this template each week and files it as `Infrastructure/Weekly Pipeline Health Review — [YYYY-MM-DD].md`. Catches slow deterioration (source drift, volume drops, Stage 2 progress) that daily briefings miss.

> **Cadence:** Every Friday, or the first pipeline session of the week if Friday is missed. First review due 2026-05-08 (7 days post-May-1 deployment).

> **How to use this template:** Copy this file to `Infrastructure/Weekly Pipeline Health Review — [week-ending-date].md`. Fill each section in order. Write the Section 5 SENTINEL flag last.

---

## Header

```
Week ending:              [YYYY-MM-DD]
Review number:            [1, 2, 3...]
FORGE filing date:        [YYYY-MM-DD]
Pipeline deployment date: 2026-05-01
Days since deployment:    [N]
```

---

## Section 1: 7-Day Game Count Summary

Run **Query A** from Section 6. Fill table from results.

| Source | Games Ingested (7d) | Avg per Run | Prior Week Avg | Delta (%) | Status |
|--------|--------------------|--------------|--------------|-----------|----|
| ECNL Girls | | | | | |
| ECNL Boys | | | | | |
| MLS NEXT | | | | | |
| GA Girls | | | | | |
| GA Boys | | | | | |
| GA ASPIRE | | | | | |
| ECNL RL Girls | | | | | |
| ECNL RL Boys | | | | | |
| NPL | | | | | |
| DPL | | | | | |
| TGS (ECNL supplemental) | | | | | |
| *Stage 2 sources (add rows after Stage 2 runs)* | | | | | |
| **TOTAL** | | | | | |

**Status definitions:**

| Status | Condition |
|--------|-----------|
| `GREEN` | Within 10% of prior week average (or new source, no prior week) |
| `YELLOW` | 10–25% drop vs prior week |
| `RED` | >25% drop vs prior week, OR 0 games during active season |
| `OFFSEASON` | Near-0 games expected based on seasonal calendar — not a health concern |

> [!warning] Always include every source
> Never omit a source because it is OFFSEASON. Mark it OFFSEASON and include it. A source unexpectedly dropping to OFFSEASON-level volume during the season is a RED signal — it will be invisible if the row is omitted.

> [!note] "GREEN overall with one YELLOW source" = YELLOW overall
> The Section 5 flag must reflect the worst-case individual source status, not the median. Be accurate.

---

## Section 2: Source Freshness Status

Run **Query B** from Section 6. Fill table from results.

| Source | Most Recent Game Date | Days Since Last Ingest | Freshness Status |
|--------|----------------------|----------------------|-----------------|
| ECNL Girls | | | |
| ECNL Boys | | | |
| MLS NEXT | | | |
| GA Girls | | | |
| GA Boys | | | |
| GA ASPIRE | | | |
| ECNL RL Girls | | | |
| ECNL RL Boys | | | |
| NPL | | | |
| DPL | | | |
| TGS (ECNL supplemental) | | | |
| *Stage 2 sources (add rows after Stage 2 runs)* | | | |

**Freshness status definitions (per Source Freshness Audit Template):**

| Status | Condition |
|--------|-----------|
| `FRESH` | Most recent game within expected cadence (≤7 days for weekly+ sources) |
| `STALE-WARNING` | 8–14 days since last ingest for a weekly source (1.5× expected cadence) |
| `STALE-CRITICAL` | >14 days since last ingest for a weekly source, OR 0 games in last 30 days during active season |
| `OFFSEASON` | 0 or near-0 games expected; seasonal — not a health concern |

> If a Source Freshness Audit was run this week, pull values from that audit rather than re-running Query B.

---

## Section 3: Stage 2 Expansion Progress

Update from `Infrastructure/Stage 2 Expansion — SENTINEL Authorization Gate.md`.

| Stage 2 Gate Condition | Status | Date Cleared |
|------------------------|--------|-------------|
| C1 — Stage 1 TX crawl results confirmed | | |
| C2 — Stage 1 results GREEN (delta=0, zero dupes, Stage 2 authorized) | | |
| C3 — Browser session complete, ≥5/7 P1 USYS state org IDs confirmed | | |
| C4 — `ecnl_verified` column live (April 28 Step 2 ✅) | | |
| C5 — Org-ID Master Reference Section 1 populated (no null org-IDs for confirmed entities) | | |
| C6 — No active YELLOW/RED errors from Stage 1 morning-after validation | | |
| **SENTINEL authorization** | | |

```
Stage 2 gate status:    [X/6 conditions met]
Stage 2 expansion start date: [date, or "Not yet authorized"]
Stage 2 new sources active:   [list, or "N/A"]
```

---

## Section 4: Error and Anomaly Log

Any errors, unexpected volume drops, or anomalies observed in the past 7 days.

| Date | Source | Error Description | Resolution | Status |
|------|--------|------------------|------------|--------|
| | | | | |

If no errors: "No errors this week."

Include any YELLOW or RED rows triggered by the GotSport Org-ID Inactive Detection Protocol (`Infrastructure/GotSport Org-ID Inactive Detection Protocol.md`) during the review window.

---

## Section 5: SENTINEL Weekly Flag Summary

*Write this section last, after completing Sections 1–4.*

```
OVERALL PIPELINE STATUS: GREEN / YELLOW / RED
REASON (if not GREEN): [one sentence — e.g., "NPL STALE-WARNING: 11 days since last ingest"]
ACTION REQUIRED FROM SENTINEL: YES / NO — [if YES, describe specifically what SENTINEL must do]
```

**Status derivation rules:**

- **GREEN:** All sources FRESH or OFFSEASON; no RED or YELLOW game count drops in Section 1; no unresolved errors in Section 4.
- **YELLOW:** ≥1 STALE-WARNING source, OR ≥1 YELLOW game count drop in Section 1, OR error in Section 4 marked "resolved." SENTINEL informed; no immediate action required unless pattern persists next week.
- **RED:** Any STALE-CRITICAL source, OR any RED game count drop in Section 1, OR any unresolved error in Section 4. SENTINEL action required before next weekly review.

---

## Section 6: Standing SQL Queries

Run these queries each week to generate Sections 1–2. Copy-paste into psql; no modification needed.

---

### Query A — 7-Day Game Count by Source (Section 1)

```sql
-- 7-day game count by event_tier (primary grouping for GotSport sources)
-- Also separates TGS (source_name = 'tgs') from GotSport sources
SELECT
  COALESCE(event_tier, 'unknown') AS source_label,
  source_name,
  COUNT(*) AS games_7d,
  ROUND(COUNT(*)::numeric / 7, 1) AS avg_per_day
FROM games
WHERE created_at >= NOW() - INTERVAL '7 days'
  AND is_hidden = false
GROUP BY event_tier, source_name
ORDER BY games_7d DESC;
```

**Usage:** Map `source_label` + `source_name` to the rows in Section 1. For prior-week average: compare against last week's filing.

**Delta calculation:** Use Query C below.

---

### Query B — Most Recent Game Date per Source (Section 2)

```sql
-- Most recent game date and ingestion date per source
SELECT
  COALESCE(event_tier, 'unknown') AS source_label,
  source_name,
  MAX(game_date) AS most_recent_game_date,
  MAX(created_at) AS most_recent_ingest,
  NOW()::date - MAX(game_date) AS days_since_last_game,
  NOW() - MAX(created_at) AS time_since_last_ingest
FROM games
WHERE is_hidden = false
GROUP BY event_tier, source_name
ORDER BY days_since_last_game DESC;
```

**Usage:** Fill the "Most Recent Game Date" and "Days Since Last Ingest" columns in Section 2.

> [!note] game_date vs created_at
> `game_date` is when the game was played. `created_at` is when FORGE ingested it. For freshness status, use `days_since_last_game` — a source with recent crawls but no new games is OFFSEASON, not FRESH.

---

### Query C — 7-Day vs Prior-7-Day Delta (Section 1 Delta column)

```sql
-- Current 7-day vs prior 7-day game count delta per source
SELECT
  COALESCE(event_tier, 'unknown') AS source_label,
  source_name,
  COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '7 days') AS count_current_7d,
  COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '14 days'
                     AND created_at < NOW() - INTERVAL '7 days') AS count_prior_7d,
  CASE
    WHEN COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '14 days'
                            AND created_at < NOW() - INTERVAL '7 days') = 0
    THEN NULL
    ELSE ROUND(
      (COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '7 days')::numeric
       - COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '14 days'
                            AND created_at < NOW() - INTERVAL '7 days')::numeric)
      / NULLIF(COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '14 days'
                                  AND created_at < NOW() - INTERVAL '7 days'), 0)
      * 100, 1)
  END AS pct_delta
FROM games
WHERE is_hidden = false
  AND created_at >= NOW() - INTERVAL '14 days'
GROUP BY event_tier, source_name
ORDER BY pct_delta ASC NULLS LAST;
```

**Usage:** Populate the "Delta (%)" column in Section 1. A negative value = volume drop. Null = new source with no prior-week baseline (set Status = GREEN for first week).

---

## Cadence Schedule

| Review # | Week Ending | Due Date |
|----------|------------|---------|
| 1 | 2026-05-08 | 2026-05-08 |
| 2 | 2026-05-15 | 2026-05-15 |
| 3 | 2026-05-22 | 2026-05-22 |
| 4 | 2026-05-29 | 2026-05-29 |

FORGE extends this table forward each month. SENTINEL adds this review to its weekly briefing as the canonical pipeline health source.

---

## References

- `Infrastructure/Daily Pipeline Log Format Spec.md` — per-run output this weekly review summarizes
- `Infrastructure/Source Freshness Audit Template.md` — freshness data for Section 2 (use if audit was run this week)
- `Infrastructure/GotSport Org-ID Inactive Detection Protocol.md` — anomaly inputs for Section 4
- `Infrastructure/Stage 2 Expansion — SENTINEL Authorization Gate.md` — gate conditions for Section 3
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — Day 1 baseline for Section 1 prior-week comparison

*FORGE — 2026-04-25*
