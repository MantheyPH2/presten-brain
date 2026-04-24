---
title: Option B Backoff — Implementation Guide
tags: [report, pipeline, gotsport, rate-limit, scraper, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: FORGE
status: implementation spec — ready for Presten to apply
priority: high
---

# Option B Backoff — Implementation Guide

**Analyst:** FORGE  
**Date:** 2026-04-24  
**Authorization:** Presten ("find a solution")  
**Task:** `task-2026-04-24-implement-option-b-backoff.md`  
**Urgency:** 847 stale teams are holding for backfill pending this change.

> [!info] Relation to pipeline-fixes-2026-04-23.md
> The structural pipeline fix doc (Fix 1) documented a backoff spec with 1200ms base delay and 4 retries. **This document supersedes that spec.** Presten's rate limit strategy decision set the parameters below. The base inter-request delay drops to 600ms; retries increase to 5; jitter is capped at ±20% of wait time (not a flat 500ms).

---

## What Changes and Why

### Current Behavior

`crawl-gotsport-api-parallel.js` uses a hardcoded **1200ms fixed delay** between requests. On a 429 or 503 response, the team ID is sent to the DLQ (after the structural fix) with up to 4 retries. The 847 stale teams from the 2026-04-21 audit represent a backlog that cannot be cleared efficiently at 1200ms/request — that is the rate limit floor, not the optimal delay.

**GotSport's new rate limit:** ~60 requests/minute total across all workers. With 3 workers at 1200ms delay, that is:
- 3 workers × (1 request / 1200ms) = 2.5 requests/sec = 150 req/min

This is well above the stated 60 req/min limit, which explains the persistent 429 pressure. The fix: reduce base delay to 600ms and use a single worker, or keep 3 workers but raise the delay to 3000ms per worker. **Option B uses 600ms base with exponential backoff on 429, which allows the scraper to run faster in clear windows and self-throttle under rate-limit pressure.**

> [!note] Practical note for Presten
> The 600ms delay is valid only if you also reduce to 1 worker, OR if GotSport's 60 req/min is per-worker (not total). Test this: run 1 worker at 600ms for 1,000 teams. If no 429s: the limit is per-worker and you can keep 3 workers. If 429s appear within the first 100 requests: limit is global, switch to 1 worker at 1200ms with the backoff spec below.

---

## Implementation Spec

### Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| Base inter-request delay | **600ms** | Above the 60 req/min threshold per worker |
| On 429 — initial backoff | **1s** | First retry starts at 1 second |
| Backoff multiplier | **2x per retry** | Exponential doubling |
| Backoff cap | **32s** | Max any single retry wait |
| Jitter | **±20% of wait time** | Prevents thundering herd across 3 workers |
| Max retries per team | **5** | Before moving to DLQ |
| On 403 | Stop entire run | Auth wall — hard stop, log `AUTH_WALL_DETECTED` |
| On 404 | Skip, no retry | Genuine no-data |

**Retry delay sequence (with ±20% jitter applied):**

| Attempt | Base Wait | With Max +20% Jitter |
|---------|-----------|----------------------|
| 1 | 1s | ~1.2s |
| 2 | 2s | ~2.4s |
| 3 | 4s | ~4.8s |
| 4 | 8s | ~9.6s |
| 5 | 16s | ~19.2s |
| 6 (DLQ) | — | Exhausted |

### Code Changes to Apply to `crawl-gotsport-api-parallel.js`

#### 1. Replace hardcoded delay constant

```javascript
// BEFORE
const INTER_REQUEST_DELAY_MS = 1200;

// AFTER
const INTER_REQUEST_DELAY_MS = 600;    // Option B — reduced base, backoff handles spikes
const BACKOFF_BASE_MS = 1000;          // First retry wait
const BACKOFF_CAP_MS = 32000;          // Max retry wait
const BACKOFF_JITTER_PCT = 0.20;       // ±20%
const MAX_RETRIES_PER_TEAM = 5;
```

#### 2. Replace the fetch call with `fetchWithBackoff`

```javascript
// BEFORE — no retry
async function fetchTeamMatches(teamId) {
  const response = await fetch(
    `${BASE_URL}/teams/${teamId}/matches`,
    { headers: { 'Accept': 'application/json' } }
  );
  if (!response.ok) {
    logger.debug({ teamId, status: response.status });
    return null;
  }
  return response.json();
}

// AFTER — Option B exponential backoff with jitter
async function fetchWithBackoff(teamId, attempt = 0) {
  const response = await fetch(
    `${BASE_URL}/teams/${teamId}/matches`,
    { headers: { 'Accept': 'application/json' } }
  );

  if (response.status === 403) {
    logger.error({ event: 'AUTH_WALL_DETECTED', teamId, status: 403 });
    process.exit(1);  // Hard stop — GotSport added auth requirement
  }

  if (response.status === 404) {
    logger.info({ event: 'TEAM_NOT_FOUND', teamId });
    return null;
  }

  if ((response.status === 429 || response.status === 503) && attempt < MAX_RETRIES_PER_TEAM) {
    const baseWait = Math.min(BACKOFF_BASE_MS * Math.pow(2, attempt), BACKOFF_CAP_MS);
    const jitterRange = baseWait * BACKOFF_JITTER_PCT;
    const jitter = (Math.random() * 2 - 1) * jitterRange;  // ±20%
    const wait = Math.round(baseWait + jitter);
    logger.warn({
      event: 'API_RETRY',
      teamId,
      attempt: attempt + 1,
      max_attempts: MAX_RETRIES_PER_TEAM,
      wait_ms: wait,
      status_code: response.status
    });
    await sleep(wait);
    return fetchWithBackoff(teamId, attempt + 1);
  }

  if (response.status === 429 || response.status === 503) {
    // Exhausted retries — caller will write to DLQ
    throw new Error(`Rate limited after ${MAX_RETRIES_PER_TEAM} retries: HTTP ${response.status}`);
  }

  if (!response.ok) {
    throw new Error(`Unexpected HTTP ${response.status} for team ${teamId}`);
  }

  return response.json();
}
```

#### 3. Add per-day backoff summary logging

Add this after each day's run completes (or at the `SYNC_RUN_SUMMARY` point from pipeline-fixes):

```javascript
// Track these counters at module scope, reset each run
let total429s = 0;
let totalBackoffEvents = 0;
let maxWaitHit = 0;
let retriesExhausted = 0;

// In the API_RETRY log path, increment:
total429s++;
totalBackoffEvents++;
if (wait > maxWaitHit) maxWaitHit = wait;

// In the exhausted-retries path:
retriesExhausted++;

// At SYNC_RUN_SUMMARY, add:
logger.info({
  event: 'BACKOFF_DAILY_SUMMARY',
  total_429s: total429s,
  total_backoff_events: totalBackoffEvents,
  max_wait_hit_ms: maxWaitHit,
  retries_exhausted: retriesExhausted
});
```

---

## Logging Reference — What New Log Lines Look Like

```json
// Rate limit hit, retry 1 of 5
{"event":"API_RETRY","teamId":84712,"attempt":1,"max_attempts":5,"wait_ms":1140,"status_code":429}

// Rate limit hit again, retry 3 of 5
{"event":"API_RETRY","teamId":84712,"attempt":3,"max_attempts":5,"wait_ms":4680,"status_code":429}

// Auth wall — process exits after this
{"event":"AUTH_WALL_DETECTED","teamId":91234,"status":403}

// End of run summary with backoff stats
{"event":"BACKOFF_DAILY_SUMMARY","total_429s":34,"total_backoff_events":34,"max_wait_hit_ms":19200,"retries_exhausted":3}
```

---

## How to Verify the Fix Is Working

1. After first run with new code: check for `API_RETRY` events in the log. If none appear on a day with active crawling, either the rate limit is not being triggered (good) or the 429s are being silently eaten (check: the old fetch call must be replaced, not left in place).

2. `BACKOFF_DAILY_SUMMARY` at end of run: `total_429s` should be non-zero during peak crawl hours if GotSport is still rate-limiting. `retries_exhausted` should be low (target <1% of teams in run).

3. No `SYNC_RUN_SUMMARY` showing teams moving from `teams_fetched_success` to `teams_dlq` due to 429s alone. Backoff should recover almost all of them.

4. If `AUTH_WALL_DETECTED` ever fires: GotSport has added authentication. Stop scraping immediately and notify Presten.

---

## After Implementation: Stale Team Backfill

Once the backoff is deployed and tested (at least one clean run with no unexpected DLQ entries from 429s):

**Phase 1: Backfill the 540 rate-limit-casualty teams**

```bash
# Run targeted backfill — 540 teams only
# Get team IDs from stale-teams-2026-04-21.csv, filter to rate-limit-casualty category
GSAPI_TEAM_IDS=$(cat Reports/stale-teams-rate-limit-540.txt) \
  node crawl-gotsport-api-parallel.js

# Verify: after run, check how many of the 540 now have last_synced_at within 24h
```

**Phase 2: Backfill the 112 confirmed-stale teams (Sub-Cats A/B/C)**

Only after Phase 1 is confirmed stable. These 112 teams are in `Logs/dlq/dlq-test-2026-04-23.jsonl` plus the 101 not in the test subset. Run same targeted approach.

**Do NOT backfill all 847 at once.** Run Phase 1 first, assess, then Phase 2.

---

## Files Affected

| File | Change |
|------|--------|
| `crawl-gotsport-api-parallel.js` | Replace `INTER_REQUEST_DELAY_MS` constant; replace `fetchTeamMatches()` with `fetchWithBackoff()`; add backoff counters; add `BACKOFF_DAILY_SUMMARY` to run summary |
| `Logs/dlq/` | DLQ files written here (pre-existing from pipeline-fixes deployment) |

---

## References

- `Reports/pipeline-fixes-2026-04-23.md` — Fix 1 (superseded by this spec for delay and retry parameters)
- `Reports/stale-teams-unexplained-2026-04-22.md` — the 847-team investigation
- `10 - Agents/FORGE/Queue/pending-2026-04-21-rate-limit.md` — Option B original spec
- [[GotSport API Scraper]] — scraper overview

---

*FORGE — 2026-04-24*
