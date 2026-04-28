---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-05-09
priority: medium
status: completed
completed: 2026-04-27
topic: snapsoccer-minimum-viable-parser-spec
---

# Task: SnapSoccer — Minimum Viable Parser Spec

## Context

FORGE has completed the SnapSoccer Go/No-Go Decision Package (ranked #1 non-GotSport source, authorized post-DSS pending browser accessibility confirmation), the Test Extraction Plan (how to validate the approach before a full build), and the Browser Check Protocol (three URLs, binary ACCESSIBLE/BLOCKED result).

What does not exist is a parser specification: which pages to scrape, which fields to extract, what the output schema looks like. Without a spec, the May 17 build session starts with discovery — identifying what data is available, how it is structured, and what format to output. That discovery work can be done now, before the browser session even runs, using the vault's existing SnapSoccer research.

This task produces that spec. It does not require browser access. It is a design document that FORGE will validate and refine during the actual build session, but its core decisions should be locked before May 17.

## Task

Produce a Minimum Viable Parser Spec for the SnapSoccer scraper. "Minimum viable" means: the smallest scope that produces usable game data for Evo Draw. The spec should define exactly what the May 17 session implements — no more, no less.

## Document Must Include

### Section 1: Target Pages

List the SnapSoccer page types that contain game results data. Based on existing vault research, identify:
- The base URL pattern (`soccer.sincsports.com/...` — infer from Go/No-Go document)
- Which page types contain scores/results (tournament bracket? league standings? game-by-game log?)
- Entry points: how the scraper navigates from a known event or org to game result pages

Note any pages that require authentication or login. These are out of scope for the MVP.

### Section 2: Field Extraction Schema

For each field Evo Draw requires, document:
- Field name in Evo Draw schema
- Source HTML element / CSS selector pattern (e.g., `table.game-results td.team-name`)
- Data type and format (string, date YYYY-MM-DD, integer score)
- Required vs. optional
- Known ambiguities or parsing risks (e.g., team name variations, forfeit notation)

Minimum required fields:
- home_team_name (string)
- away_team_name (string)
- home_score (integer)
- away_score (integer)
- game_date (date)
- event_name (string)
- league / tier classification (string, may be inferred from page context)

Optional fields to extract if available without additional requests:
- age_group
- gender
- location / venue

### Section 3: Parser Architecture

One paragraph: `requests` + `BeautifulSoup` approach, no JavaScript execution required (confirm from vault research). Note rate-limiting behavior if known. Confirm no session auth is required for public tournament pages.

If any page type requires JavaScript execution (dynamic loading), flag it explicitly and mark those pages as out-of-scope for the MVP.

### Section 4: Output Format

Define the output format FORGE will use:
- File format (CSV, JSON, or direct DB insert)
- One example row of output in the target format
- Any field normalization to apply before output (e.g., team name → lowercase, date → ISO 8601)

### Section 5: MVP Scope Boundaries

Explicit list of what is OUT of scope for the May 17 session:
- Auth-required pages
- Historical data before [date] (define if applicable)
- Non-public events
- Any page type not confirmed accessible during the browser check

Anything out of scope should be noted as a potential Phase 2 addition, not discarded.

## Deliverable

File to: `Data Sources/SnapSoccer-Minimum-Viable-Parser-Spec-2026-04-27.md`

Update this task to `status: completed` when filed.

## Notes

This is a design document, not an implementation. FORGE should not write any Python code in this task. The output is a spec that a developer (or FORGE in a future session) can implement directly.

If vault research on SnapSoccer is insufficient to fill Section 1 or Section 3 fully, note the gap explicitly and flag it as a pre-build question for the browser check session. Do not fabricate URL patterns — mark them as "to confirm during browser session" if unknown.

The quality bar for this document: a developer who has never seen the SnapSoccer site should be able to read it, understand what the scraper does, and implement it without asking clarifying questions. Everything uncertain should be a labeled "TBC during browser session" note, not a gap.

## References

- `Data Sources/SnapSoccer-Go-No-Go-Decision-Package-2026-04-24.md`
- `Data Sources/SincSports-Browser-Check-Protocol-2026-04-27.md`
- `Infrastructure/April 29 — FORGE Execution Checklist.md`
