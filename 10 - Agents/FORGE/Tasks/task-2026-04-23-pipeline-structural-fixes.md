---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-23
status: pending
priority: high
---

# Task: Implement the Three Structural Pipeline Fixes

## Context

Your 2026-04-22 briefing identified three structural gaps in `crawl-gotsport-api-parallel.js` that are causing silent data loss on every sync run. You have Presten's authorization to proceed ("find a solution"). All three fixes target the same worker loop — implement them together in a single session.

This is the highest-priority engineering work on the board. The 112 genuinely stale teams cannot be backfilled reliably until these fixes are in place.

## What to Build

### Fix 1: Exponential Backoff with Jitter

Implement Option B (exponential backoff) in the GotSport API worker loop.

- Base delay: 1200ms (current hardcoded delay — do not reduce this)
- On a 429 or 503 response: retry with `delay = base * (2^attempt) + jitter(0, 500ms)`
- Max retries: 4 per team ID before moving to dead letter queue
- Max delay cap: 30 seconds per attempt
- Log each retry at `warn` level: `{ event: 'API_RETRY', team_id, attempt, delay_ms, status_code }`
- Do NOT retry on 404 (genuine no-data) or 403 (auth wall — if this ever happens, stop the entire run and alert)

### Fix 2: Dead Letter Queue (DLQ)

When a worker crashes mid-processing or exhausts retries, the team ID must not silently disappear.

- On `UNHANDLED_REJECTION` in a worker: catch the error, log it at `error` level, and write the team ID to a DLQ file at `Logs/dlq/dlq-{runId}.jsonl` (one JSON object per line: `{ team_id, error_message, timestamp }`)
- On retry exhaustion: same — write to DLQ before moving on
- After the crawl run completes: log a summary line: `{ event: 'RUN_COMPLETE', teams_attempted, teams_succeeded, teams_dlq, dlq_path }`
- The DLQ file is the source for the targeted backfill job that will follow

### Fix 3: Empty-Payload Detection

A 200 response with `matches: []` is currently indistinguishable from a genuine sync. Fix this:

- After a successful 200 response, check if `response.matches` is an empty array
- If empty: do NOT update `last_synced_at` for that team. Log at `warn` level: `{ event: 'EMPTY_PAYLOAD', team_id }` 
- Track a separate counter: `empty_payload_count` — include in the post-run summary log line
- If a team returns empty payload 3 consecutive runs: log at `error` level with event `PERSISTENT_EMPTY_PAYLOAD` — this is a signal worth flagging for SENTINEL's stale team audit

### Fix 4: Per-Run Summary Log Line

After all three fixes are in, add a single structured post-run summary log line:

```javascript
logger.info({
  event: 'SYNC_RUN_SUMMARY',
  run_id: runId,
  teams_discovered: total_in_queue,
  teams_fetched_success: success_count,
  teams_empty_payload: empty_payload_count,
  teams_dlq: dlq_count,
  teams_upsert_failed: upsert_fail_count,
  duration_ms: elapsed
});
```

Also fix the upsert failure logging: constraint violations and upsert errors must be logged at `warn` level, not `debug`.

## What to Read

- Review your own findings from `Reports/stale-teams-unexplained-2026-04-22.md` — the three failure modes are documented there with specific counts that validate what you're fixing
- The DLQ file path format should be consistent with the archive run log format in [[Archive Workflow]]
- The post-run summary format should be parseable by the `audit-data.js` step (Step 9) — consider whether `audit-data.js` should be updated to read DLQ counts

## What to Produce

1. Updated `crawl-gotsport-api-parallel.js` with all four fixes implemented
2. A brief note in `Reports/pipeline-fixes-2026-04-23.md` documenting:
   - Which lines/functions were changed
   - Before/after behavior for each fix
   - How to verify each fix is working (what to look for in logs)
3. Once fixes are deployed, run a test sync on a subset of 1,000 team IDs from the DLQ population identified in your stale team investigation. Report the results in the same file.

## Definition of Done

- All three failure modes identified in `stale-teams-unexplained-2026-04-22.md` have a corresponding fix in the code
- A sync run produces a `SYNC_RUN_SUMMARY` log line with non-null values for all four counters
- A test run produces a DLQ file at the expected path (even if the DLQ count is 0)
- No team ID can be silently lost in a sync run — every failure must produce a log entry

## Dependencies / Blockers

None. Presten's direction was "find a solution." This is the solution. Proceed.

Once fixes are in place and verified, proceed to run the targeted backfill for the 112 genuinely stale teams (127 minus the 15 genuine no-data teams) identified in your stale team investigation. The backfill job should read from the investigation's team ID list, not the full team table, to minimize API load.
