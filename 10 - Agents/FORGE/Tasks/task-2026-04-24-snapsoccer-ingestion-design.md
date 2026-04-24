---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: medium
due: 2026-05-02
---

# Task: SnapSoccer Ingestion Pipeline — MVP Design

## Context

SnapSoccer is a recurring coverage gap in the Evo Draw record. SENTINEL has flagged it in the 21:36 and 23:52 briefings. Your SnapSoccer Source Research (`02 - Tiger Tournaments/Projects/Data Sources/SnapSoccer Source Research.md`) documented the source — what it is, what data it exposes, and what the estimated coverage uplift would be. That research is done.

The gap: there is no ingestion design. USA Rank uses SnapSoccer. We do not. DSS is 22 days out. Getting SnapSoccer games into the DB before May 16 is unlikely but not impossible if the design work is done now and Presten can run a targeted ingestion session in early May.

This task: design the MVP ingestion pipeline — just enough to get SnapSoccer game results into the `games` table using the simplest possible approach. Not a full production scraper. Not a multi-phase project. One working script that ingests results from SnapSoccer's most accessible public data path.

## What You Need to Do

### Step 1: Identify the Simplest Data Access Path

Read your SnapSoccer Source Research document. Identify the single most accessible path to game result data:

- **Option A: Public results pages** — if SnapSoccer has public HTML result pages by league or division, an HTML scraper is the lowest-friction path
- **Option B: Unofficial API** — if SnapSoccer exposes an API (authenticated or not) that returns structured game data
- **Option C: Export / download** — if SnapSoccer publishes downloadable standings or schedule CSVs

For each option you identified in the research, assess:
- Authentication required? (Yes/No)
- Rate limiting observed?
- Data format (HTML, JSON, CSV)?
- Coverage estimate (games per month)?

Pick one option as the MVP target. State your reasoning.

### Step 2: Map the Data to the Games Table Schema

SnapSoccer game data will have some combination of: home team name, away team name, home score, away score, date, event/league name.

Our `games` table schema (from `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md`) requires specific columns. Map every SnapSoccer field to its target column:

| SnapSoccer Field | `games` Column | Notes |
|---|---|---|
| home_team | home_team_name | May need normalization |
| ... | ... | ... |

Identify any columns in `games` that are required but SnapSoccer does not provide. For each gap, document the fallback value (e.g., `source = 'snapsoccer'`, `event_tier = 'snapsoccer'`).

**The team matching problem:** SnapSoccer team names will not match GotSport team names exactly. Define the matching strategy:
- First pass: fuzzy match on `teams.name` using `ILIKE '%keyword%'`
- Second pass: if no match, insert as a new unverified team with `source = 'snapsoccer'`
- Do NOT skip games where teams don't match — insert them with the raw name and let the merge pipeline handle it

Document this strategy explicitly. It is the biggest risk in SnapSoccer ingestion.

### Step 3: Write the MVP Script Spec

Write a spec for `scripts/ingest-snapsoccer.js` (or `.py` if Python is more appropriate given the codebase):

**Inputs:**
- A list of SnapSoccer league/division URLs (hardcoded config or CSV file)
- `--dry-run` flag: logs what would be inserted without writing to DB

**Processing:**
1. For each URL: fetch the page / API response
2. Parse game results into standardized format: `{home, away, home_score, away_score, date, event_name}`
3. For each game: lookup or insert team records in `teams`
4. Insert into `games` with `source = 'snapsoccer'`
5. Log: games fetched, games inserted, games skipped (no team match), errors

**Outputs:**
- Console log of run summary
- Optional: `Logs/snapsoccer-ingest-{date}.jsonl` for audit trail

Write the function signatures and data flow, not full implementation. Use the same pattern as `crawl-gotsport-api-parallel.js` for consistency.

### Step 4: Identify Pre-Conditions for a May 1 Test Run

For Presten to run a test ingestion before May 9 (last safe implementation date before DSS), what must be true?

List the pre-conditions:
- [ ] SnapSoccer source URL confirmed accessible (no auth wall)
- [ ] `games` table schema confirmed (columns match the mapping from Step 2)
- [ ] Target league list (which 2–3 leagues to ingest in the test)
- [ ] De-duplication logic confirmed (what prevents re-ingesting the same game twice)

The de-duplication strategy is critical — define it now. Likely: composite unique key on `(home_team_name, away_team_name, date, event_name)`. Or insert with `ON CONFLICT DO NOTHING` if a unique constraint exists. Document the approach you recommend.

### Step 5: Scope and Effort Estimate

Provide an honest effort estimate:
- Script writing: X hours (by Presten or FORGE spec + Presten execution)
- Test ingest run: Y hours (data volume dependent)
- Full production ingest: Z hours

Flag if SnapSoccer coverage, based on your research, is large enough to meaningfully move the DSS coverage claim. If coverage is limited (e.g., <5,000 games), it may not be worth the session time vs the Org-ID expansion (20,000+ games) which is already specced.

## Deliverable

Write the MVP design to:
`02 - Tiger Tournaments/Projects/Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md`

The document must:
- Name the MVP data access path and justify the choice
- Include the field mapping table
- Include the script spec (function signatures, data flow, logging)
- Include the pre-conditions checklist
- Include an effort estimate and a go/no-go recommendation based on coverage vs effort

## Definition of Done

- MVP design document written and filed
- Data access path is specific (not "could try Option A or B" — pick one)
- Field mapping is complete (no unmapped required columns)
- Team matching strategy is defined
- Go/no-go recommendation is stated with reasoning
- Summary in your briefing: access path chosen, estimated games/month, go or no-go for May 1 test

## Why This Matters

SnapSoccer is the one source family USA Rank uses that we don't. At DSS, if a club director asks "do you have our SnapSoccer league results?" the answer should not be "no." If the coverage is material (10,000+ games), this is worth a session before May 9. If it's not, SENTINEL needs to know that so we stop revisiting it.

## References

- `02 - Tiger Tournaments/Projects/Data Sources/SnapSoccer Source Research.md` — your prior research
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — games + teams schema
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — their SnapSoccer usage
- `crawl-gotsport-api-parallel.js` — script pattern to follow
