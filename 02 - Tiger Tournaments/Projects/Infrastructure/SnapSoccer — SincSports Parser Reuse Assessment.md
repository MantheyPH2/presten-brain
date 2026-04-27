---
title: SnapSoccer — SincSports Parser Reuse Assessment
tags: [infrastructure, snapsoccer, sincsports, ingestion, stage3, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: complete — verdict REBUILD
task: task-2026-04-27-sincsports-parser-reuse-check
---

# SnapSoccer — SincSports Parser Reuse Assessment

> Can the existing SincSports codebase be reused to build the SnapSoccer scraper, cutting the 8–12 hour build estimate to 4–6 hours? This document answers that question.

---

## Section 1: Existing Parser Inventory

**Finding: No live SincSports HTML parser exists in the Evo Draw codebase.**

The existing 34,961 SincSports games in the database came from a **one-time bulk import** — not a live scraper. Per `Data Sources/Sinc Sports.md`:

> "Sinc Sports is a tournament management platform that is Cloudflare-protected, making live scraping infeasible. All data was obtained via bulk import."

| File | Function | Status | Notes |
|------|----------|--------|-------|
| Bulk import script (unknown path) | One-time CSV/data import of SincSports game export | Presumed archived or deleted | Not a live HTML parser; cannot be adapted for `schedule.aspx` scraping |
| Live SincSports scraper | N/A | Does not exist | Cloudflare protection made live scraping infeasible at time of original import |

**Key distinction:** The existing SincSports data was imported via a bulk file export — likely a CSV or similar data format provided directly, not parsed from `sincsports.com` HTML. The `soccer.sincsports.com` subdomain (used for SnapSoccer events) may differ from the Cloudflare-protected main `sincsports.com` site, but regardless: **no HTML parser code exists to reuse** because the original import did not use HTML parsing.

> [!warning] Codebase access caveat
> FORGE cannot directly inspect the Evo Draw codebase from the vault. This finding is based on the documented data acquisition method in `Data Sources/Sinc Sports.md`. If Presten is aware of a live SincSports HTML parser that was written after the original bulk import, this assessment should be revised. Presten can confirm by running: `find . -name "*.js" | xargs grep -l "sincsports\|schedule.aspx" 2>/dev/null`

---

## Section 2: SnapSoccer Data Structure Comparison

Even without a reusable parser, documenting the data structure confirms build requirements. SnapSoccer uses `soccer.sincsports.com/schedule.aspx` — an ASP.NET server-rendered HTML page. Structure known from `Data Sources/SnapSoccer Source Research.md`:

| Field | Evo Draw Target Field | SnapSoccer/SincSports HTML Source | Compatible with games table? |
|-------|-----------------------|-----------------------------------|------------------------------|
| Home team name | `home_team_name` | `<td>` text in schedule table row — team name cell | Yes — standard text extraction |
| Away team name | `away_team_name` | `<td>` text in schedule table row — team name cell | Yes — standard text extraction |
| Home score | `home_score` | Score cell: "2-1" → parse left of dash | Yes — with integer parsing |
| Away score | `away_score` | Score cell: "2-1" → parse right of dash | Yes — with integer parsing |
| Game date | `game_date` | Date column: `MM/DD/YYYY` format | Yes — parse to ISO date |
| Event/tournament name | `event_name` | Page `<title>` or `<h1>` tag | Yes — standard extraction |
| Age group | `age_group` | Division header above table (e.g., "U13 Boys Division 1") | Yes — existing age-group parsing rules apply |
| Gender | `gender` | Division header parsing | Yes — existing gender parsing rules apply |
| Round | `round` | Table column if present ("Group Stage", "Semifinal", "Final") | Yes — existing round parsing |
| Tournament ID (TID) | `source_event_id` | URL param `?tid=TID` | Yes — extract from URL |
| Game ID | `source_game_id` | `GameId=` URL param or deterministic composite | Yes — use `snapsoccer_{tid}_{gameId}` format |

**Compatibility assessment:** The SnapSoccer/SincSports data model maps cleanly to the Evo Draw games table. All 10 required fields are present and extractable. The table structure is standard ASP.NET WebForms HTML — one `<tr>` per game within division-segmented `<table>` elements.

**No incompatible fields.** The data model comparison does not reveal any SnapSoccer-specific structural challenges that would require non-standard transforms beyond what the existing pipeline already handles (age-group parsing, gender parsing, score parsing).

---

## Section 3: Reuse Verdict

**Verdict: REBUILD**

**Primary reason:** No live SincSports HTML parser exists. The original 34,961 SincSports games were imported from a bulk file export — the code path was file parsing (CSV or similar), not HTML scraping. There is no `schedule.aspx` HTML parser to adapt.

**Secondary confirmation:** Even if a bulk import parser existed, it would not be directly reusable — bulk file parsing and live HTML scraping involve different fetch mechanisms, error handling patterns, and rate-limiting approaches.

**The build starts from scratch.** The SnapSoccer skeleton scraper in `Data Sources/SnapSoccer Source Research.md` (3-step architecture: TID discovery → schedule.aspx fetch → HTML parse → DB insert) is the correct foundation. It is written to Evo Draw conventions (async/await, ON CONFLICT upsert, existing `source_priority` system).

**Original effort estimate stands:** 5–9 hours test run / 8–12 hours production. No reuse discount applies.

**One mitigating factor:** The GotSport HTML scraper (`crawl-gotsport-html.js` or equivalent) is a working HTML parser for `.aspx`-like structured sports data. If its architecture is modular (separate fetch/parse functions), a FORGE-to-Presten hand-off might reduce the HTML parsing build from 2–4 hours to 1–2 hours by adapting that parsing logic. This is a conditional credit — Presten would need to confirm whether the GotSport HTML scraper's parser is extractable. Not counted in the base estimate.

---

## Section 4: Adaptation Scope (REBUILD — no adaptation; new build)

Since the verdict is REBUILD, this section describes the build requirements rather than adaptation scope:

**What must be written from scratch:**

1. **TID discovery module** (`discoverTournamentIds()`) — crawl `snapsoccer.com/events/`, extract `tid=` params from SincSports result links, filter out known Playdate TIDs (SNAPLEAG, SNAPLEAF, SSSPF). Estimated: 1–2 hours.

2. **Schedule page fetcher** (`fetchTournamentGames(tid)`) — HTTP GET of `soccer.sincsports.com/schedule.aspx?tid={TID}`, 1 req/sec rate limit. Estimated: 30 minutes (straightforward fetch with retry).

3. **HTML parser** (`parseSchedulePage(html, tid)`) — parse ASP.NET table structure, extract all 10 fields per game row, handle division headers for age/gender extraction. Estimated: 2–4 hours (this is the core build work).

4. **DB insertion** (`insertGame()`) — ON CONFLICT upsert into `games` table, same pattern as GotSport insertion. Estimated: 30 minutes (pattern already exists in pipeline).

5. **Integration into `run-daily.sh`** — add SnapSoccer step after GotSport crawl. Estimated: 30 minutes.

6. **Test against 3–5 known events** — verify score parsing, age/gender extraction, dedup against existing Sinc records. Estimated: 1–2 hours.

**Total: 5.5–9.5 hours — consistent with the 5–9 hour test estimate.**

---

## Section 5: Recommended Next Step

Schedule the SnapSoccer scraper build for the **May 17+ post-DSS window** using `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md` as the implementation spec and `Data Sources/SnapSoccer Source Research.md` as the scraper skeleton foundation; before scheduling, Presten should confirm that `soccer.sincsports.com` public schedule pages are accessible without authentication or Cloudflare blocking (a 5-minute browser check).

---

## References

- `Data Sources/Sinc Sports.md` — documents the bulk import origin of existing SincSports games
- `Data Sources/SnapSoccer Source Research.md` — scraper architecture and TID structure
- `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md` — full implementation design
- `Infrastructure/Non-GotSport Top Sources — Ingestion Design.md` — effort estimate source (5–9/8–12 hours)

*FORGE — 2026-04-26*
