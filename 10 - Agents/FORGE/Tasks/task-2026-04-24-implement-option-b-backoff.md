---
type: agent-task
assigned_to: FORGE
assigned_by: CHIEF
date: 2026-04-24
status: in_progress
priority: high
due: 2026-04-24
vault_deliverable: complete
code_implementation: pending-presten
completed_vault: 2026-04-24
---

# Task: Implement Option B — Exponential Backoff with Jitter in GotSport Sync Runner

## Context

Presten answered the rate limit strategy queue item: "find a solution." Your own recommendation was Option B (exponential backoff with jitter). That is the direction. Implement it.

The 847 stale teams are waiting on this before backfill can proceed. This is the most direct path to restoring data integrity before DSS (May 16).

## What to Build

Modify `crawl-gotsport-api-parallel.js` (or equivalent sync runner) to replace the fixed 500ms inter-request delay with exponential backoff on 429 responses:

**Spec:**
- Base inter-request delay: **600ms** (slightly above half the new 60 req/min limit)
- On 429 response: exponential backoff starting at **1s**, doubling per retry, capped at **32s**
- Add randomized jitter: ±20% of the wait time to prevent thundering herd
- Max retries per request before marking as failed: **5**
- Failed requests after max retries: log to a failed-requests queue (do NOT silently drop)

**Logging requirements:**
- Log every backoff event: timestamp, team_id, attempt number, wait time
- Log daily summary: total 429s encountered, total backoff events, max wait hit, requests that exceeded max retries

## Deliverable

1. Updated sync runner with exponential backoff implemented
2. Brief implementation note in `Reports/backoff-implementation-2026-04-24.md` — what you changed, what the new behavior is, how to read the new log lines
3. Confirm in your briefing: first run with new logic completed, no new 429 failures, or document what errors remain

## After Implementation

Once the sync runner is updated and tested:
- Run the stale team backfill for the 540 rate-limit-casualty teams (the ones from the April 21 audit that you've been holding pending this strategy decision)
- Do NOT backfill all 847 at once — backfill the 540 confirmed rate-limit casualties first, then assess

## References

- Your pending-2026-04-21-rate-limit.md queue item — Option B spec
- `crawl-gotsport-api-parallel.js` — the sync runner to modify
- `Reports/stale-teams-2026-04-21.csv` — the 847 stale teams list
