---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: high
due: 2026-04-25
---

# Task: Expand GotSport Team Discovery to USYS State Association Org IDs

## Context

Your USYS State Cup research (delivered 2026-04-23) found that ~80–85% of USYS state associations publish results via GotSport, but that state cup and state league teams in **unranked divisions** never appear in the GotSport rankings endpoint that drives our team ID discovery feed. These teams have games in GotSport but are invisible to our crawler because we only discover team IDs from the public rankings feed.

The fix is not a new scraper — it is a configuration change. USYS state association org IDs exist in GotSport. If we explicitly seed our team discovery with those org IDs (bypassing the rankings endpoint for those orgs), we capture an estimated **20,000–35,000 games currently missing from the database**.

This is the highest-impact pre-DSS pipeline change on the board. Estimated 1–2 days. DSS is May 16.

## What You Need to Do

### Step 1: Identify the USYS State Association GotSport Org IDs

For each of the top 10 USYS state associations (TX, CA, FL, NY, NJ, PA, IL — note: IL uses Affinity Soccer, skip; also WA, OH, CO), find their GotSport org ID.

Method:
- Check the USYS State Cup Source Research doc (`02 - Tiger Tournaments/Projects/Data Sources/USYS State Cup Source Research.md`) — you already documented org ID patterns for TX, CA, FL, NY, NJ, PA there. Pull those org IDs.
- For any states not covered: query the `games` table — state cup events from those states are likely already in the DB from prior GotSport crawls. Look for event names matching `%state cup%` or `%state league%` and pull the event's associated org ID from the event metadata.
- Target: a list of at minimum 5 confirmed org IDs covering the largest state associations.

Document the org IDs in a table:
| State Association | GotSport Org ID | Event Count (from DB) | Notes |
|---|---|---|---|

### Step 2: Understand Current Team Discovery Configuration

Read the GotSport API scraper to find where team IDs are seeded into the discovery queue:
- `crawl-gotsport-api-parallel.js` — where does it pull the initial set of team IDs? Is it from a config file (`GSAPI_TEAM_IDS`), a DB query, or a live rankings endpoint call?
- Check `run-overnight.sh` and `run-daily.sh` (once built) — how is the team ID list generated?
- If there is a `config/gotsport-orgs.json` or similar org ID list, document its current contents.

The goal: understand exactly which file or query to modify to add org-level team discovery.

### Step 3: Implement Org-Level Team Discovery

Modify the pipeline's team ID seed process to add a step that enumerates team IDs from specific org IDs.

GotSport's API pattern for org-level team enumeration (if not already implemented):
- `GET /api/v1/organization/{org_id}/teams` — fetches all teams registered under an org
- If this endpoint is not already in the codebase, check the GotSport API Scraper notes for alternative methods (HTML scraper org team pages, or the event-level team listing endpoint)

Implementation:
1. Add the identified USYS state association org IDs to the org list in the appropriate config file or DB table
2. Ensure the org-level team enumeration runs before the ranked team ID fetch (so these org-sourced IDs are included in every sync run, not just full sweeps)
3. If org-level discovery doesn't exist as a code path: write `scripts/discover-teams-by-org.js` — a simple script that accepts a list of org IDs and outputs team IDs to stdout (to be piped into the main sync runner)

### Step 4: Test Run on One State

Before adding all org IDs, test the configuration change on one state association (recommend TX — largest, most established).

- Run the org-level team discovery against the TX state association org ID
- Count new team IDs discovered (not already in `teams` table)
- Spot check 5 of the new teams — confirm they are real USYS state league/cup teams and not duplicates
- Estimate extrapolated new team count across all added org IDs

Document results in: `Reports/gotsport-org-expansion-test-2026-04-24.md`

### Step 5: Production Config Update

Once the test passes:
- Add all confirmed org IDs to the production config
- Run a targeted crawl of the new team IDs (do NOT re-crawl all 107,528 IDs — crawl only the newly discovered IDs)
- Report game count added to the database

## Deliverable

1. `Reports/gotsport-org-expansion-test-2026-04-24.md` — test run results for the TX org, with:
   - List of org IDs added to config
   - New team count discovered vs already-known
   - Estimated total game volume increase across all orgs
   - Any errors or rate-limit behavior from org-level discovery
2. Updated config file or DB record showing the new org IDs (note the file path in the report)
3. Summary in your 2026-04-24 briefing: confirmed new team and game counts ingested

## Success Criteria

- At least 5 USYS state association org IDs confirmed and added to the discovery config
- A test crawl runs successfully against at least 1 org ID
- The number of new team IDs discovered is documented
- No disruption to the existing team discovery process (ranked endpoint still runs as before)

## Why Before DSS

Source breadth is a key demo differentiator vs USA Rank. Going from 7 source families to "8 + USYS state infrastructure" is a story. Adding 20,000+ games before May 16 means more teams ranked, better coverage for the demo region (NJ/NE teams are the exact demographic in the USA Rank comparison). This is the easiest per-game-added win on the board.

## References

- `02 - Tiger Tournaments/Projects/Data Sources/USYS State Cup Source Research.md` — org ID notes and coverage gap estimate (your document)
- [[GotSport API Scraper]] — team discovery step
- [[Data Pipeline]] — Step 0 (team ID seeding)
- `crawl-gotsport-api-parallel.js` — modify or extend the discovery input
