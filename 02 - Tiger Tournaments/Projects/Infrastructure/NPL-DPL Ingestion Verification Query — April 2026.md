---
title: NPL-DPL Ingestion Verification Query — April 2026
tags: [infrastructure, npl, dpl, data-quality, sql, monitoring, evo-draw]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: ready — run at next psql session
---

# NPL/DPL Ingestion Verification Query — April 2026

---

## Context

NPL (National Premier League) and DPL (Development Player League) are ranked #5 in the Source Coverage Priority Matrix (Score 7 — tied with USYS), marked **Active, unverified**. Both are Tier 3 / High Competitive leagues with an estimated 1,000–5,000 combined games per season.

"Active, unverified" means the pipeline is presumed to be ingesting from these sources, but last ingestion date has never been confirmed. If either league has drifted silent, rankings for the teams they serve are using stale game history — and Event Strength Phase 1 `strength_class` assignments for NPL/DPL events will be incorrect.

This document contains a single copy-paste query to confirm NPL and DPL ingestion is current — or surface a staleness problem in under 10 minutes. Run it at the next `psql youth_soccer` session, ideally before April 29.

**When to run:** Next psql session. Recommended before any Event Strength Phase 1 work involving NPL/DPL-tagged events.

---

## Primary Verification Query

```sql
SELECT
  event_tier,
  season,
  COUNT(*) AS total_games,
  MAX(match_date) AS most_recent_game,
  MIN(match_date) AS oldest_game,
  (CURRENT_DATE - MAX(match_date)) AS days_since_most_recent
FROM games
WHERE is_hidden = false
  AND (
    event_tier IN ('npl', 'dpl')
    OR event_name ILIKE '%National Premier League%'
    OR event_name ILIKE '%NPL%'
    OR event_name ILIKE '%Development Player League%'
    OR event_name ILIKE '% DPL%'
    OR division ILIKE '%NPL%'
    OR division ILIKE '%DPL%'
  )
GROUP BY event_tier, season
ORDER BY season DESC, event_tier;
```

**Note on multi-condition filter:** The query filters on `event_tier IN ('npl', 'dpl')` as the primary match, with `event_name` and `division` text patterns as fallback. This catches games that are ingested but not yet tier-classified, preventing a false PASS due to a classification gap. If the fallback conditions return rows with `event_tier = NULL` or an unexpected tier value, flag those rows for FORGE.

---

## Fallback Query — Last 365 Days

If current-season filter is uncertain (academic year vs. fiscal year boundary), use this version to capture all recent ingestion regardless of season label:

```sql
SELECT
  event_tier,
  COUNT(*) AS total_games,
  MAX(match_date) AS most_recent_game,
  (CURRENT_DATE - MAX(match_date)) AS days_since_most_recent,
  COUNT(DISTINCT CASE WHEN match_date >= CURRENT_DATE - INTERVAL '90 days' THEN source_id END) AS games_last_90d
FROM games
WHERE is_hidden = false
  AND match_date >= CURRENT_DATE - INTERVAL '365 days'
  AND (
    event_tier IN ('npl', 'dpl')
    OR event_name ILIKE '%National Premier League%'
    OR event_name ILIKE '%NPL%'
    OR event_name ILIKE '%Development Player League%'
    OR event_name ILIKE '% DPL%'
  )
GROUP BY event_tier
ORDER BY event_tier;
```

Use this version if the primary query returns no current-season row but you suspect ingestion exists under a different season label.

---

## Expected Output Columns

| Column | What it means |
|--------|---------------|
| `event_tier` | Tier label (`npl`, `dpl`, or NULL/other if classification gap) |
| `season` | Competition season (e.g., `"2025-2026"`) |
| `total_games` | Count of visible NPL or DPL games for that season |
| `most_recent_game` | Date of the most recently ingested game |
| `oldest_game` | Earliest game on record for that season |
| `days_since_most_recent` | Days between today and most recent game — primary staleness indicator |

**Expected rows:** Two current-season rows minimum — one for `npl`, one for `dpl`. If only one league appears, the other may have a classification gap (check the fallback query).

---

## Pass / Fail Thresholds

### PASS — No Action Required

Both of the following must be true for both NPL **and** DPL:

- `days_since_most_recent` ≤ 30 for the current season row, **AND**
- `total_games` for the current season ≥ 1,000 combined (NPL + DPL together; ≥ 500 each is preferred but combined floor applies)

**Next action:** No action. Note PASS result in next FORGE briefing: `"NPL/DPL staleness check: PASS. NPL most recent: [date], [N] games. DPL most recent: [date], [N] games."`

---

### YELLOW — Flag in Next Briefing

Either of the following:

- `days_since_most_recent` is 31–60 days for NPL **or** DPL (ingestion may be lagging but not stopped), **OR**
- Combined `total_games` for current season is 500–999 (below expected but above zero)

**Next action:** Flag in next FORGE briefing with specific values. SENTINEL reviews before any Event Strength Phase 1 NPL/DPL classifications are finalized. No immediate halt.

---

### FAIL — Escalate to SENTINEL

Any of the following:

- `days_since_most_recent` > 60 days for either NPL **or** DPL, **OR**
- Combined `total_games` for current season < 500, **OR**
- No current-season row returned for NPL or DPL at all

**Next action:** Escalate to SENTINEL immediately. Do not proceed with Event Strength Phase 1 classifications for NPL/DPL-adjacent leagues until ingestion is confirmed current. Post a queue item with query output and days-since-most-recent values.

---

## Downstream Dependencies

Event Strength Phase 1 uses team ratings as inputs to assign `strength_class` to events. NPL and DPL team ratings are derived from game history. If NPL/DPL ingestion is stale:

- `strength_class` assignments for NPL/DPL events will reflect outdated team quality
- Teams that have improved since last ingestion will be underrated, pushing event strength down
- SENTINEL should not finalize Event Strength Phase 1 for NPL/DPL events until this check returns PASS

This check is a gate condition for Phase 1 accuracy on these two leagues.

---

## Recommended Run Timing

Run at the next psql session, ideally on or before April 29. Takes approximately 5–10 minutes to execute and review.

---

## References

- `Infrastructure/Source Coverage Priority Matrix — April 2026.md` — NPL/DPL ranked Score 7, tied #5
- `Infrastructure/Source Health Monitoring Queries.md` — weekly monitoring template (less targeted)
- `02 - Tiger Tournaments/Projects/Leagues/League Hierarchy.md` — tier and calibration values
- `Infrastructure/ECNL RL Staleness Verification Query — April 2026.md` — document this mirrors in structure

*FORGE — 2026-04-25*
