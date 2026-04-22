---
title: SnapSoccer Source Research
tags: [data-sources, snapsoccer, scraping, competitive-response, evo-draw]
created: 2026-04-23
updated: 2026-04-23
status: research-complete
recommendation: conditional-go
---

# SnapSoccer Source Research Memo

**Researched by:** FORGE  
**Date:** 2026-04-23  
**Priority:** HIGH — Confirmed USA Rank source

---

## 1. Go / No-Go Recommendation

**CONDITIONAL GO** — with important qualification.

SnapSoccer's competitive tournaments (Coastal Soccer Invitational, Coastal Academy Cup, Panama City Beach Classic, Gulf States Showcase, Publix SuperCup, etc.) are managed on **SincSports** (`soccer.sincsports.com`), and scores are publicly accessible via server-rendered HTML. These tournament events are legitimate, scoreable, and targetable.

**Critical caveat:** SnapSoccer's "Playdates" series (the events USA Rank references as "SnapSoccer Fall Playdates" / "Spring Playdates") are explicitly **development-only — no scores kept**. These Playdate events cannot be scraped for game results because none are recorded. The Playdate tournament IDs (`SNAPLEAG`, `SNAPLEAF`, `SSSPF`) will return schedule data but no scores.

**Therefore:** Build the scraper targeting SnapSoccer's **competitive tournaments** on SincSports, skip the Playdate series. The Playdate branding in USA Rank's event list likely means USA Rank is pulling the schedule/team data but not actual scored results either — or they've indexed it as a known event family without game data.

---

## 2. Estimated Game Volume

SnapSoccer runs **25+ soccer weekends per year**, primarily across Alabama, Florida, Georgia, and Louisiana (Gulf Coast / Southeast). Events range from ~50 to 200+ teams.

Rough volume estimate:
- 10–15 scoreable competitive events per year
- Average 8 fields × 8 games/day × 2 days = ~128 games per event
- **Estimated annual game volume: 1,500–2,500 games/year**
- Historical archive on SincSports likely spans 5–8 years → **7,500–20,000 historical games**

Order of magnitude: **~10,000–20,000 total games** in the SnapSoccer universe on SincSports. This is modest relative to GotSport (1.49M) but meaningful for Southeast regional coverage, and it is a confirmed USA Rank source we currently lack entirely.

---

## 3. Scraper Approach

### Backend Platform
SnapSoccer competitive events are hosted on **SincSports** (`soccer.sincsports.com`).

SincSports uses **server-rendered HTML** via classic ASP.NET WebForms (`.aspx` pages). No client-side JSON fetch is required. All schedule and score data renders in the initial HTML response.

### Auth Required?
**No authentication required.** SincSports schedule and results pages are fully public. The URL pattern is:
```
https://soccer.sincsports.com/schedule.aspx?tid={TOURNAMENT_ID}
```

### Key URLs to Hit

| Page | URL Pattern | Data Available |
|------|-------------|----------------|
| Tournament schedule/scores | `soccer.sincsports.com/schedule.aspx?tid={TID}` | All games, scores, dates, divisions |
| Tournament info | `soccer.sincsports.com/TTMore.aspx?tid={TID}` | Event name, age groups, dates |
| Club teams in event | `soccer.sincsports.com/TTClub.aspx?tid={TID}` | Team names, club affiliations |
| Event details | `soccer.sincsports.com/details.aspx?tid={TID}` | Event metadata |

### Known SnapSoccer Tournament IDs (SincSports)
| Tournament | TID |
|------------|-----|
| SnapSoccer Spring Playdates (NO SCORES) | `SNAPLEAG` |
| SnapSoccer Fall Playdates (NO SCORES) | `SNAPLEAF`, `SSSPF` |
| Competitive events | Must be discovered via event listing crawl |

To discover all SnapSoccer competitive tournament IDs, crawl `snapsoccer.com/events/` and extract SincSports `tid=` parameters from linked result pages.

### Scraper Template
**Best template: existing Sinc Sports bulk import pipeline.** We already have 34,961 SincSports games imported. The Sinc Sports scraper is the correct foundation — SnapSoccer events are simply more tournament IDs to feed into the same HTML parsing logic.

If a live Sinc Sports scraper does not exist (current import was bulk/manual), the **GotSport HTML Scraper** is the second-best template for `.aspx` server-rendered HTML parsing.

### HTTP Structure
- Server-rendered ASP.NET HTML (`.aspx`)
- Standard HTTP GET requests, no JavaScript execution required
- Scores and schedules present in initial HTML response
- Pagination: Likely single-page per tournament, or tabbed by division

### Rate Limit
SincSports does not publicly document rate limits. Safe approach: **800–1000ms between requests**. With ~15–30 event pages to crawl, a full SnapSoccer crawl completes in under 1 minute — rate limits are a non-issue at this volume.

### Cloudflare / Bot Protection
SincSports is an `.aspx` site with no known Cloudflare protection (unlike Sinc Sports being noted as protected in current docs — clarify if `soccer.sincsports.com` vs `sincsports.com` differ). Standard HTTP fetch should work.

---

## 4. Dedup Risk

**LOW overlap risk.**

SnapSoccer events are Southeast regional tournaments hosted exclusively on SincSports. GotSport (our primary source) operates nationally and is a separate platform. The same game appearing in both GotSport and SincSports is theoretically possible if a tournament director cross-lists, but SnapSoccer events are SincSports-native — there is no GotSport presence for these events.

We already handle Sinc vs GotSport dedup (Sinc is priority 5, GotSport is 1/2). SnapSoccer games inserted as SincSports games would follow the same dedup rules automatically.

---

## 5. Estimated Dev Effort

| Task | Effort |
|------|--------|
| Crawl `snapsoccer.com/events/` to extract SincSports `tid=` parameters | 1–2 hours |
| Adapt existing Sinc HTML parser to `schedule.aspx` format | 2–4 hours |
| Map SnapSoccer field names → `games` table columns | 1–2 hours |
| Test on 3 known events, verify score parsing | 1–2 hours |
| Discovery loop for historical event IDs | 1–2 hours |
| **Total** | **6–12 hours** |

If the existing Sinc Sports bulk import already has HTML parsing logic, effort drops to **4–6 hours**.

---

## 6. Proposed Source Priority

**Priority 5** — same as existing Sinc Sports source.

SnapSoccer events ARE SincSports events. Inserting them under the same `source = 'sinc'` family with `source_priority = 5` is correct and requires no changes to the dedup priority system. Alternatively, use `source = 'snapsoccer'` with `source_priority = 5` to track them separately for reporting.

Recommendation: use `source = 'snapsoccer'` for attribution clarity while keeping priority 5.

---

## 7. Source ID Format

```
snapsoccer_{tid}_{gameId}
```

**Example:** `snapsoccer_COASTALINV_887234`

Where:
- `tid` = SincSports tournament ID (e.g., `COASTALINV`, `ACADCUP`)
- `gameId` = SincSports internal game ID or sequence number from the schedule page

If SincSports does not expose stable game IDs in the HTML, use a deterministic composite:
```
snapsoccer_{tid}_{date}_{homeTeamSlug}_{awayTeamSlug}
```

---

## Skeleton Scraper Outline

### Step 1: Tournament Discovery

```javascript
// crawl-snapsoccer.js — Step 1: Discover SincSports tournament IDs
const SNAPSOCCER_EVENTS_URL = 'https://snapsoccer.com/events/';

async function discoverTournamentIds() {
  const html = await fetch(SNAPSOCCER_EVENTS_URL).then(r => r.text());
  // Extract all href links containing 'soccer.sincsports.com' or 'tid='
  const tidRegex = /tid=([A-Z0-9]+)/gi;
  const tids = [...html.matchAll(tidRegex)].map(m => m[1]);
  // Deduplicate and filter out known Playdate TIDs (no scores)
  const PLAYDATE_TIDS = ['SNAPLEAG', 'SNAPLEAF', 'SSSPF'];
  return [...new Set(tids)].filter(tid => !PLAYDATE_TIDS.includes(tid));
}
```

### Step 2: Schedule/Score Fetch

```javascript
// Step 2: Fetch and parse schedule.aspx for each TID
async function fetchTournamentGames(tid) {
  const url = `https://soccer.sincsports.com/schedule.aspx?tid=${tid}`;
  const html = await fetch(url).then(r => r.text());
  return parseSchedulePage(html, tid);
}
```

### Step 3: HTML Parsing (Key Fields)

```javascript
function parseSchedulePage(html, tid) {
  // SincSports schedule.aspx renders a table per division
  // Fields to extract from each game row:
  const games = [];
  // Parse each <tr> in the schedule table:
  //   - game_date: date column (MM/DD/YYYY)
  //   - home_team_name: home team cell
  //   - away_team_name: away team cell
  //   - home_score: score cell (left of dash)
  //   - away_score: score cell (right of dash)
  //   - division: section header above the table
  //   - round: "Group Stage", "Semifinal", "Final", etc. (if present)
  //   - event_name: page title / meta
  return games;
}
```

### Step 4: Insertion Logic Stub

```javascript
async function insertGame(game, tid, gameId) {
  const sourceId = `snapsoccer_${tid}_${gameId}`;
  await db.query(`
    INSERT INTO games (source, source_id, source_priority,
      event_name, home_team_name, away_team_name,
      home_score, away_score, match_date, division,
      age_group, gender, game_status, season)
    VALUES ($1, $2, 5, $3, $4, $5, $6, $7, $8, $9, $10, $11, 'completed', $12)
    ON CONFLICT (source, source_id) DO UPDATE
      SET home_score = EXCLUDED.home_score,
          away_score = EXCLUDED.away_score,
          updated_at = NOW()
  `, ['snapsoccer', sourceId, game.event_name, game.home_team,
      game.away_team, game.home_score, game.away_score,
      game.date, game.division, game.age_group, game.gender, game.season]);
}
```

### Data Fields to Extract

| Field | Source in HTML | Notes |
|-------|----------------|-------|
| `event_name` | Page `<title>` or `<h1>` | e.g., "Coastal Soccer Invitational" |
| `home_team_name` | Schedule table row | Cell 1 or 2 |
| `away_team_name` | Schedule table row | Cell 2 or 3 |
| `home_score` | Score cell | Parse "2-1" → 2 |
| `away_score` | Score cell | Parse "2-1" → 1 |
| `match_date` | Date column | Parse to ISO date |
| `division` | Section header | e.g., "U13 Boys Division 1" |
| `age_group` | Parsed from division | Run through existing backfill logic |
| `gender` | Parsed from division | Boys/Girls |
| `round` | Column if present | Group/Semi/Final |

---

## Summary

| Dimension | Assessment |
|-----------|------------|
| Recommendation | **CONDITIONAL GO** (competitive events only, skip Playdates) |
| Game volume | ~10,000–20,000 historical games |
| Scraper approach | HTML scrape of `soccer.sincsports.com/schedule.aspx` |
| Auth required | None |
| Dedup risk | **LOW** (SincSports-native, no GotSport overlap) |
| Dev effort | **6–12 hours** |
| Source priority | **5** (same tier as Sinc Sports) |
| Source ID format | `snapsoccer_{tid}_{gameId}` |

> [!warning] Playdates Have No Scores
> Do NOT scrape SnapSoccer Playdate events (TIDs: SNAPLEAG, SNAPLEAF, SSSPF). They are explicitly score-free by design. Only scrape competitive tournament events.

---

*Researched by FORGE — 2026-04-23*
