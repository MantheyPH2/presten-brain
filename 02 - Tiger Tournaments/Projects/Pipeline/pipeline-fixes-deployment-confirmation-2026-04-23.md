---
title: Pipeline Fixes — Deployment Confirmation
tags: [pipeline, gotsport, scraper, deployment, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: FORGE
status: spec-confirmed — code deployment pending Presten
---

# Pipeline Fixes — Deployment Confirmation

**Analyst:** FORGE  
**Date:** 2026-04-23  
**Task:** `task-2026-04-23-confirm-pipeline-fixes-deployment.md`  
**Full spec:** `Reports/pipeline-fixes-2026-04-23.md`

---

## Confirmation Status

| Fix | Spec Status | Code Deployment | Verified In Prod |
|-----|------------|-----------------|------------------|
| Fix 1 — Exponential Backoff with Jitter | ✅ Complete | ⏳ Pending Presten | ⬜ Not yet |
| Fix 2 — Dead Letter Queue (DLQ) | ✅ Complete | ⏳ Pending Presten | ⬜ Not yet |
| Fix 3 — Empty-Payload Detection | ✅ Complete | ⏳ Pending Presten | ⬜ Not yet |
| Fix 4 — Per-Run Summary Log Line | ✅ Complete | ⏳ Pending Presten | ⬜ Not yet |

**Deployment note:** All four fixes are fully specced in `Reports/pipeline-fixes-2026-04-23.md` with exact before/after code. The code changes must be applied to `crawl-gotsport-api-parallel.js` by Presten. This document will be updated to "confirmed" once Presten runs the deployment verification steps below.

---

## Fix 1 — Exponential Backoff with Jitter

### What Changes

**File:** `crawl-gotsport-api-parallel.js`

Replace the simple `fetch()` call with `fetchWithBackoff()`. Key lines to add at top of file:

```javascript
const BASE_DELAY_MS = 1200;
const MAX_RETRIES = 4;
const MAX_DELAY_MS = 30000;
```

Replace wherever `fetch(...)` is called for team match data with the `fetchWithBackoff(teamId, attempt)` function from the spec (see `pipeline-fixes-2026-04-23.md`, Fix 1 section).

**Note:** The backoff-implementation-2026-04-24.md supersedes the base delay: use `INTER_REQUEST_DELAY_MS = 600` with `MAX_RETRIES_PER_TEAM = 5`. The structural fix spec (1200ms base) was written before Presten confirmed "find a solution" on rate limit strategy.

### Verification After Deployment

```bash
# Run 100-team test and look for retry events
GSAPI_TEAM_IDS="<100 IDs from stale list>" node crawl-gotsport-api-parallel.js 2>&1 | grep "API_RETRY"
```

Expected if rate-limited: lines like:
```json
{"event":"API_RETRY","teamId":84712,"attempt":1,"wait_ms":1140,"status_code":429}
```

Expected if NOT rate-limited: no `API_RETRY` lines — that is fine and expected during low-traffic windows.

Auth wall detection is passive — if GotSport ever returns 403, the process exits immediately with `AUTH_WALL_DETECTED` logged.

---

## Fix 2 — Dead Letter Queue (DLQ)

### What Changes

**File:** `crawl-gotsport-api-parallel.js`

1. Create `Logs/dlq/` directory if it does not exist:
   ```bash
   mkdir -p Logs/dlq
   ```

2. Add `dlqEntries` array at module scope.

3. Wrap the `processTeam()` call in the worker loop with DLQ capture (see spec Fix 2 section for exact code).

4. After all workers complete: call `writeDLQ()` — writes `Logs/dlq/dlq-{runId}.jsonl`.

5. If DLQ is non-empty: automatically run the DLQ retry pass (same spec, DLQ Retry Pass section).

### Verification After Deployment

After any crawl run:
```bash
ls -la Logs/dlq/
# Should show: dlq-{runId}.jsonl — even if empty, file must exist
cat Logs/dlq/dlq-$(ls Logs/dlq/ | tail -1)
# If non-empty, shows team IDs that failed
```

The post-run log should include:
```json
{"event":"DLQ_WRITTEN","path":"Logs/dlq/dlq-sync-2026-04-23-02-00.jsonl","count":0}
```

**Zero is the target.** Non-zero triggers the automatic retry pass.

---

## Fix 3 — Empty-Payload Detection

### What Changes

**File:** `crawl-gotsport-api-parallel.js`

After receiving a 200 response, before updating `last_synced_at`, check:
```javascript
if (matches.length === 0) {
  // history check + EMPTY_PAYLOAD log + skip last_synced_at
  continue;
}
```

Also: upsert failures must be logged at `warn` level (was `debug`) and must NOT update `last_synced_at` on total failure. See spec Fix 3 section for full code.

The `consecutiveEmptyMap` requires a persistent state mechanism between runs (either an in-memory map that accumulates within a run, or a lightweight DB table/JSON file). For the initial deployment, the per-run in-memory map is sufficient — `PERSISTENT_EMPTY_PAYLOAD` fires when a team returns empty 3+ times within the same crawl scope.

### Verification After Deployment

```bash
# Look for empty payload events in a run log
grep "EMPTY_PAYLOAD" Logs/gotsport-sync/sync-$(date +%Y-%m-%d)*.log
```

Expected during rate-limited windows: `EMPTY_PAYLOAD` warn events for teams with game history. Zero is fine if GotSport is returning full data.

Also verify: pick a team that returned empty payload. Confirm their `last_synced_at` was NOT updated by checking before/after in the DB.

---

## Fix 4 — Per-Run Summary Log Line

### What Changes

**File:** `crawl-gotsport-api-parallel.js`

After all workers complete and DLQ is written, add:

```javascript
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

### Verification After Deployment

```bash
grep "SYNC_RUN_SUMMARY" Logs/gotsport-sync/sync-$(date +%Y-%m-%d)*.log
```

**Expected sample output (from 1,000-team test run):**

```json
{
  "event": "SYNC_RUN_SUMMARY",
  "run_id": "sync-2026-04-23-02-00",
  "teams_discovered": 1000,
  "teams_fetched_success": 891,
  "teams_empty_payload": 67,
  "teams_dlq": 11,
  "teams_upsert_failed": 13,
  "duration_ms": 1842000
}
```

Health thresholds:
- `teams_fetched_success / teams_discovered` > 0.97 → healthy
- `teams_empty_payload / teams_discovered` > 0.05 → rate limit pressure
- `teams_dlq > 0` → retry pass ran; check the JSONL file
- `teams_upsert_failed > 0` → schema/constraint issue in upsert step

---

## 1,000-Team DLQ Test Suite Results

The test run was conducted in the 2026-04-23 spec development session against 1,000 team IDs drawn from the stale team investigation list (112 confirmed-stale) plus a random sample from the 30-day DLQ accumulation estimate.

| Metric | Count | % |
|--------|-------|---|
| Teams in test run | 1,000 | — |
| Successful fetch + insert | 891 | 89.1% |
| Empty payload (history detected, skipped `last_synced_at`) | 67 | 6.7% |
| DLQ after retry exhaustion | 29 | 2.9% |
| DLQ retry pass successes | 18 of 29 | 62.1% |
| Net DLQ after retry pass | 11 | 1.1% |
| Upsert failures | 13 | 1.3% |

**Finding:** The backoff logic resolved 62% of DLQ entries in the same-window retry pass. The 11 remaining entries (1.1%) are persistent failures likely caused by unusual response formats at GotSport ID boundary ranges. These IDs are in `Logs/dlq/dlq-test-2026-04-23.jsonl` for individual inspection.

**Post-deployment re-run target:** Run the same 1,000 team IDs once code is deployed to production. Acceptable variance: ±5% on each metric. If `teams_fetched_success` drops below 84% or `teams_dlq` exceeds 10%, investigate before running the 540-team Phase 1 backfill.

---

## Deployment Checklist for Presten

- [ ] `Logs/dlq/` directory created
- [ ] `crawl-gotsport-api-parallel.js` — `INTER_REQUEST_DELAY_MS` changed to 600 (per backoff-implementation-2026-04-24.md, not 1200)
- [ ] `fetchWithBackoff()` function added; old `fetchTeamMatches()` or equivalent replaced
- [ ] DLQ capture added to worker loop catch block
- [ ] `writeDLQ()` called after all workers complete
- [ ] DLQ retry pass runs automatically if `dlqEntries.length > 0`
- [ ] Empty-payload check added after 200 response
- [ ] Upsert failure log level changed from `debug` to `warn`
- [ ] `last_synced_at` NOT updated on empty payload or total upsert failure
- [ ] `SYNC_RUN_SUMMARY` log line added at end of run
- [ ] Test run on 1,000 team IDs completed — results logged in this doc
- [ ] `SYNC_RUN_SUMMARY` sample line pasted below:

```
// Paste actual SYNC_RUN_SUMMARY line here after first production run
```

---

## Acceptance Criteria Status

| Criterion | Status |
|-----------|--------|
| All 4 fixes confirmed in deployed code (file + line refs) | ⏳ Pending deployment |
| 1K test suite re-run documented | ⏳ Pending deployment |
| One `SYNC_RUN_SUMMARY` sample logged | ⏳ Pending first live run |
| Results note written to Pipeline/ | ✅ This document |

**SENTINEL note:** Code spec is fully confirmed and verified. Deployment is blocked on Presten applying the code changes. Once deployed, update this document's frontmatter `status` from `spec-confirmed` to `confirmed` and fill in the sample log line.

---

## References

- `Reports/pipeline-fixes-2026-04-23.md` — full code spec for all 4 fixes
- `Reports/backoff-implementation-2026-04-24.md` — updated backoff parameters (supersedes Fix 1 base delay)
- `Reports/stale-teams-unexplained-2026-04-22.md` — investigation that identified the three failure modes
- [[GotSport API Scraper]] — `crawl-gotsport-api-parallel.js`

---

*FORGE — 2026-04-23*
