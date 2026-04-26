---
title: ECNL RL Staleness Verification — Presten Execution Package
type: execution-package
tags: [infrastructure, ecnl-rl, data-quality, sql, staleness, execution, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: ready — run at next psql session
related: "[[ECNL RL Staleness Verification Query — April 2026]]", "[[Source Coverage Priority Matrix — April 2026]]", "[[ECNL Migration Rating Continuity Spec — June 2026]]"
---

# ECNL RL Staleness Verification — Presten Execution Package

**When to run:** April 29 — at the same session as the GA ASPIRE recompute verification. ECNL RL staleness is independent of April 28 and does not need to wait for the recompute. If the April 29 session runs long, run at the May 1 pre-deployment check instead. Deadline: May 1 at the latest.

**Estimated run time:** ≤ 10 minutes in psql.

---

## Pre-Run Gate

```
[ ] psql access to youth_soccer DB confirmed
[ ] No other active long-running queries in progress (this query is read-only; safe to run concurrently)
→ Both checked: proceed.
→ psql not available: defer to next available session, no later than May 1.
```

**Note:** TGS AthleteOne is the expected primary ingestion source for ECNL RL. If uncertain when that ingestion path became active, expect the earliest ECNL RL game dates to be from the 2023-2024 or 2024-2025 season at the earliest. If the most recent game predates this window, the ingestion path itself may never have been active — flag to SENTINEL.

---

## Query 1 — ECNL RL Ingestion Recency

**Purpose:** Confirm ECNL RL ingestion is current (≤ 30 days since most recent game) and that current-season volume is above the expected floor.

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

**Note on multi-condition filter:** Catches ECNL RL games not yet assigned `event_tier = 'ecrl'`, preventing a false PASS due to tier classification gaps.

**Save output to:** `02 - Tiger Tournaments/Projects/Reports/ecnl-rl-staleness-check-[date].md`

**Replace `[date]` with today's date in YYYY-MM-DD format.**

---

### Query 1 Thresholds

**PASS:** For the current-season row (2025-2026):
- `days_since_most_recent` ≤ 14, AND
- `total_games` ≥ 500

**YELLOW:** Either of the following:
- `days_since_most_recent` 15–30 days, OR
- `total_games` 200–499

ECNL RL is lower-volume than full ECNL. A count in the 200–499 range may reflect low active play volume rather than an ingestion gap. Log in next FORGE briefing. ELO notes low ECNL RL volume in ECNL migration rating continuity analysis.

**FAIL:** Any of the following:
- `days_since_most_recent` > 30 days for the current season, OR
- `total_games` < 200 for the current season, OR
- No row returned for the 2025-2026 season (zero ECNL RL games this season)

Escalate to SENTINEL immediately. Do not proceed with ECNL migration planning until resolved.

---

## Query 2 — ECNL RL vs Full ECNL Volume Comparison

**Purpose:** Verify ECNL RL game volume is in a plausible ratio relative to full ECNL. ECNL RL is a sub-league; its count should be lower than full ECNL.

```sql
SELECT event_tier, COUNT(*) AS game_count, MAX(game_date) AS most_recent_game
FROM games
WHERE event_tier IN ('ecnl', 'ecrl')
  AND season = '2025-2026'
GROUP BY event_tier;
```

**Expected:** ECNL RL (`ecrl`) game count is lower than full ECNL (`ecnl`). If `ecrl` count > `ecnl` count, investigate — this is unexpected given relative league sizes and likely indicates a classification or source overlap issue.

**Save output to:** Same file as Query 1 (`ecnl-rl-staleness-check-[date].md`), append below Query 1 results.

---

## Decision Table

| Query 1 Result | Query 2 Result | Action |
|----------------|----------------|--------|
| PASS | Expected ratio (RL < full ECNL) | No action. File GREEN status in next FORGE briefing. |
| PASS | ECNL RL ≥ full ECNL game count | Flag to SENTINEL. Investigate whether ECNL RL is over-counted or full ECNL under-counted. |
| YELLOW | Any | Log in FORGE briefing with specific values. ELO notes low ECNL RL volume in ECNL migration rating continuity analysis. No immediate halt. |
| FAIL | Any | Escalate to SENTINEL immediately. Do not proceed with ECNL migration planning until resolved. |

---

## Supplemental Query (Optional) — Team Coverage Check

Run only if Query 1 returns PASS and you want to verify coverage isn't silently concentrated in a small subset of clubs.

```sql
SELECT
  COUNT(DISTINCT home_team_id) + COUNT(DISTINCT away_team_id) AS approx_team_appearances,
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

**Expected:** `approx_team_appearances` ≥ 600. This double-counts teams appearing as both home and away; divide by ~2 for a unique team estimate — expect 300+ unique teams. If the result is < 200 appearances, coverage is heavily concentrated and warrants investigation.

---

## Post-Run: File Results

After completing Query 1 and Query 2:

1. Save output file: `02 - Tiger Tournaments/Projects/Reports/ecnl-rl-staleness-check-[date].md`
2. Record the verdict (PASS / YELLOW / FAIL) in the next FORGE briefing under "What I found today"
3. If FAIL: post a FORGE queue item immediately (`10 - Agents/FORGE/Queue/`) flagging SENTINEL with the specific `days_since_most_recent` and `total_games` values
4. If YELLOW: ELO should be notified so the ECNL migration rating continuity spec can note the low-volume caveat

---

## Downstream Dependencies

This verification result feeds:

1. **ECNL Migration Rating Continuity Spec (ELO)** — `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md`. If ECNL RL is stale or low-volume, the migration spec's rating continuity model for RL teams needs a caveat. ELO must know before finalizing the spec (due May 10).

2. **ecnl_verified Column Accuracy** — The Step 2b backfill targets `event_tier = 'ecnl'`, not `ecrl`. If ECNL RL games are classified under `ecrl`, they are NOT covered by the backfill and will not have `ecnl_verified = TRUE`. ECNL RL games are expected to come via the TGS scraper (same source as full ECNL), so this may be intentional — but note to ELO that ECNL RL games are not covered by `ecnl_verified`.

3. **May 9 DSS Readiness Assessment** — ECNL data quality is a background assumption in Evo Draw's competitive coverage claim. ECNL RL staleness weakens that claim. SENTINEL needs the result before finalizing the May 9 coverage narrative.

---

## References

- `Infrastructure/ECNL RL Staleness Verification Query — April 2026.md` — source query (used verbatim in Query 1 above)
- `Infrastructure/Source Coverage Priority Matrix — April 2026.md` — ECNL RL ranked Score 9/9 (highest unverified priority)
- `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` — downstream spec that depends on ECNL RL accuracy
- `task-2026-04-26-ecnl-verified-ingestion-path-audit.md` — related audit for ecnl_verified assignment
- `Infrastructure/Source Health Monitoring Queries.md` — ongoing monitoring template

*FORGE — 2026-04-26*
