---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stale Team Backfill — 112-Team Execution Results.md"
---

# Task: Execute 112-Team Stale Backfill and File Results

## Context

The 2026-04-22 stale team investigation identified 127 teams with unexplained staleness. Of those, 15 were confirmed genuine no-data teams (teams that exist in the DB but played zero games in the active window). That leaves **112 teams** that should have data and don't — they are stale due to pipeline fetch failures, not genuine inactivity.

The backfill for these 112 teams has been deferred since April 22 pending the pipeline structural fixes. Those fixes are now in place:
- Exponential backoff with jitter (`crawl-gotsport-api-parallel.js`)
- Dead-letter queue for failed fetches
- Empty-payload detection
- Per-run summary logging

The infrastructure to safely run this backfill now exists. It has not been run.

## What to Produce

### Step 1: Confirm the 112-Team ID List

Pull the 112 team_ids from the 2026-04-22 stale team investigation. Source: `Infrastructure/Unexplained Stale Teams Investigation.md` (or equivalent). The IDs should already be isolated in the investigation output as "genuine pipeline failures" (127 total minus 15 no-data confirmed = 112).

If the investigation file does not have a clean 112-team list, reconstruct it from the raw list of 127 minus the 15 confirmed no-data teams. File the 112-team ID list as a named constant in the execution script or as a flat text file so SENTINEL can audit the exact scope.

### Step 2: Run the Targeted Backfill

Execute `crawl-gotsport-api-parallel.js` with `GSAPI_TEAM_IDS` set to the 112-team list.

Run parameters:
- Use the current production configuration (backoff enabled, DLQ active)
- Target: GotSport API, same endpoint as the regular crawl
- Scope: team_ids only — no org-level sweep
- Log output: full per-run summary logging to a named log file

Do NOT run this against all active teams. This is a targeted remediation, not a full re-crawl.

### Step 3: File the Results

File: `02 - Tiger Tournaments/Projects/Infrastructure/Stale Team Backfill — 112-Team Execution Results.md`

Structure:

```
## Run Date
[date and time]

## Scope
- Team IDs targeted: 112
- Source of list: [file reference]

## Run Summary
- Teams with new data fetched: [N]
- Teams still returning empty payload after backoff: [N]
- Teams in DLQ (exhausted retries): [N]
- Total new games ingested: [N]

## DLQ Entries
[List of team_ids that hit DLQ — these are candidates for manual investigation or permanent deferral]

## Empty-Payload Teams (Persistent)
[Team_ids where empty-payload detection fired on all attempts — likely genuine no-data or GotSport-side issue]

## Post-Run Observation
[Did any team_ids that were in the 15 "genuine no-data" group show up in the DLQ? If so, flag — may indicate the 15-team classification was incorrect for some entries]

## SENTINEL Disposition
[ ] Results reviewed
[ ] DLQ entries: reviewed / deferred / escalated
[ ] Backfill complete: YES / PARTIAL (N of 112 recovered) / FAILED
Notes: ___
```

---

## Quality Standard

- Run the backfill against exactly 112 team_ids — not all active teams, not the full 127, not an approximation.
- The DLQ list must be explicit: every team_id that exhausted retries should be named.
- If the run produces errors that suggest the 112-team list was constructed incorrectly (e.g., invalid team_ids), stop and flag before completing the run.
- The SENTINEL Disposition section must be blank at filing.

## Why This Matters

These 112 teams have been stale since before April 22. Their ratings are not updating, which means for every game they've played since the stall, their Glicko-2 RD is inflating and their rating is diverging from truth. Any team that plays regularly is accumulating RD inflation that compounds over time.

May 1 the daily pipeline goes live. If the 112-team backfill has not run, the daily pipeline begins operations with 112 teams that are already behind — and will not catch up via the daily crawl at the normal rate because they're stale by weeks, not hours. Running the targeted backfill before May 1 gives the daily pipeline a clean starting state.

## References

- `Infrastructure/Unexplained Stale Teams Investigation.md` — source of the 127-team list
- `Infrastructure/GotSport API Scraper.md` — confirms backoff and DLQ are implemented
- `task-2026-04-22-unexplained-stale-teams.md` — original investigation task
