---
title: GotSport API Scraper
aliases: [crawl-gotsport-api-parallel.js, GS API Scraper]
tags: [scraper, gotsport, api, parallel, evo-draw]
created: 2026-04-21
updated: 2026-04-23
file: crawl-gotsport-api-parallel.js
source_priority: 1
---

# GotSport API Scraper

The **PRIMARY** scraper for Evo Draw. Fetches structured game data from the [[GotSport]] public API using parallel workers.

---

## Files

| File                              | Purpose                   |
| --------------------------------- | ------------------------- |
| `crawl-gotsport-api-parallel.js`  | **Primary** — 3 parallel workers |
| `crawl-gotsport-api.js`           | Sequential fallback       |

---

## Configuration

| Parameter       | Value                                          |
| --------------- | ---------------------------------------------- |
| Workers         | 3 parallel                                     |
| Delay           | 1200ms between requests per worker             |
| Auth            | **None required** (public API)                 |
| Base URL        | `https://system.gotsport.com/api/v1`           |
| Matches endpoint| `/teams/{id}/matches`                          |
| Rankings endpoint| `/team_ranking_data`                          |

### Environment Variables

| Variable            | Purpose                              |
| ------------------- | ------------------------------------ |
| `GSAPI_SKIP_RANKINGS` | Skip the rankings discovery step   |
| `GSAPI_TEAM_IDS`    | Comma-separated list of specific team IDs to scrape |
| `GSAPI_AGES`        | Filter by age groups                 |
| `GSAPI_GENDER`      | Filter by gender                     |

---

## How It Works

1. **Team Discovery:** Fetches all team IDs from `/api/v1/team_ranking_data` — discovered **107,528 team IDs**
2. **Match Fetching:** For each team ID, fetches `/api/v1/teams/{id}/matches`
3. **Parallel Processing:** 3 workers process the team ID queue concurrently
4. **Upsert:** Games inserted/updated via `(source, source_id)` upsert into [[Database Schema|games table]]

Source ID format: `gs_api_{eventId}_{matchId}`

> [!info] No Authentication
> The GotSport API is publicly accessible with no API key or authentication required. Rate limiting is handled by the 1200ms base delay between requests per worker, plus exponential backoff with jitter on 429 responses (implemented 2026-04-23).

---

## Structural Fixes (Implemented 2026-04-23)

Four structural improvements were implemented following the 2026-04-22 stale team investigation:

| Fix | Behavior |
|-----|---------|
| **Exponential Backoff + Jitter** | On 429/503: retry up to 4× with `delay = 1200ms * (2^attempt) + jitter(0, 500ms)`, cap 30s. Team ID goes to DLQ after retry exhaustion. Stops entire run on 403 (auth wall signal). |
| **Dead Letter Queue (DLQ)** | Worker crashes and retry exhaustion write team IDs to `Logs/dlq/dlq-{runId}.jsonl`. No team ID can be silently lost. |
| **Empty-Payload Detection** | HTTP 200 with `matches: []` does NOT update `last_synced_at`. Logged at `warn` level as `EMPTY_PAYLOAD`. Three consecutive empty runs escalates to `PERSISTENT_EMPTY_PAYLOAD`. |
| **Per-Run Summary Log** | Every run ends with a `SYNC_RUN_SUMMARY` log line: `teams_discovered / fetched_success / empty_payload / dlq / upsert_failed / duration_ms`. Upsert failures elevated from `debug` to `warn`. |

---

## Fallback: Sequential Scraper

`crawl-gotsport-api.js` is a single-threaded version used as a fallback when:
- Debugging specific team IDs
- Avoiding rate limit issues
- Running targeted scrapes

---

## Related Notes
- [[GotSport]] — source overview (1.49M games)
- [[GotSport HTML Scraper]] — complementary HTML-based scraper
- [[Data Sources Overview]] — priority 1 source
- [[Dedup Strategy]] — API (pri 1) vs HTML (pri 2) resolution
- [[Data Pipeline]] — processing after scraping
