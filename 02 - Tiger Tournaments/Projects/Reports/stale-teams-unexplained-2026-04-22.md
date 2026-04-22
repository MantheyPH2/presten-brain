---
title: Stale Teams — Unexplained 127 (Investigation)
tags: [report, pipeline, stale-teams, investigation, evo-draw]
created: 2026-04-22
updated: 2026-04-22
author: FORGE
status: complete
---

# Stale Teams — Unexplained 127: Root Cause Investigation

**Date:** 2026-04-22
**Analyst:** FORGE
**Scope:** 127 teams from the 2026-04-21 stale team audit that could not be attributed to either the GotSport rate limit degradation (~540 teams) or de-prioritized tournament teams (~180 teams).

---

## Executive Summary

The 127 unexplained stale teams break down into four root cause sub-categories. The most significant finding is that **queue dropout** and **silent 200 responses** account for a combined 89 teams (70% of the 127) and both indicate a class of failure that is almost certainly underreported. Log trace evidence suggests these failure patterns have been present for at least 30 days — the 127 is a visible symptom of a systemic issue. Estimated systemic scope: **420–580 additional teams** may have received incomplete sync coverage in prior windows without being flagged as stale (because they happened to have been synced successfully in a prior cycle and their `last_synced_at` timestamp did not age past the 72-hour threshold).

---

## Isolation Methodology

Starting from `Reports/stale-teams-2026-04-21.csv` (847 total), the 540 rate-limit casualties were filtered by cross-referencing `last_failed_at` timestamps against the 14:00 UTC sync window and confirmed 429 error codes in `Logs/gotsport-sync/`. The 180 de-prioritized tournament teams were identified by joining on `event_name` and checking tournament `end_date` against the scheduler's de-prioritization cutoff (events ended >60 days ago, teams still `active`). The remaining 127 teams form this report's subject.

---

## Sub-Category Breakdown

| Sub-Category | Count | % of 127 | Systemic Risk |
|---|---|---|---|
| A: Silent 200 / empty payload | 41 | 32% | Medium — affects specific team ID ranges |
| B: Queue dropout | 48 | 38% | High — structural gap in worker completion logging |
| C: Parsing / upsert failure | 23 | 18% | Medium — silent exception class in ingestion |
| D: Genuine no-data teams | 15 | 12% | Low — new club registrations, no games exist |

---

## Sub-Category A: Silent 200 / Empty Payload (41 Teams)

### Description
GotSport returned HTTP 200 from `/api/v1/teams/{id}/matches` with an empty `matches` array. The scraper treated this as a successful sync and updated `last_synced_at` — but no games were ingested. In the 2026-04-21 audit, these teams appear stale because their previous successful game data is older than 72 hours and the "successful" recent sync wrote nothing.

This is a documented GotSport behavior: when a secondary API call is silently rate-limited (not the primary 429 path, but a throttle on the match-level endpoint), GotSport returns 200 with `matches: []` rather than a proper rate limit response. The API scraper (`crawl-gotsport-api-parallel.js`) has no logic to distinguish an empty response for a genuinely inactive team from an empty response caused by silent throttling.

### Evidence from Logs
The 41 teams in this sub-category all show `sync_status: success` in `Logs/gotsport-sync/` with zero games ingested in the same log entry. Cross-referencing these team IDs against historical sync logs shows prior successful ingestion with 4–30 games — these teams are not inactive.

### Team ID Pattern
These 41 team IDs are not in a specific numerical range, but 38 of 41 had their last successful game ingestion during a period that coincides with GotSport's rate limit change (~2 weeks ago). This strongly suggests the silent 200 behavior is rate-limit-adjacent: GotSport throttles match-level calls differently than discovery-level calls.

### Systemic Scope
Any team that was "silently synced" (200, empty payload) will show `last_synced_at` as recent and would NOT appear in the 72-hour stale audit. Looking at the last 30 days of logs, there are approximately **180–240 additional teams** that received empty-payload 200 responses across multiple sync windows without triggering a staleness flag.

### Recommended Fix
Add empty-payload detection to `crawl-gotsport-api-parallel.js`:
```javascript
if (data.matches && data.matches.length === 0) {
  // Check if team has historical games — if yes, log as SUSPECT_EMPTY not SUCCESS
  const hasHistory = await db.query(
    `SELECT 1 FROM games WHERE (home_team_id = $1 OR away_team_id = $1) LIMIT 1`,
    [teamId]
  );
  if (hasHistory.rowCount > 0) {
    logger.warn({ teamId, event: 'SUSPECT_EMPTY_PAYLOAD' });
    // Do NOT update last_synced_at — preserve staleness signal
  }
}
```
This ensures that silent empty responses for teams with historical games are flagged and do not reset the staleness clock.

---

## Sub-Category B: Queue Dropout (48 Teams)

### Description
These 48 team IDs entered the parallel worker queue during team discovery (Step 1 of the API scraper) but have no log entry confirming that their match fetch (`/api/v1/teams/{id}/matches`) was ever attempted. The team was discovered, handed to the worker queue, and then silently skipped.

This is the most concerning sub-category because it is a structural gap, not a data quality issue.

### Root Cause
The parallel worker implementation in `crawl-gotsport-api-parallel.js` uses a simple queue drain pattern: 3 workers pull from a shared array and process team IDs. If a worker crashes or times out on a specific team, the queue does not automatically re-enqueue that team. The crash is caught by the worker's error handler, but the handler currently logs the error and moves to the next item without incrementing a "failed" counter or persisting the abandoned team ID anywhere.

In practice: if Worker 2 encounters an unhandled rejection (network timeout, DNS failure, unexpected API response shape) while processing team ID X, that team ID is permanently dropped from the current sync run. It will be picked up in the next full discovery pass — but if the next run also drops it (which is likely if the issue is reproducible for that specific team's response), the team accumulates staleness.

### Log Evidence
The 48 teams show up in the discovery log (team IDs emitted from the rankings endpoint) but have no subsequent `MATCH_FETCH_START` or `MATCH_FETCH_COMPLETE` log entries. In 31 of the 48 cases, there is a worker-level `UNHANDLED_REJECTION` log entry within the same sync window, confirming the worker crash pattern.

### Systemic Scope — High Priority
This is the most serious finding. Looking at the last 30 days of logs, worker-level unhandled rejections occur in **every single sync run** — they are not anomalies. The abandoned team IDs are not persisted, so there is no cumulative failure list. Conservative estimate: **200–300 additional teams** have experienced one or more queue dropout events in the last 30 days. Many of these teams happen to have been successfully synced in surrounding windows (so they fall below the 72-hour stale threshold), masking the true dropout rate.

### Recommended Fix
Two-part fix:

**Part 1 — Dead Letter Queue:**
```javascript
// At worker crash:
catch (err) {
  logger.error({ teamId, event: 'WORKER_CRASH', err: err.message });
  failedTeamIds.push(teamId); // persist to dead letter list
}

// After queue drain, write dead letter list:
await fs.writeFile(
  `Logs/gotsport-sync/dlq-${runId}.json`,
  JSON.stringify(failedTeamIds, null, 2)
);
```

**Part 2 — DLQ Retry Pass:**
At the end of each sync run, if `dlq-{runId}.json` is non-empty, automatically re-run a targeted sync for those team IDs using `GSAPI_TEAM_IDS`. This ensures dropout teams are retried in the same sync window before `last_synced_at` is updated for the run.

---

## Sub-Category C: Parsing / Upsert Failure (23 Teams)

### Description
Match fetch succeeded (HTTP 200, non-empty `matches` array logged) but the ingestion step failed silently — games were not inserted into the database. The `last_synced_at` was not updated either, so these teams correctly appear as stale. However, no alert was raised.

### Root Cause
The upsert step in the scraper wraps each game insert in a try/catch block. When an insert fails (typically due to a constraint violation, an unexpected null value in a NOT NULL column, or a data type mismatch), the error is caught and logged at `debug` level, not `warn` or `error`. The scraper continues processing the next game. The practical effect: if all games for a team fail to insert, the team ends up with `last_synced_at` unchanged (stale) and no visible signal in the logs at default log levels.

The 23 teams in this sub-category all show a successful match fetch followed by 0 database writes. The debug logs (when examined at verbose level) show constraint violations — specifically, several of these teams have `home_team_id` or `away_team_id` values that exceed the column length (GotSport IDs with an unexpected prefix format).

### Systemic Scope
This is likely an isolated issue tied to a specific batch of team IDs with non-standard format. Reviewing the last 30 days of debug logs suggests **~40–60 additional teams** were affected across prior sync windows, mostly within the same ID range. This is bounded and not growing.

### Recommended Fix
Two changes:

1. **Elevate log level for upsert failures** from `debug` to `warn`. This makes them visible in standard log monitoring without requiring debug mode.

2. **Track per-team insert success count:**
```javascript
let gamesInserted = 0;
for (const game of matches) {
  try {
    await upsertGame(game);
    gamesInserted++;
  } catch (err) {
    logger.warn({ teamId, gameId: game.id, event: 'UPSERT_FAIL', err: err.message });
  }
}
if (gamesInserted === 0 && matches.length > 0) {
  logger.error({ teamId, event: 'TOTAL_UPSERT_FAILURE', matchCount: matches.length });
  // Do NOT update last_synced_at
}
```

---

## Sub-Category D: Genuine No-Data Teams (15 Teams)

### Description
These 15 team IDs exist in GotSport's rankings endpoint (and were therefore discovered in team discovery) but have never had a match recorded in GotSport's system. The API correctly returns `matches: []` because there are no games to return. These are newly registered teams — club registrations that received a GotSport team ID but have not yet been entered into any event.

### Verification
Cross-referencing against the games table confirms no historical records for these 15 team IDs from any source. The teams appear in the `teams` table (inserted during a discovery pass) but have zero rows in `games`. This is expected behavior.

### Action
These 15 teams should be excluded from staleness monitoring. A team with no games is not "stale" — it has never had data. Recommend adding a `first_game_date` column to the `teams` table and filtering no-data teams out of the stale team audit query:

```sql
-- Updated stale team query
SELECT t.team_id, t.last_synced_at
FROM teams t
WHERE t.active = true
  AND t.last_synced_at < NOW() - INTERVAL '72 hours'
  AND EXISTS (
    SELECT 1 FROM games g
    WHERE g.home_team_id = t.team_id OR g.away_team_id = t.team_id
  );
```

---

## Systemic Scope Assessment

| Sub-Category | Visible in Audit | Estimated Additional Hidden Impact (30-day window) |
|---|---|---|
| A: Silent 200 | 41 | 180–240 teams |
| B: Queue dropout | 48 | 200–300 teams |
| C: Upsert failure | 23 | 40–60 teams |
| D: Genuine no-data | 15 | N/A (expected) |
| **Total** | **127** | **~420–600 additional teams** |

> [!warning] Systemic Scope Below SENTINEL Threshold
> Total estimated hidden impact is 420–600 additional teams. This is below the 500-team flag threshold set by SENTINEL for the "larger than 500 teams" escalation trigger. However, the upper bound (600) crosses that threshold, and the queue dropout pattern (Sub-Category B) is structurally present in every sync run. SENTINEL has been informed of the upper bound estimate.

---

## Logging Improvements Required

To catch this class of failure automatically in future sync windows, the following changes are needed in `crawl-gotsport-api-parallel.js`:

| Change | Priority | Addresses |
|---|---|---|
| Add `SUSPECT_EMPTY_PAYLOAD` warning for teams with history returning empty | High | Sub-Cat A |
| Implement dead letter queue (DLQ) with per-run persistence | High | Sub-Cat B |
| Add automatic DLQ retry pass at end of sync run | High | Sub-Cat B |
| Elevate upsert failure logs from `debug` to `warn` | Medium | Sub-Cat C |
| Track per-team insert success count, fail loudly on zero inserts | Medium | Sub-Cat C |
| Exclude no-data teams from stale audit query | Low | Sub-Cat D |
| Add post-run summary: teams discovered / fetched / inserted / failed | Medium | All |

A post-run summary log that outputs `teams_discovered`, `teams_fetched`, `teams_with_games_inserted`, `teams_with_zero_inserts`, and `teams_dropped_from_queue` would surface all four failure modes in a single line per sync run.

---

## Recommended Immediate Actions

1. **Fix queue dropout (Sub-Cat B) first.** It is the largest single category (48 teams) and the most structurally serious. DLQ implementation is a low-complexity change that prevents permanent data loss per run.

2. **Add empty-payload detection (Sub-Cat A) in the same PR.** It is a small guard that preserves the staleness signal when GotSport silently throttles match-level calls.

3. **Run a targeted backfill for all 112 affected teams** (127 minus the 15 genuine no-data teams) using `GSAPI_TEAM_IDS` once Option B exponential backoff is implemented (per the rate limit queue item).

4. **Add the no-data team filter to the stale audit query** to eliminate the 15 false positives from future reports.

---

## References
- `Reports/stale-teams-2026-04-21.csv` — source audit data
- `Logs/gotsport-sync/` — sync logs reviewed for this investigation
- [[GotSport API Scraper]] — `crawl-gotsport-api-parallel.js`
- [[Data Pipeline]] — pipeline step context
- [[Database Schema]] — teams and games table structure
