---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-25
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/NPL-DPL Ingestion Verification Query — April 2026.md"
---

# Task: NPL/DPL Ingestion Recency Verification Document

## Context

NPL and DPL are ranked #5 in the Source Coverage Priority Matrix (Score 7 — same as USYS). Both are marked "Active, unverified" — meaning FORGE has never confirmed whether current-season games are actually being ingested. The Source Coverage Matrix already contains the core query sketch:

```sql
SELECT event_tier, MAX(created_at) AS last_game_ingested, COUNT(*) AS game_count
FROM games
WHERE event_tier IN ('npl', 'dpl')
GROUP BY event_tier;
```

This task converts that sketch into a complete, runnable verification document — same structure as `Infrastructure/ECNL RL Staleness Verification Query — April 2026.md`.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/NPL-DPL Ingestion Verification Query — April 2026.md`

The document must contain:

1. **Context** — what NPL and DPL are, their estimated volume and tier, why verifying ingestion recency matters before May 9 DSS prep.

2. **Primary verification query** — finalized SQL. Include a fallback filter using `event_name ILIKE` in case `event_tier` values differ from what's expected (same multi-condition pattern as ECNL RL query).

3. **PASS / YELLOW / FAIL thresholds:**
   - PASS: `days_since_most_recent` ≤ 30 for both NPL and DPL, and combined game count ≥ 2,000 for current season
   - YELLOW: 31–60 days on either, OR 1,000–1,999 combined games → flag in next SENTINEL briefing, no immediate action
   - FAIL: > 60 days on either, OR < 1,000 combined games, OR no current-season row → escalate to SENTINEL immediately. Do not proceed with any NPL/DPL-adjacent decisions until resolved.

4. **Recommended run timing** — when Presten should run it (next psql session, ideally before April 29).

5. **Downstream dependencies** — note that Event Strength Phase 1 classifications will use NPL/DPL team ratings as inputs. If NPL/DPL ingestion is stale, Event Strength Phase 1 `strength_class` assignments for NPL/DPL events will be incorrect. This is a gate condition for Phase 1 accuracy.

## Quality Standard

Match the structure and tone of `Infrastructure/ECNL RL Staleness Verification Query — April 2026.md`. Presten should be able to open this document, run the query in one psql session, and know exactly what to do with the result.

## Notes

- Do not write a separate document for NPL and DPL — one document covers both, since the query checks both in a single GROUP BY.
- If the current-season filter is uncertain (fiscal year? academic year?), note that uncertainty and include a version of the query that checks last 365 days as a fallback.
