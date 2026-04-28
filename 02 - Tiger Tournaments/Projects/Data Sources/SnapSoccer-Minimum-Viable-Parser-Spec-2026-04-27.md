---
title: SnapSoccer — Minimum Viable Parser Spec
tags: [data-sources, snapsoccer, sincsports, parser, spec, post-dss, evo-draw, forge]
created: 2026-04-27
author: FORGE
status: design-complete — ready for May 17 build session
task: task-2026-04-27-snapsoccer-minimum-viable-parser-spec
---

# SnapSoccer — Minimum Viable Parser Spec

> **Purpose:** This is the implementation spec for the May 17 SnapSoccer scraper build session. It defines exactly what gets built — no more, no less. A developer who has never seen the SnapSoccer site should be able to implement this without asking clarifying questions. Every unknown is marked **TBC during browser session**, not left as a silent gap.

**Prerequisites before coding:**
- Browser check (`Data Sources/SincSports-Browser-Check-Protocol-2026-04-27.md`) must return **ACCESSIBLE**
- Existing SincSports parser: no reuse possible (verdict: REBUILD — see `Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md`)
- Build session authorization: SENTINEL post-DSS GO, contingent on browser accessibility

---

## Section 1: Target Pages

### Platform

SnapSoccer competitive events are managed on **SincSports** — specifically the `soccer.sincsports.com` subdomain. This is distinct from the main `sincsports.com` domain (which is Cloudflare-protected). The subdomain serves public, server-rendered HTML with no JavaScript execution required.

### Page Types

| Page Type | URL Pattern | Data Available | In MVP Scope |
|-----------|-------------|----------------|--------------|
| Tournament schedule/scores | `soccer.sincsports.com/schedule.aspx?tid={TID}` | All games, scores, dates, divisions for a tournament | **YES — primary target** |
| Event metadata | `soccer.sincsports.com/TTMore.aspx?tid={TID}` | Event name, age groups, dates | Optional (fall back to page `<title>` for event name) |
| Club teams | `soccer.sincsports.com/TTClub.aspx?tid={TID}` | Team names, club affiliations | No — not needed for MVP |
| SnapSoccer event listing | `snapsoccer.com/events/` | Links to SincSports events (TID discovery) | **YES — entry point for TID discovery** |

**Authentication required:** None. All schedule and score pages are fully public. No login wall, no session token required for public tournament data.

### Entry Points

The scraper navigates as follows:

1. **TID Discovery:** Fetch `https://snapsoccer.com/events/`. Extract all `href` attributes containing `soccer.sincsports.com` or the pattern `tid=`. Parse the `tid=` query parameter from each link. This produces the list of tournament IDs (TIDs) to scrape.

2. **Filter Playdate TIDs:** Remove known no-score tournament IDs before scraping:
   - `SNAPLEAG` (Spring Playdates — no scores by design)
   - `SNAPLEAF` (Fall Playdates — no scores by design)
   - `SSSPF` (no scores by design)
   Scraping these will return schedule data but no scores — they are explicitly development-format events.

3. **Schedule Fetch:** For each remaining TID, fetch `https://soccer.sincsports.com/schedule.aspx?tid={TID}`.

4. **Parse → Insert:** Parse the schedule page HTML for all game rows. Insert into `games` table via ON CONFLICT upsert.

### Pages Requiring Authentication

None identified. If a page redirects to a login wall during the browser check, mark it out of scope and do not add its TID to the scrape list.

---

## Section 2: Field Extraction Schema

### HTML Structure (Based on Research — Confirm During Browser Session)

SincSports `schedule.aspx` uses **ASP.NET WebForms server-rendered HTML**. The expected structure is a `<table>` per division, with division name in a section-header row and one `<tr>` per game. All data is present in the initial HTTP response — no JavaScript execution or AJAX calls required.

**TBC during browser session:** The exact CSS class names for section headers (`<tr class="?">`) and game rows (`<tr class="?">`) must be confirmed by inspecting the actual HTML. The structure below is based on ASP.NET WebForms conventions and existing vault research. If the actual HTML differs, adjust selectors before coding.

### Required Fields

| Evo Draw Field | HTML Source | CSS/XPath Hint | Data Type | Required | Parsing Notes |
|----------------|-------------|----------------|-----------|----------|---------------|
| `home_team_name` | Schedule table row, team name cell (position 1 or 2) | `<td>` in game `<tr>` — **TBC: column index during browser session** | string | **Yes** | Extract raw text; pass through team name normalization (lowercase, strip extra whitespace) |
| `away_team_name` | Schedule table row, team name cell (position 2 or 3) | `<td>` in game `<tr>` — **TBC: column index during browser session** | string | **Yes** | Same normalization as home team |
| `home_score` | Score cell — left side of separator | Score cell contains format like `"2 - 1"` or `"2:1"` — **TBC: separator character during browser session** | integer | **Yes** | Parse left of separator; strip whitespace; cast to int; if non-numeric (e.g., "W", "F/F"), treat as forfeit — skip insertion (game not scoreable) |
| `away_score` | Score cell — right side of separator | Same cell as home_score | integer | **Yes** | Parse right of separator; same rules |
| `game_date` | Date column | `<td>` in game `<tr>` — **TBC: column index during browser session**; expected format `MM/DD/YYYY` | date | **Yes** | Parse using Python `datetime.strptime(val, "%m/%d/%Y")`, output as ISO 8601 (`YYYY-MM-DD`) |
| `event_name` | Page `<title>` tag or `<h1>` | `soup.title.string` or `soup.find('h1').text` | string | **Yes** | Strip " - SincSports" suffix if present in `<title>` |
| `division` | Section header row above game table block | `<tr>` with class indicating section header — **TBC: class name during browser session** | string | **Yes** | Raw division string (e.g., "U13 Boys Division 1"); used as input to age/gender parsers |

### Optional Fields (Extract If Available Without Extra Requests)

| Evo Draw Field | HTML Source | Notes |
|----------------|-------------|-------|
| `age_group` | Derived from `division` | Run through existing age-group parser (backfill logic); extract "U13", "U14", etc. |
| `gender` | Derived from `division` | "Boys" → `'M'`; "Girls" → `'F'`; "Coed" → `null` |
| `round` | Table column if present (e.g., "Group Stage", "Semifinal", "Final") | **TBC: whether round column exists during browser session**; if absent, set to `null` |
| `venue` / `location` | Table column if present | Not a required `games` column; skip if not present without additional requests |

### Hardcoded Fields (Not in Source HTML)

| Field | Value | Rationale |
|-------|-------|-----------|
| `source` | `'snapsoccer'` | Attribution clarity; separate from `'sinc'` for reporting |
| `source_priority` | `5` | Same tier as existing Sinc Sports data |
| `game_status` | `'completed'` | Only insert games with numeric scores; skip scheduled games |
| `event_tier` | `null` | SnapSoccer events are regional invitationals; leave null for tier classifier to set post-ingestion |
| `is_hidden` | `false` | Default for new records |
| `duplicate_of` | `null` | Handled by dedup pipeline post-ingestion |
| `season` | Derived from `game_date` | If `game_date` month ≥ 8: season = `"{year}-{year+1}"`; else: season = `"{year-1}-{year}"` |

### Source ID Format

```
snapsoccer_{TID}_{gameId}
```

**Preferred:** If SincSports exposes a stable game ID in the HTML (e.g., `GameId=12345` in a link or attribute on the game row), use `snapsoccer_{TID}_{gameId}`.

**Fallback (if no stable game ID):** Deterministic composite:
```
snapsoccer_{TID}_{YYYY-MM-DD}_{homeSlug}_{awaySlug}
```
Where `homeSlug` and `awaySlug` are lowercased, whitespace-stripped, alphanumeric-only fragments of the team name (first 10 characters). This ensures idempotent re-runs even without an explicit game ID in the source HTML.

**TBC during browser session:** Inspect one game row's HTML to confirm whether a stable game ID attribute is present.

---

## Section 3: Parser Architecture

**Approach:** `requests` + `BeautifulSoup` (Python). No JavaScript execution required — all score and schedule data is present in the initial server-rendered HTML response from `soccer.sincsports.com/schedule.aspx`. This is a classic ASP.NET WebForms site; scores are not loaded asynchronously.

**Rate limiting:** SincSports does not publicly document rate limits. Safe approach: **800–1000ms sleep between requests**. At 15–30 tournament pages total, a full SnapSoccer crawl completes in under 1 minute at this rate. Rate limiting is a non-issue at this volume.

**Session auth:** Not required. Standard HTTP GET requests with a generic browser User-Agent header are sufficient. No cookies, session tokens, or OAuth required.

**Cloudflare / bot protection:** The `soccer.sincsports.com` subdomain is distinct from `sincsports.com` (which is Cloudflare-protected). The browser check protocol must confirm the subdomain is publicly accessible before build begins. If the browser check returns BLOCKED, the entire MVP is blocked and this spec is suspended.

**If any page type requires JavaScript execution (dynamic loading):** Flag it explicitly and mark it out of scope for the MVP. As of this spec, no page type in the SnapSoccer/SincSports research requires JS execution.

---

## Section 4: Output Format

### Primary Output: Direct DB Insert

The MVP inserts directly into the `games` table via `psycopg2` (Python) using an ON CONFLICT upsert:

```sql
INSERT INTO games (
    source, source_id, source_priority,
    event_name, home_team_name, away_team_name,
    home_score, away_score, game_date,
    division, age_group, gender, round,
    game_status, season, event_tier, is_hidden
)
VALUES (
    'snapsoccer', %(source_id)s, 5,
    %(event_name)s, %(home_team_name)s, %(away_team_name)s,
    %(home_score)s, %(away_score)s, %(game_date)s,
    %(division)s, %(age_group)s, %(gender)s, %(round)s,
    'completed', %(season)s, NULL, false
)
ON CONFLICT (source, source_id)
DO UPDATE SET
    home_score = EXCLUDED.home_score,
    away_score = EXCLUDED.away_score,
    updated_at = NOW();
```

### Secondary Output: Audit Log (Optional, `--log` flag)

`Logs/snapsoccer-ingest-YYYY-MM-DD.jsonl` — one JSON object per game:

```json
{
  "source_id": "snapsoccer_STXSCC25_2025-10-12_ne-surf_ocean-city",
  "tid": "STXSCC25",
  "result": "inserted",
  "home_team": "NE Surf",
  "away_team": "Ocean City Nor'easters",
  "score": "2-1",
  "game_date": "2025-10-12",
  "division": "U13 Boys Division 1",
  "age_group": "U13",
  "gender": "M"
}
```

### Example Row (Normalized)

| field | value |
|-------|-------|
| source | snapsoccer |
| source_id | snapsoccer_STXSCC25_2025-10-12_ne-surf_ocean-city |
| source_priority | 5 |
| event_name | Coastal Soccer Invitational |
| home_team_name | ne surf |
| away_team_name | ocean city nor'easters |
| home_score | 2 |
| away_score | 1 |
| game_date | 2025-10-12 |
| division | U13 Boys Division 1 |
| age_group | U13 |
| gender | M |
| round | Group Stage |
| game_status | completed |
| season | 2025-2026 |

### Field Normalization Applied Before Insert

- `home_team_name`, `away_team_name`: lowercase, strip leading/trailing whitespace
- `game_date`: parse from `MM/DD/YYYY`, output as `YYYY-MM-DD`
- `gender`: `"Boys"` → `'M'`; `"Girls"` → `'F'`; anything else → `null`
- `home_score`, `away_score`: cast to integer; non-numeric → skip game (not inserted)
- `season`: derive from `game_date` year and month as described in Section 2

---

## Section 5: MVP Scope Boundaries

### Out of Scope for May 17 Session

The following are explicitly excluded from the MVP build. Each is noted as a Phase 2 candidate.

| Out-of-Scope Item | Reason | Phase 2 Path |
|-------------------|--------|--------------|
| Authentication-required pages | No auth pages identified in SnapSoccer research; if any are discovered during build, skip them | Post-MVP: assess whether auth is obtainable |
| Historical data before 2023-01-01 | Out-of-scope for MVP test run; historical TIDs from 2018–2022 may exist but discovery is not part of the May 17 session | Phase 2: add historical TID discovery pass for pre-2023 events |
| SnapSoccer Playdate series (SNAPLEAG, SNAPLEAF, SSSPF) | Explicitly no scores by design — no game results to ingest | Not a Phase 2 candidate; no scores exist |
| Non-public events | Any event page that requires login or returns 403 | Phase 2 only if auth access is negotiated |
| `soccer.sincsports.com` events not linked from `snapsoccer.com/events/` | Scope is SnapSoccer-branded events only; not a general SincSports crawl | Phase 2: separate SincSports expansion if warranted |
| Any page type not confirmed accessible during browser check | If the browser check returns BLOCKED on schedule pages, those TIDs are excluded | Revisit if access changes |
| Real-time / live score updates | Evo Draw ingests completed games only | Not a planned future scope |
| Team ID resolution (home_team_id / away_team_id) | MVP uses raw team names; existing team merge pipeline handles ID linkage post-ingestion | Phase 2: tune team matching pass; ID linkage improves over time |

### What "Phase 2" Means

Phase 2 = a separate build session after the MVP is confirmed working in production. Phase 2 additions are not part of the May 17 authorization. SENTINEL must authorize each Phase 2 addition separately.

---

## Pre-Build Gate (Must Be True Before May 17 Coding Begins)

- [ ] Browser check (`Data Sources/SincSports-Browser-Check-Protocol-2026-04-27.md`) result: **ACCESSIBLE**
- [ ] Presten has confirmed: one `schedule.aspx?tid=XXXX` URL loads a visible HTML schedule table with game rows (Step 2 of browser check)
- [ ] SENTINEL has authorized post-DSS SnapSoccer MVP build

If the browser check returns BLOCKED: this spec is suspended. Do not begin coding.

---

## References

- `Data Sources/SnapSoccer Source Research.md` — platform research, URL structure, game volume estimates, skeleton scraper
- `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md` — full implementation design with script spec and team matching strategy
- `Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md` — REBUILD verdict; no existing parser to reuse
- `Data Sources/SincSports-Browser-Check-Protocol-2026-04-27.md` — accessibility gate that must pass before build begins
- `Data Sources/Sinc Sports.md` — existing SincSports data (34,961 games via bulk import; source priority 5)

*FORGE — 2026-04-27*
