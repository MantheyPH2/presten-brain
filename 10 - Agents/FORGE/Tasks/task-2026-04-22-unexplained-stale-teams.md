---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-22
status: pending
priority: high
---

# Task: Investigate 127 Unexplained Stale Teams

## Context

Your stale team audit (2026-04-21) identified 847 teams with data older than 72 hours. You correctly attributed:
- ~540 to the GotSport rate limit degradation
- ~180 to de-prioritized tournament teams still marked `active`
- **~127 to unknown causes — this is what you must investigate now**

The 127 unexplained cases were deferred in your briefing. SENTINEL is marking this as high priority. Unexplained data gaps in a pipeline are rarely isolated — they usually indicate a class of failure that affects more records than the visible symptom count suggests.

## Required Investigation

### Step 1: Isolate the 127 Teams
From your `Reports/stale-teams-2026-04-21.csv`, filter out the 540 rate-limit casualties (teams last failed during a 14:00 UTC window with 429 errors) and the 180 de-prioritized tournament teams. What remains is your 127.

### Step 2: Log Trace by Team ID
For each of the 127 teams, trace through the scraper logs (`Logs/gotsport-sync/`) to find the last successful sync and what happened after. Look for:
- Silent 200 responses with empty or partial payloads (GotSport sometimes returns HTTP 200 with an empty `matches` array for rate-limited secondary calls)
- Teams that appear in the team discovery step (`/api/v1/team_ranking_data`) but whose match fetch (`/api/v1/teams/{id}/matches`) is never logged as completing
- Any pattern in team IDs — are these all in a specific numerical range that suggests they were skipped or dequeued?

### Step 3: Identify the Root Cause Pattern
Classify the 127 into sub-categories:
- **Empty payload / silent rate limit:** GotSport returned 200 with no data
- **Queue dropout:** Team IDs entered the worker queue but match fetch was never logged as attempted
- **Parsing failure:** Match fetch succeeded but ingestion step failed (check for silent exceptions in the upsert step)
- **Genuine no-data teams:** Teams that exist in GotSport but have never had a match (edge case but possible for new club registrations)

### Step 4: Determine If This Indicates a Systemic Issue
If the root cause is queue dropout or parsing failure, estimate how many additional teams beyond these 127 may have been silently affected in prior sync windows (check the last 30 days of logs, not just the last 7 days).

## Deliverable

Write your findings to `Reports/stale-teams-unexplained-2026-04-22.md` with:
1. Breakdown of the 127 by root cause sub-category
2. Whether this indicates a systemic issue with scope beyond 127 teams
3. Recommended fix or mitigation for each sub-category
4. Any changes needed to the sync logging to catch this class of failure automatically in the future

Post a summary to your briefing for 2026-04-22 and flag SENTINEL if the systemic scope is larger than 500 teams.

## Dependencies

This task does not depend on the rate limit strategy decision (that only affects how you approach the 540 rate-limit casualties). You can investigate the 127 independently.

## References

- `Reports/stale-teams-2026-04-21.csv` — your stale team list
- `Logs/gotsport-sync/` — sync logs for log tracing
- [[GotSport API Scraper]] — `crawl-gotsport-api-parallel.js` — review the worker queue and upsert logic
- [[Data Pipeline]] — pipeline step context
