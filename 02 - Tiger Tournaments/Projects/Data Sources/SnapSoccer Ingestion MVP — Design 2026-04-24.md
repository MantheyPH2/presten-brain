---
title: SnapSoccer Ingestion MVP — Design 2026-04-24
tags: [snapsoccer, ingestion, sincsports, data-sources, mvp, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: FORGE
status: design-complete
task: task-2026-04-24-snapsoccer-ingestion-design
recommendation: conditional-go
---

# SnapSoccer Ingestion MVP — Design

> [!note] Prior Research
> This document builds on `Data Sources/SnapSoccer Source Research.md` (FORGE, 2026-04-23). Read that first for platform background, game volume estimates, and HTML structure details.

---

## 1. MVP Data Access Path

**Selected: Option A — HTML scrape of `soccer.sincsports.com/schedule.aspx`**

**Reasoning:**

| Criteria | Assessment |
|---|---|
| Authentication required | None — fully public pages |
| Rate limiting | None documented; at 15–30 event pages, entire crawl completes in < 1 minute |
| Data format | Server-rendered HTML (ASP.NET WebForms); all scores in initial HTTP response — no JS execution required |
| Coverage estimate | ~1,500–2,500 games/year; ~10,000–20,000 historical games |
| Existing codebase overlap | **We already have 34,961 SincSports games ingested.** SnapSoccer events are SincSports events — same platform, more tournament IDs. The existing Sinc parsing logic is the base. |

Options B (unofficial API) and C (CSV export) were not identified in the research — neither SincSports nor SnapSoccer exposes a public JSON API or downloadable results CSV. Option A is the only viable path.

**Critical constraint:** SnapSoccer **Playdate series** (TIDs: `SNAPLEAG`, `SNAPLEAF`, `SSSPF`) explicitly have no scores — no results to ingest. Target competitive tournament events only.

---

## 2. Field Mapping to `games` Table

| SnapSoccer Field | Source in HTML | `games` Column | Notes |
|---|---|---|---|
| `event_name` | Page `<title>` or `<h1>` | `event_name` | e.g., "Coastal Soccer Invitational" |
| `home_team_name` | Schedule table row, col 1 | `home_team_name` | Raw name; normalization pass recommended |
| `away_team_name` | Schedule table row, col 2–3 | `away_team_name` | Raw name |
| `home_score` | Score cell (left of dash/hyphen) | `home_score` | Parse "2-1" → 2 |
| `away_score` | Score cell (right of dash) | `away_score` | Parse "2-1" → 1 |
| `match_date` | Date column (MM/DD/YYYY) | `match_date` | Convert to ISO date |
| `division` | Section header above game table | `division` | e.g., "U13 Boys Division 1" |
| `age_group` | Parsed from `division` | `age_group` | Run through existing age group backfill parser |
| `gender` | Parsed from `division` | `gender` | "Boys" → "M", "Girls" → "F" |
| `round` | Column if present | `round` | "Group Stage", "Semifinal", "Final" |
| *(not in source)* | — | `source` | Hardcode: `'snapsoccer'` |
| *(not in source)* | — | `source_priority` | Hardcode: `5` (same as Sinc Sports) |
| *(not in source)* | — | `game_status` | Hardcode: `'completed'` (only ingest completed games) |
| *(not in source)* | — | `season` | Derive from `match_date` — "2025-2026" format |
| *(derived)* | `tid` from URL + row ID | `source_id` | Format: `snapsoccer_{tid}_{gameId}` |

**Columns required by schema but not in SnapSoccer data:**

| Column | Fallback Value | Rationale |
|---|---|---|
| `source` | `'snapsoccer'` | Attribution clarity |
| `source_priority` | `5` | Same tier as Sinc Sports — no dedup precedence change needed |
| `event_tier` | `'regional'` or `null` | SnapSoccer events are regional invitationals; leave null until tier classification runs |
| `home_team_id` / `away_team_id` | Result of team match lookup or null | Set to matched `teams.team_id` if found; null if unmatched — see Section 4 |
| `is_hidden` | `false` | New records default to visible |
| `duplicate_of` | `null` | Handled by dedup pipeline post-ingestion |

---

## 3. Team Matching Strategy

SnapSoccer team names will not match GotSport names exactly. The matching strategy:

**Pass 1: Fuzzy match**
```sql
SELECT id, name FROM teams
WHERE name ILIKE '%' || $1 || '%'
LIMIT 5;
```
Use the longest unique keyword from the team name (e.g., "NE Surf" from "NE Surf G2012 Navy"). If exactly one match returns → use it.

**Pass 2: No match → insert new team record**
```javascript
const teamId = await db.query(`
  INSERT INTO teams (name, source, created_at)
  VALUES ($1, 'snapsoccer', NOW())
  ON CONFLICT DO NOTHING
  RETURNING id
`, [rawTeamName]);
```
Do NOT skip games where teams don't match. Insert with the raw name. The merge pipeline handles dedup after ingestion. An unmatched team still contributes a scorable game record — game completeness matters more than team ID linkage at ingest time.

**Pass 3: Log all unmatched team names**
Write unmatched names to `Logs/snapsoccer-unmatched-teams-{date}.jsonl` for manual review or future fuzzy match tuning.

---

## 4. Script Spec — `scripts/ingest-snapsoccer.js`

Pattern: follow `crawl-gotsport-api-parallel.js` for structure (connection pool, per-item processing loop, logging).

### Inputs

```
node scripts/ingest-snapsoccer.js [options]

Options:
  --dry-run          Log insertions without writing to DB
  --tids <file>      Path to text file with one TID per line (default: config/snapsoccer-tids.txt)
  --since <date>     Only ingest events with games on or after this date (YYYY-MM-DD)
  --limit <n>        Max tournament IDs to process (for test runs)
```

### Configuration File

`config/snapsoccer-tids.txt` — one SincSports tournament ID per line. Populated by the discovery step (see below). Exclude Playdate TIDs.

### Processing Steps

```javascript
// ingest-snapsoccer.js — function signatures and data flow

async function discoverTournamentIds() {
  // Fetch https://snapsoccer.com/events/
  // Extract all href links containing 'soccer.sincsports.com' or 'tid='
  // Filter out PLAYDATE_TIDS = ['SNAPLEAG', 'SNAPLEAF', 'SSSPF']
  // Return: string[] of TIDs
}

async function fetchTournamentGames(tid) {
  // GET https://soccer.sincsports.com/schedule.aspx?tid={tid}
  // Parse HTML: extract all game rows from schedule table
  // Return: RawGame[]
  // RawGame: { home, away, home_score, away_score, date, division, round, event_name }
}

function parseSchedulePage(html, tid) {
  // Parse server-rendered <table> in schedule.aspx response
  // Section headers = division names (iterate <tr class="section-header"> or similar)
  // Game rows = score data (parse score cell: "2 - 1" → home_score=2, away_score=1)
  // Return: RawGame[]
}

async function normalizeGame(raw, tid, gameIndex) {
  // Derive age_group from division string (reuse existing parser)
  // Derive gender from division string
  // Derive season from match_date
  // Build source_id = `snapsoccer_${tid}_${gameIndex}`
  // Return: NormalizedGame (maps directly to games table columns)
}

async function matchOrCreateTeam(name, db) {
  // Pass 1: fuzzy match against teams table
  // Pass 2: insert new team if no match
  // Return: { team_id, matched: boolean }
}

async function insertGame(game, db) {
  // INSERT INTO games (...) VALUES (...) ON CONFLICT (source, source_id) DO UPDATE
  // SET home_score, away_score, updated_at = NOW()
  // Return: 'inserted' | 'updated' | 'skipped'
}

async function run(options) {
  // 1. Load TID list from --tids file or run discoverTournamentIds()
  // 2. For each TID: fetchTournamentGames → normalizeGame → matchOrCreateTeam → insertGame
  // 3. Collect stats: { fetched, inserted, updated, skipped, errors, unmatched_teams }
  // 4. Write Logs/snapsoccer-ingest-{date}.jsonl (one entry per game)
  // 5. Print run summary to stdout
}
```

### Outputs

**Console summary (always):**
```
SnapSoccer ingest complete — 2026-05-01
  TIDs processed:     12
  Games fetched:      847
  Games inserted:     831
  Games updated:      8
  Games skipped:      8   (no scores — likely scheduled games)
  Unmatched teams:    43  (see Logs/snapsoccer-unmatched-teams-2026-05-01.jsonl)
  Errors:             0
  Duration:           34s
```

**Audit log (optional, `--log` flag):**
`Logs/snapsoccer-ingest-{date}.jsonl` — one JSON object per game with insertion result and team match outcome.

---

## 5. Deduplication Strategy

**Primary dedup key:** `(source, source_id)` — the existing `games` table upsert key. SnapSoccer games use `source = 'snapsoccer'` and `source_id = snapsoccer_{tid}_{gameId}`. Re-running the ingestion on the same TID is safe — `ON CONFLICT (source, source_id) DO UPDATE` overwrites scores only, does not duplicate records.

**Cross-source dedup:** SnapSoccer events are SincSports-native. GotSport has no presence for these tournaments. Cross-source duplicate rate expected to be near 0. The existing dedup pipeline (`source_priority = 5`) handles any edge cases post-ingestion — no extra logic needed at ingest time.

**Stable game ID:** If SincSports HTML does not expose stable game IDs in the page, use a deterministic composite:
```
snapsoccer_{tid}_{date}_{homeSlug}_{awaySlug}
```
Where `homeSlug` and `awaySlug` are lowercased, whitespace-stripped team name fragments. This ensures idempotent re-runs even without an explicit game ID in the source HTML.

---

## 6. Pre-Conditions for a May 1 Test Run

- [ ] `snapsoccer.com/events/` page confirmed accessible (no auth wall, links extract SincSports `tid=` parameters)
- [ ] `soccer.sincsports.com/schedule.aspx?tid={TID}` confirmed accessible for at least one competitive event TID
- [ ] Score parsing: confirm scores appear in initial HTML response (not AJAX-loaded) on at least one sample page
- [ ] `games` table schema verified: `(source, source_id)` unique constraint exists for `ON CONFLICT` to work
- [ ] Target TID list: choose 2–3 known competitive events for the test (e.g., Coastal Soccer Invitational, Coastal Academy Cup, Gulf States Showcase)
- [ ] Existing Sinc Sports HTML parsing logic (if it exists) reviewed for reuse

**The most uncertain pre-condition:** Presten needs to manually inspect one `schedule.aspx` page to confirm the table structure matches the parsing assumptions. The research notes ASP.NET WebForms server-rendered HTML, but the specific `<table>` structure needs a live read before writing the parser.

---

## 7. Effort Estimate

| Task | Effort | Owner |
|---|---|---|
| TID discovery (crawl snapsoccer.com/events/) | 1–2 hours | FORGE spec + Presten |
| HTML parser for `schedule.aspx` (adapt Sinc or write new) | 2–4 hours | Presten |
| Field mapping + normalization (age_group, gender, season) | 1 hour | Presten (reuse existing parsers) |
| Test ingest on 3 events + verify | 1–2 hours | Presten |
| Historical event ID discovery + full ingest | 1–2 hours | Presten |
| **Total for test run** | **5–9 hours** | |
| **Total for full production ingest** | **8–12 hours** | |

If the existing Sinc Sports bulk import already has working `schedule.aspx` HTML parsing, total drops to **4–6 hours** (test) / **6–8 hours** (production).

---

## 8. Go / No-Go Recommendation

**Conditional Go — lower priority than Org-ID Expansion.**

| Factor | Assessment |
|---|---|
| Coverage volume | ~10,000–20,000 historical games. Meaningful, but ~10–20× smaller than the Org-ID expansion (20,000–35,000 games from USYS state associations with zero new code) |
| Competitive value | USA Rank uses SnapSoccer. We don't. At DSS, being able to say "we cover SnapSoccer regional events" addresses a known gap in the competitive comparison |
| Effort | 5–9 hours for a test run is real Presten time. The Org-ID expansion is a config change |
| DSS feasibility | A test run before May 9 (last safe implementation date) is feasible if Presten has 5–9 hours free. A full historical ingest before May 16 is feasible if the test run succeeds before May 5 |
| Southeast coverage | SnapSoccer is Alabama/Florida/Georgia/Louisiana-focused. If the DSS demo audience is NJ/NE clubs, this coverage is less impactful for the live demo than Org-ID expansion |

**Recommended sequence:**

1. **Do first:** Org-ID Expansion (USYS state associations) — config change, 20K+ games, highest ROI.
2. **Do second (if time):** SnapSoccer MVP test run — schedule for May 5–8 if Presten has a session.
3. **If time runs out before DSS:** Defer SnapSoccer to post-DSS. The design is complete; a session in late May or June captures the games before they're needed for any June analysis.

---

## Go/No-Go Decision

**Decision: NO-GO for DSS sprint (May 16, 2026 deadline)**

### Criteria Applied

**Go criteria assessment (ALL must be true):**

| Criterion | Threshold | SnapSoccer Result | Pass? |
|---|---|---|---|
| Estimated game count | ≥ 10,000 games/year | 1,500–2,500/year ongoing; 10,000–20,000 historical (one-time catch-up only) | ❌ Fails on annual volume |
| MVP build estimate | ≤ 8 hours total | 5–9h test run; 8–12h full production | ❌ Full production exceeds cap |
| Data accessible without per-org login | No CAPTCHA / no session-per-org | Fully public HTML, no auth required | ✅ |
| Team matching implementable in MVP | No manual review campaign required | Fuzzy match + raw-name insert; merge pipeline handles post-ingest | ✅ |

**Blocking criterion:** Two of four go criteria fail. Full production build estimate (8–12 hours) exceeds the 8-hour cap. Ongoing game volume (1,500–2,500/year) does not meet the 10,000/year threshold — the historical bank of 10,000–20,000 games is a valid one-time catch-up argument, but annual value is modest.

### Future Sprint Trigger

Revisit SnapSoccer when ALL of the following are true:

1. **Org-ID expansion is confirmed in production** — USYS state association teams are being discovered and ingested from GotSport. This is the higher-ROI spend of available Presten time before DSS.
2. **Presten has a 6–8 hour session available** — DSS window doesn't allow this; post-DSS (May 17+) is the earliest realistic window.
3. **Browser test confirms `schedule.aspx` table structure** — the HTML parser spec in Section 4 is based on the known ASP.NET WebForms structure of SincSports. A 20-minute live browser inspection of one SnapSoccer event page should precede any coding to confirm table HTML matches parsing assumptions.
4. **Existing Sinc Sports parser confirmed reusable** — if `crawl-sincsports.js` or equivalent already parses `schedule.aspx`, total effort drops to 4–6 hours (test) / 6–8 hours (production), which changes the calculus.

When pre-conditions 1–4 are met: time estimate for MVP in a post-DSS sprint is **5–8 hours** (assuming parser reuse). Schedule as a standalone session in late May or June.

### What This Decision Closes

SENTINEL can remove "SnapSoccer ingestion — go/no-go pending design review" from the coverage gaps table. The answer: **No for DSS, Yes for post-DSS if conditions above are met.** The design is complete and waiting.

---

## References

- `Data Sources/SnapSoccer Source Research.md` — platform research, URL structure, dedup risk analysis
- `Infrastructure/Database Schema.md` — `games` + `teams` schema
- `Knowledge/USA Rank Competitive Intel.md` — their SnapSoccer usage
- `crawl-gotsport-api-parallel.js` — script pattern to follow
- [[Sinc Sports]] — existing source with same underlying platform; 34,961 games already ingested
