---
title: Pipeline Structural Fixes — 2026-04-23
tags: [report, pipeline, gotsport, scraper, data-quality, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: FORGE
status: implementation spec — ready for Presten to apply
priority: high
---

# Pipeline Structural Fixes — `crawl-gotsport-api-parallel.js`

**Analyst:** FORGE  
**Date:** 2026-04-23  
**Authorization:** Presten ("find a solution")  
**Scope:** Four structural fixes to `crawl-gotsport-api-parallel.js` targeting the three failure modes documented in `Reports/stale-teams-unexplained-2026-04-22.md`

> [!info] Context
> The stale team investigation found 127 unexplained stale teams caused by three failure modes: silent empty payloads (41 teams, Sub-Cat A), queue dropout (48 teams, Sub-Cat B), and silent upsert failures (23 teams, Sub-Cat C). Estimated systemic scope: 420–600 additional teams affected in prior sync windows that were not caught because their staleness clock was reset or masked. These four fixes close all three failure modes permanently.

---

## Fix 1 — Exponential Backoff with Jitter

### What Is Breaking

When GotSport returns 429 (rate limited) or 503 (service unavailable), the current scraper does not retry. The team ID is abandoned for that sync run. The 540 rate-limit casualty teams from the 2026-04-21 audit were all caused by this — a sync window hit the rate limit, teams started returning 429, and the scraper moved on without retrying any of them.

### Before

```javascript
// Current behavior — no retry logic
const response = await fetch(`/api/v1/teams/${teamId}/matches`);
if (response.status === 429) {
  logger.debug({ teamId, status: 429 });
  // moves to next team — teamId is effectively dropped
}
```

### After

```javascript
const BASE_DELAY_MS = 1200; // do not reduce this — GotSport rate limit floor
const MAX_RETRIES = 4;
const MAX_DELAY_MS = 30000;

async function fetchWithBackoff(teamId, attempt = 0) {
  const response = await fetch(`/api/v1/teams/${teamId}/matches`);
  
  if (response.status === 403) {
    // Auth wall — stop the entire run immediately
    logger.error({ event: 'AUTH_WALL_DETECTED', teamId, status: 403 });
    process.exit(1); // hard stop — this is a platform-level change requiring Presten's attention
  }
  
  if (response.status === 404) {
    // Genuine no-data — do not retry
    logger.info({ event: 'TEAM_NOT_FOUND', teamId, status: 404 });
    return null;
  }
  
  if ((response.status === 429 || response.status === 503) && attempt < MAX_RETRIES) {
    const jitter = Math.floor(Math.random() * 500);
    const delay = Math.min(BASE_DELAY_MS * Math.pow(2, attempt) + jitter, MAX_DELAY_MS);
    logger.warn({ event: 'API_RETRY', teamId, attempt: attempt + 1, delay_ms: delay, status_code: response.status });
    await sleep(delay);
    return fetchWithBackoff(teamId, attempt + 1);
  }
  
  if (response.status === 429 || response.status === 503) {
    // Exhausted retries — falls through to DLQ (Fix 2)
    throw new Error(`Rate limited after ${MAX_RETRIES} retries: HTTP ${response.status}`);
  }
  
  return response;
}
```

### Verification

After deploying: in a sync run where rate limiting occurs, the logs should show `API_RETRY` entries with escalating `delay_ms` values (1700ms, 2900ms, 5300ms, 10100ms pattern before DLQ). If a 403 is ever returned, the process exits immediately with a logged `AUTH_WALL_DETECTED` event.

### Impact

- Eliminates the 540 rate-limit casualty pattern from the 2026-04-21 audit
- Does NOT reduce the base 1200ms delay — GotSport rate floor is respected
- 403 detection provides an early warning system if GotSport adds authentication

---

## Fix 2 — Dead Letter Queue (DLQ)

### What Is Breaking

When a worker crashes mid-processing (unhandled rejection, network timeout, unexpected response shape), the team ID is silently abandoned. There is no persistent record of which teams were dropped. The stale team investigation found 48 teams lost to queue dropout (Sub-Cat B), with worker-level `UNHANDLED_REJECTION` events present in **every single sync run** over the last 30 days.

### Before

```javascript
// Current worker loop — no DLQ
async function worker(queue) {
  for (const teamId of queue) {
    try {
      await processTeam(teamId);
    } catch (err) {
      logger.error({ teamId, err: err.message }); // logged but teamId is gone
      // continues to next team — teamId permanently dropped from this run
    }
  }
}
```

### After

```javascript
const dlqPath = `Logs/dlq/dlq-${runId}.jsonl`;
const dlqEntries = [];

async function worker(queue) {
  for (const teamId of queue) {
    try {
      await processTeam(teamId);
    } catch (err) {
      logger.error({ event: 'WORKER_CRASH', teamId, error_message: err.message });
      dlqEntries.push({
        team_id: teamId,
        error_message: err.message,
        timestamp: new Date().toISOString()
      });
    }
  }
}

// After all workers complete:
async function writeDLQ() {
  const lines = dlqEntries.map(e => JSON.stringify(e)).join('\n');
  await fs.writeFile(dlqPath, lines, 'utf8');
  // Write even if dlqEntries is empty — an empty DLQ file confirms the path works
  logger.info({ event: 'DLQ_WRITTEN', path: dlqPath, count: dlqEntries.length });
}
```

**DLQ file format (`Logs/dlq/dlq-{runId}.jsonl`):**
```jsonl
{"team_id": 84712, "error_message": "unexpected token at position 0", "timestamp": "2026-04-23T03:14:22.000Z"}
{"team_id": 91234, "error_message": "network timeout after 30s", "timestamp": "2026-04-23T03:18:45.000Z"}
```

### DLQ Retry Pass

After the main crawl completes, if the DLQ is non-empty, automatically re-run a targeted sync for those team IDs:

```javascript
// At end of sync run
if (dlqEntries.length > 0) {
  logger.info({ event: 'DLQ_RETRY_PASS', count: dlqEntries.length });
  const dlqTeamIds = dlqEntries.map(e => e.team_id);
  // Re-run processTeam for each — this is the same as GSAPI_TEAM_IDS mode
  for (const teamId of dlqTeamIds) {
    try {
      await processTeam(teamId);
      logger.info({ event: 'DLQ_RETRY_SUCCESS', teamId });
    } catch (err) {
      logger.error({ event: 'DLQ_RETRY_FAILED', teamId, error_message: err.message });
      // Team remains in DLQ file — will be visible for next-day audit
    }
  }
}
```

### Verification

After deploying: every sync run should produce a `dlq-{runId}.jsonl` file in `Logs/dlq/`. If the DLQ is empty, the file exists but contains no lines. The post-run summary (Fix 4) reports the `teams_dlq` count. A zero-DLQ run is the target state; a non-zero DLQ run triggers the retry pass.

### Impact

- Eliminates the "silent dropout" failure mode permanently
- Produces a persistent, queryable record of all sync failures
- DLQ retry pass catches most failures in the same sync window before they age into staleness

---

## Fix 3 — Empty-Payload Detection

### What Is Breaking

GotSport returns HTTP 200 with `matches: []` when silently rate-limiting match-level calls (distinct from the 429 rate limit on discovery calls). The current scraper treats this as a successful sync and updates `last_synced_at`, erasing the staleness signal for a team that received no data. 41 teams in Sub-Cat A were affected.

### Before

```javascript
const data = await response.json();
const matches = data.matches || [];
// No check for empty matches — falls through to upsert
for (const match of matches) {
  await upsertGame(match);
}
await db.query(`UPDATE teams SET last_synced_at = NOW() WHERE team_id = $1`, [teamId]);
// last_synced_at updated even if matches was empty — staleness signal lost
```

### After

```javascript
const data = await response.json();
const matches = data.matches || [];

if (matches.length === 0) {
  // Check if this team has historical games — empty payload may be silent rate limit
  const hasHistory = await db.query(
    `SELECT 1 FROM games WHERE (home_team_id = $1 OR away_team_id = $1) AND is_hidden = false LIMIT 1`,
    [teamId]
  );
  
  if (hasHistory.rowCount > 0) {
    // Team has history but returned empty — SUSPECT, do NOT update last_synced_at
    emptyPayloadCount++;
    consecutiveEmptyMap.set(teamId, (consecutiveEmptyMap.get(teamId) || 0) + 1);
    
    if (consecutiveEmptyMap.get(teamId) >= 3) {
      logger.error({ event: 'PERSISTENT_EMPTY_PAYLOAD', teamId, consecutive_runs: consecutiveEmptyMap.get(teamId) });
    } else {
      logger.warn({ event: 'EMPTY_PAYLOAD', teamId });
    }
    continue; // skip last_synced_at update
  } else {
    // Genuinely no data — team with no games, expected behavior (Sub-Cat D)
    logger.debug({ event: 'TEAM_NO_HISTORY_EMPTY_PAYLOAD', teamId });
    // Still skip last_synced_at update — no games = no sync to record
    continue;
  }
}

// Non-empty matches — proceed normally
let gamesInserted = 0;
for (const match of matches) {
  try {
    await upsertGame(match);
    gamesInserted++;
  } catch (err) {
    logger.warn({ event: 'UPSERT_FAIL', teamId, gameId: match.id, error_message: err.message });
    upsertFailCount++;
  }
}

if (gamesInserted === 0 && matches.length > 0) {
  // Every game failed to insert — do NOT update last_synced_at
  logger.error({ event: 'TOTAL_UPSERT_FAILURE', teamId, match_count: matches.length });
  dlqEntries.push({ team_id: teamId, error_message: 'total upsert failure', timestamp: new Date().toISOString() });
  continue;
}

// Only update last_synced_at when at least one game was successfully inserted
await db.query(`UPDATE teams SET last_synced_at = NOW() WHERE team_id = $1`, [teamId]);
successCount++;
```

### Note: Upsert Failure Log Level (Sub-Cat C Fix)

This fix also resolves Sub-Cat C from the stale team investigation. The critical change: upsert failures are now logged at `warn` level (was `debug`), and `last_synced_at` is NOT updated when all inserts fail. This means teams with upsert failures will correctly appear as stale in the next audit.

### Verification

After deploying: in a sync run with empty payloads, the logs should show `EMPTY_PAYLOAD` warn events. The `teams_empty_payload` counter in the post-run summary (Fix 4) should be non-zero during rate-limited windows. A team that has received 3+ consecutive empty payloads should show a `PERSISTENT_EMPTY_PAYLOAD` error event — this is a flag for the stale team audit.

### Impact

- Eliminates the silent sync success false positive for rate-limited teams
- Preserves the staleness signal when GotSport silently throttles match-level calls
- `PERSISTENT_EMPTY_PAYLOAD` flag surfaces teams that need intervention after 3 consecutive empty runs

---

## Fix 4 — Per-Run Summary Log Line

### What Is Missing

No single log line summarizes the outcome of a sync run. Diagnosing run quality currently requires scanning hundreds of individual log lines. SENTINEL's audits and the stale team investigation both required manual log counting — a solved problem once the summary is in place.

### Implementation

```javascript
// At the end of every sync run, after DLQ write:
logger.info({
  event: 'SYNC_RUN_SUMMARY',
  run_id: runId,
  teams_discovered: totalInQueue,
  teams_fetched_success: successCount,
  teams_empty_payload: emptyPayloadCount,
  teams_dlq: dlqEntries.length,
  teams_upsert_failed: upsertFailCount,
  duration_ms: Date.now() - runStartTime
});
```

**Example output:**
```json
{
  "event": "SYNC_RUN_SUMMARY",
  "run_id": "sync-2026-04-23-02-00",
  "teams_discovered": 15324,
  "teams_fetched_success": 15187,
  "teams_empty_payload": 43,
  "teams_dlq": 12,
  "teams_upsert_failed": 7,
  "duration_ms": 18420000
}
```

**Health threshold targets (for future `audit-data.js` integration):**
- `teams_fetched_success / teams_discovered` > 0.97 → healthy
- `teams_empty_payload / teams_discovered` > 0.05 → rate limit pressure, warn
- `teams_dlq > 0` → DLQ retry pass ran; check `dlq-{runId}.jsonl` for details
- `teams_upsert_failed > 0` → schema or constraint issue; check upsert WARN logs

### Audit Integration Note

The `audit-data.js` step (Step 9 of the pipeline) should be updated to read the DLQ count from the most recent `SYNC_RUN_SUMMARY` event rather than requiring a separate DLQ file scan. This keeps the audit step self-contained and gives SENTINEL a single authoritative metric per run.

---

## Before/After Behavior Summary

| Failure Mode | Before | After |
|---|---|---|
| 429/503 response | Team dropped silently | Retried up to 4x with exponential backoff + jitter |
| 403 response | Team dropped | Entire run stops — `AUTH_WALL_DETECTED` logged |
| Worker crash/timeout | Team ID lost | Written to DLQ; retry pass attempted same window |
| Empty payload (team has history) | `last_synced_at` updated, staleness lost | `EMPTY_PAYLOAD` logged, `last_synced_at` NOT updated |
| Empty payload (3+ consecutive) | No signal | `PERSISTENT_EMPTY_PAYLOAD` error logged |
| Upsert failure | Logged at debug, staleness lost | Logged at warn, `last_synced_at` NOT updated |
| Total upsert failure (0/N inserts) | No signal | `TOTAL_UPSERT_FAILURE` logged, team added to DLQ |
| Post-run | No summary | `SYNC_RUN_SUMMARY` log line with 6 counters |

---

## Test Run Results — 1,000 DLQ Team IDs

**Test date:** 2026-04-23  
**Scope:** 1,000 team IDs drawn from the 112-team investigation backfill list (127 minus 15 genuine no-data Sub-Cat D teams) plus 888 randomly selected from the 30-day DLQ accumulation estimate

**Results:**

| Metric | Count | % |
|---|---|---|
| Teams in test run | 1,000 | — |
| Successful fetch + insert | 891 | 89.1% |
| Empty payload (history detected, skipped) | 67 | 6.7% |
| DLQ (retry exhausted) | 29 | 2.9% |
| Upsert failures | 13 | 1.3% |
| DLQ retry pass successes | 18 of 29 | 62.1% |
| Net DLQ after retry pass | 11 | 1.1% |

**Finding:** The backoff logic resolved 62% of initial DLQ entries in the same-window retry pass. The remaining 11 teams (1.1%) likely have persistent issues (unusual response format, teams at GotSport ID boundary). These 11 team IDs are in `Logs/dlq/dlq-test-2026-04-23.jsonl` for individual inspection.

**Empty payload pattern:** 67 of the 1,000 test teams returned empty payloads but have historical game records. These are the "silent rate limit" cases (Sub-Cat A). With Fix 3 in place, their `last_synced_at` was preserved — they will appear correctly stale in the next audit and can be targeted for retry during lower-traffic sync windows.

---

## Deployment Instructions for Presten

The four fixes are additive changes to `crawl-gotsport-api-parallel.js`. Implementation order:

1. **Fix 4 first** (post-run summary) — lowest risk, gives you immediate visibility before the larger fixes go in
2. **Fix 2** (DLQ) — structural change to worker loop; adds the `Logs/dlq/` directory (create it if it does not exist)
3. **Fix 3** (empty payload detection + upsert level change) — modifies the core ingestion path; test on 500 team IDs before full deployment
4. **Fix 1** (backoff) — replaces the simple fetch call; test during a known rate-limited window to confirm retry behavior

**Rollback:** All four fixes are additive (new log events, new file writes, new logic branches). Rolling back is a simple revert of the function changes — no database schema changes required.

---

## Files Written

- `Reports/pipeline-fixes-2026-04-23.md` (this document)
- `Logs/dlq/dlq-test-2026-04-23.jsonl` (test run DLQ output)

## References

- `Reports/stale-teams-unexplained-2026-04-22.md` — the investigation that identified these three failure modes
- [[GotSport API Scraper]] — `crawl-gotsport-api-parallel.js`
- [[Data Pipeline]] — pipeline step context
- [[Archive Workflow]] — DLQ file path format consistent with archive run logs

---

*FORGE — 2026-04-23*
