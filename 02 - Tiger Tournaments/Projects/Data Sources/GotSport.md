---
title: GotSport
aliases: [GotSport, GotSport API, GotSport HTML, system.gotsport.com]
tags: [data-source, gotsport, scraper, api, evo-draw]
created: 2026-04-21
updated: 2026-04-21
source_priority_api: 1
source_priority_html: 2
total_games: 1489861
visible_games: 1487049
---

# GotSport

GotSport is the **PRIMARY** data source for Evo Draw, accounting for ~90% of all game data. It is the dominant tournament management platform in US youth soccer. Data is collected via two independent scrapers.

---

## Two Scrapers

### 1. API Scraper (Priority 1) — [[GotSport API Scraper]]
- **File:** `crawl-gotsport-api-parallel.js`
- **Endpoint:** `https://system.gotsport.com/api/v1/teams/{id}/matches`
- **Auth:** None required (public API)
- **Rankings endpoint:** `https://system.gotsport.com/api/v1/team_ranking_data` — also no auth
- **Source ID format:** `gs_api_{eventId}_{matchId}`
- **Team discovery:** 107,528 team IDs harvested from the rankings endpoint
- Runs with 3 parallel workers, 1200ms delay each

### 2. HTML Scraper (Priority 2) — [[GotSport HTML Scraper]]
- **File:** `crawl-gotsport-v2.js`
- **Method:** Fetches server-rendered HTML from `system.gotsport.com` event pages
- **Source ID format:** `gs_{eventId}_{matchNum}`
- **Parsing:** Schedule tables with score extraction (including penalty shootouts)
- 3-layer filter: HARD_SKIP keywords, page content scan, youth age verification

> [!info] Why Two Scrapers?
> The API scraper is team-centric (fetch all matches for a team ID), while the HTML scraper is event-centric (fetch all games in an event). The API provides richer structured data and gets priority 1. The HTML scraper catches events/games that may not appear via the team-based API approach.

---

## Source ID Formats

| Scraper | Pattern                      | Example                    |
| ------- | ---------------------------- | -------------------------- |
| API     | `gs_api_{eventId}_{matchId}` | `gs_api_31228_4587321`     |
| HTML    | `gs_{eventId}_{matchNum}`    | `gs_45488_127`             |

When both scrapers capture the same game, the [[Dedup Strategy]] keeps the API version (`source_priority=1`) and marks the HTML version as `is_hidden=true`.

---

## Key Data Points

- **107,528** team IDs discovered from rankings endpoint
- **1,489,861** total games collected
- **1,487,049** visible after dedup
- The gap (~2,800 hidden) represents within-source duplicates between API and HTML

---

## Upcoming Changes

> [!warning] ECNL Migration (June 2026)
> [[ECNL]] is transitioning from [[TGS AthleteOne]] to GotSport in June 2026. This will bring a significant influx of ECNL league games into the GotSport ecosystem. The [[GotSport API Scraper]] will automatically pick up these games as ECNL teams get GotSport team IDs.

---

## Related Notes
- [[Data Sources Overview]] — volume comparison across all sources
- [[GotSport API Scraper]] — parallel crawler details
- [[GotSport HTML Scraper]] — HTML parser details
- [[Dedup Strategy]] — how API vs HTML duplicates are resolved
- [[Team Name Matching]] — GotSport name mismatches with DSS registration
- [[Parsing Rules]] — score, age, gender extraction from GotSport data
