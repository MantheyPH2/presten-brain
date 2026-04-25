---
title: ECNL RL Staleness Verification Query — April 2026
tags: [infrastructure, ecnl-rl, data-quality, sql, monitoring, evo-draw]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: ready — run at next psql session
---

# ECNL RL Staleness Verification Query — April 2026

---

## Context

ECNL RL (Regional League) is the highest-priority gap in the Source Coverage Priority Matrix: score 9/9 across Volume, Impact, and Effort dimensions. It represents ~700 clubs at Tier 2 — High Competitive. If ingestion has silently drifted, ECNL RL is the largest currently-undetected ranking distortion possible in Evo Draw.

This document contains a single copy-paste query that confirms ECNL RL ingestion is current — or surfaces a staleness problem in 2 minutes. Run it at the next `psql youth_soccer` session, before any Stage 2 planning decisions that involve ECNL RL-adjacent leagues.

**When to run:** Next psql session after April 28. Recommended as the first query of any data quality review session.

---

## Query

```sql
SELECT
  season,
  COUNT(*) AS total_games,
  MAX(match_date) AS most_recent_game,
  MIN(match_date) AS oldest_game,
  (CURRENT_DATE - MAX(match_date)) AS days_since_most_recent
FROM games
WHERE is_hidden = false
  AND (
    event_tier = 'ecrl'
    OR event_name ILIKE '%ECNL RL%'
    OR event_name ILIKE '%ECNL Regional%'
    OR division ILIKE '%ECNL RL%'
  )
GROUP BY season
ORDER BY season DESC;
```

---

## Expected Output Columns

| Column | What it means |
|--------|---------------|
| `season` | Competition season (e.g., "2025-2026", "2024-2025") |
| `total_games` | Count of visible ECNL RL games for that season |
| `most_recent_game` | Date of the most recently ingested ECNL RL game |
| `oldest_game` | Earliest ECNL RL game on record for that season |
| `days_since_most_recent` | Days between today and most recent game — the primary staleness indicator |

**Note on multi-condition filter:** The query filters on `event_tier = 'ecrl'` (the normalized tier label set by the pipeline) as well as `event_name` and `division` text patterns. This catches ECNL RL games that may not yet have the `ecrl` event_tier applied, preventing a false PASS due to tier classification gaps.

---

## Pass / Fail Thresholds

### PASS — No Action Required

- `days_since_most_recent` ≤ 30 for the current season row, AND
- `total_games` for the current season ≥ 3,000 (ECNL RL is ~700 clubs, ~47 leagues; 3,000 games is a conservative floor for an active mid-season dataset)

**Next action:** No action. Note PASS result in next FORGE briefing: "ECNL RL staleness check: PASS. Most recent game: [date]. Current season game count: [N]."

---

### YELLOW — Flag in Next Briefing

Either of the following:
- `days_since_most_recent` is 31–60 days (ingestion may be lagging but not stopped), OR
- `total_games` for current season is 1,500–2,999 (below expected but above zero)

**Next action:** Flag in next FORGE briefing with specific values. SENTINEL reviews before Stage 2 decisions involving ECNL RL-adjacent leagues. No immediate halt.

---

### FAIL — Escalate to SENTINEL

Any of the following:
- `days_since_most_recent` > 60 days for the current season, OR
- `total_games` for the current season < 1,500, OR
- No row returned for the current season at all (zero ECNL RL games this season)

**Next action:** Escalate to SENTINEL immediately. Halt Stage 2 planning decisions that depend on ECNL RL coverage being current. Post a queue item with the query results and days-since-most-recent value. SENTINEL will direct the investigation.

---

## Supplemental Query: Team Coverage Check

If the main query returns PASS but you want to verify team-level coverage is not silently concentrated in a subset of clubs:

```sql
SELECT
  COUNT(DISTINCT home_team_id) + COUNT(DISTINCT away_team_id) AS approx_team_count,
  COUNT(*) AS total_games,
  MAX(match_date) AS most_recent_game
FROM games
WHERE is_hidden = false
  AND (
    event_tier = 'ecrl'
    OR event_name ILIKE '%ECNL RL%'
    OR event_name ILIKE '%ECNL Regional%'
  )
  AND match_date >= CURRENT_DATE - INTERVAL '365 days';
```

**Expected:** `approx_team_count` ≥ 600 (note this double-counts teams appearing as both home and away; divide by ~2 for unique team estimate — expect 300+). If this returns < 200 distinct appearances, coverage is heavily concentrated and warrants investigation.

---

## References

- `Infrastructure/Source Coverage Priority Matrix — April 2026.md` — ECNL RL ranked score 9/9
- `Infrastructure/Source Health Monitoring Queries.md` — weekly monitoring template (less targeted)
- `02 - Tiger Tournaments/Projects/Leagues/ECNL RL.md` — ECNL RL structure and calibration
- `02 - Tiger Tournaments/Projects/Leagues/League Hierarchy.md` — tier system context

*FORGE — 2026-04-25*
