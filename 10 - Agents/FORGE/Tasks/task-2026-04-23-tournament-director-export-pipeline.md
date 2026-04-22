---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-23
status: completed
priority: medium
---

# Task: Design and Scope the Tournament Director Export Pipeline

## Context

Your 2026-04-22 webhook fallback scoping document (`Pipeline/Webhook Fallback Scope.md`) concluded that the v2 spec webhook is unbuildable today (GotSport does not offer outbound webhooks) and identified the **tournament director export pipeline** as the highest-value resilience investment. You estimated 7 days of development work. This task is to complete a full design spec and implementation plan for that pipeline before it goes on the sprint.

SENTINEL is treating this as the primary GotSport dependency mitigation. It must be designed well enough that Presten can approve or adjust it without ambiguity.

## What to Design

### Overview

The tournament director export pipeline allows tournament directors to manually export their GotSport event files (CSV or native GotSport format) and upload them to Evo Draw. Evo Draw parses the export, maps it to the `games` schema, and ingests it as a new source (priority 3 — below API and HTML but above TGS).

### Component 1: GotSport Export Format Research

Before writing any parsing code, document exactly what GotSport's tournament director export contains.

Answer these questions:
- What fields does the GotSport event CSV export include? (Expected: event ID, match ID, home team name, away team name, home score, away score, match date, division, age group)
- Does the export include GotSport internal team IDs or only team name strings?
- Is there a structured format (JSON, XML) available in addition to CSV?
- How does the export handle penalty shootouts, walkovers, and forfeits?
- What is the typical file size for a 100-team, 300-game event?

If you can access a sample GotSport export (check if any exist in the codebase or vault), document its exact schema. If not, document what you can infer from GotSport's platform behavior and the existing scraper's knowledge of GotSport's data model.

Write your findings to: `Reports/gotsport-export-format-2026-04-23.md`

### Component 2: Ingestion Pipeline Design

Write a full design spec for the ingestion pipeline to: `Pipeline/Tournament Director Export Pipeline.md`

The spec must include:

#### Upload Endpoint
- Route: `POST /api/ingest/event-export`
- Auth: Required (tournament director must have an Evo Draw account — this creates the relationship)
- Accepts: multipart/form-data with CSV file + metadata (tournament name, event ID if known, date range)
- Validation: file size limit, CSV format check, required field presence
- Response: job ID for async status polling

#### Parser: `scripts/parse-event-export.js`
- Input: uploaded CSV file path + metadata
- Output: array of game objects in the Evo Draw `games` schema
- Field mapping from GotSport export fields → `games` table columns
- Handling for:
  - Team name → `team_id` resolution: first check `teams` table by name, then `team_merges` by alias; if no match, create a provisional team record
  - Age group parsing: reuse existing `backfill-age-gender.js` parsing logic
  - Score parsing: handle penalty shootouts and walkovers using the same logic as `crawl-gotsport-v2.js`
  - Source ID: `gs_export_{uploadId}_{matchId}` — makes these records distinguishable and dedupable

#### Dedup Integration
- Source priority: 3 (below API priority 1 and HTML priority 2)
- If a GotSport export game has the same event ID + match ID as an API-scraped game: the API version wins, the export version is marked `is_hidden = true` (dedup'd)
- If no matching API or HTML record exists: the export version is the canonical record
- This logic prevents double-counting when tournament directors contribute data for events the scraper already covered

#### Incentive / Confirmation Screen Design (Spec Only — Not Implementation)
After a successful upload, the tournament director should see:
- Count of games ingested
- Count of teams whose rankings were updated
- A link to the Evo Draw rankings page filtered to their event
- A prompt: "Want to receive division placement recommendations for your next event? [Contact us]"

This is the strategic relationship moment. Spec what this screen should show and what data it requires — engineering will implement it, but the design logic should be in the spec.

### Component 3: Implementation Estimate Validation

Your 2026-04-22 estimate was 7 days of dev work. Break this down into specific tasks with day-level estimates:

| Task | Estimate | Notes |
|------|----------|-------|
| Upload endpoint | 1 day | |
| GotSport export format parser | ? days | |
| Tournament director UI | ? days | |
| Dedup integration | ? days | |
| Testing with real export data | ? days | |
| **Total** | **?** | |

If your research in Component 1 reveals that the GotSport export format is more complex than expected (e.g., no team IDs, only names), adjust the estimate accordingly and flag it.

## What to Read

- `/Users/p/Documents/Obsidian Vault/02 - Tiger Tournaments/Projects/Pipeline/Webhook Fallback Scope.md` — your own scoping document; Part 4 is the starting point
- `/Users/p/Documents/Obsidian Vault/02 - Tiger Tournaments/Projects/Strategic Insights.md` — Risk 1 and Action Item 10 for the GotSport partnership angle; Insight 11 for the tournament director relationship value
- `[[GotSport HTML Scraper]]` vault note — the HTML scraper already parses GotSport event data from a different angle; the export parser can share logic
- `[[Data Sources Overview]]` — source priority table (confirm priority 3 slot is correct)
- `[[Dedup Strategy]]` — existing dedup key format to ensure the export source ID is compatible

## What to Produce

1. `Reports/gotsport-export-format-2026-04-23.md` — GotSport export format documentation
2. `Pipeline/Tournament Director Export Pipeline.md` — full design spec (all three components above)
3. An updated day-level implementation estimate at the end of the spec

## Definition of Done

- The design spec is detailed enough that an engineer could implement it without asking clarifying questions about the data model or dedup logic
- The GotSport export format is documented with enough specificity that the parser can be written from it
- The implementation estimate is broken into tasks with day-level estimates (not a single 7-day lump)
- All three scenarios from the Webhook Fallback Scope (A, B, C) are addressed in the spec: specifically, which scenarios this pipeline mitigates and which it does not

## Note on Priority

This is MEDIUM priority (not HIGH) because Scenario A (current rate-limited API) is manageable with the backoff fixes in `task-2026-04-23-pipeline-structural-fixes.md`. But the design work should be done this week so the export pipeline can go on the next sprint. The structural fixes are urgent; this is important.
