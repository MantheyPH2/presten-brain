---
title: SnapSoccer — Test Extraction Plan
tags: [infrastructure, snapsoccer, sincsports, ingestion, testing, stage3, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: ready — execute after pre-DSS accessibility check confirms soccer.sincsports.com is accessible
task: task-2026-04-27-snapsoccer-test-extraction-plan
reuse_assessment: REBUILD (Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md)
---

# SnapSoccer — Test Extraction Plan

> This plan documents the minimum viable test extraction for SnapSoccer before committing to the full 5.5–9.5 hour build session. Reuse Assessment verdict: **REBUILD** — no existing SincSports HTML parser to adapt. This test extraction plan is designed to validate the build approach against real data before the full session.

---

## Section 1: Test Scope

**Reuse Assessment verdict context:** REBUILD. The full build starts from scratch. The test extraction therefore requires writing a minimal version of the fetch + parse stack (TID discovery skip — use a known TID directly; fetch one page; parse one division's results). Estimated test infrastructure build time: **1.5–2.5 hours**.

### Target Event

**Selection criteria:** Choose a SnapSoccer event where (a) the tournament is listed on `snapsoccer.com/events/` with a visible `?tid=` link to `soccer.sincsports.com/schedule.aspx`, AND (b) the event's teams appear in the Evo Draw DB as existing teams (enabling cross-validation). ECNL events and MLS NEXT events are unlikely — focus on USYS or US Club Soccer tournaments hosted on SnapSoccer.

**Recommended discovery method:** Go to `snapsoccer.com/events/` (or the Events tab). Find a completed event (past date) in a state where Evo Draw has strong coverage (TX, FL, CA). Click through to the schedule page — confirm the URL follows `soccer.sincsports.com/schedule.aspx?tid=XXXX`. Note the TID.

**Target URL pattern:** `https://soccer.sincsports.com/schedule.aspx?tid=[TID]`

**Target URL:** `[TBD — Presten confirms TID from snapsoccer.com/events/ prior to running the test]`

**Expected game count:** Unknown until event is selected. After selecting the event, count visible game rows on the schedule page before running the parser — that count is the expected extraction target.

**Gender and age group for test:** One division only. Recommended: **Boys U15** or **Girls U15** (high match count in most events, tests age/gender parser for both label formats). If the chosen event does not have a U15 division, use the largest available division by visible row count.

---

## Section 2: Test Execution Steps

### Setup

**Requirements:**
- Node.js ≥ 16 in the Evo Draw project environment
- `axios` or `node-fetch` for HTTP requests (already in project dependencies)
- `cheerio` for HTML parsing (may need: `npm install cheerio` if not present)
- Access to the test database (not production — or production with `--dry-run` flag on insert)
- Confirmed TID from Section 1

**Minimal test script:** Create `test-snapsoccer-extraction.js` in the project root. This is a throwaway test script — not production code. It runs one TID and prints extracted rows to stdout. Do not integrate into `run-daily.sh` at this stage.

### Run

**Step 1 — Fetch the schedule page:**
```javascript
const axios = require('axios');
const cheerio = require('cheerio');

const TID = 'REPLACE_WITH_TARGET_TID';
const url = `https://soccer.sincsports.com/schedule.aspx?tid=${TID}`;

async function run() {
  const res = await axios.get(url, {
    headers: { 'User-Agent': 'Mozilla/5.0 (compatible; EvoDrawBot/1.0)' },
    timeout: 15000
  });
  const $ = cheerio.load(res.data);
  // Step 2 below
}
run().catch(console.error);
```

**Step 2 — Parse the division header and game rows (add inside `run()` after fetch):**
```javascript
  // Find division headers and extract target division
  const TARGET_DIVISION = 'REPLACE_WITH_DIVISION_LABEL'; // e.g. "U15 Boys Division 1"
  let currentDivision = null;
  const games = [];

  $('table tr').each((i, el) => {
    const headerText = $(el).find('th').text().trim();
    if (headerText.length > 0) {
      currentDivision = headerText;
      return;
    }
    if (currentDivision !== TARGET_DIVISION) return;

    const cells = $(el).find('td');
    if (cells.length < 5) return;

    const gameDate  = $(cells[0]).text().trim(); // MM/DD/YYYY
    const homeTeam  = $(cells[1]).text().trim();
    const score     = $(cells[2]).text().trim(); // "2-1" or "-"
    const awayTeam  = $(cells[3]).text().trim();
    const roundCell = cells.length > 4 ? $(cells[4]).text().trim() : '';

    const scoreParts = score.includes('-') ? score.split('-') : [null, null];
    games.push({
      game_date:  gameDate,
      home_team:  homeTeam,
      away_team:  awayTeam,
      home_score: scoreParts[0] ? parseInt(scoreParts[0]) : null,
      away_score: scoreParts[1] ? parseInt(scoreParts[1]) : null,
      round:      roundCell || null,
      event_name: $('h1').first().text().trim() || $('title').text().trim(),
      source:     'snapsoccer',
      source_event_id: TID,
    });
  });

  console.log(JSON.stringify(games, null, 2));
  console.log(`\n--- Extracted ${games.length} games from division: ${TARGET_DIVISION} ---`);
```

**Expected output:** JSON array of game objects to stdout. One object per game row in the target division. Example row:
```json
{
  "game_date": "03/15/2026",
  "home_team": "FC Dallas U15",
  "away_team": "Solar SC U15",
  "home_score": 2,
  "away_score": 1,
  "round": "Group Stage",
  "event_name": "SnapSoccer Spring Classic 2026",
  "source": "snapsoccer",
  "source_event_id": "12345"
}
```

### Output Format

- **Format:** JSON to stdout
- **Required fields:** `game_date`, `home_team`, `away_team`, `home_score`, `away_score`, `event_name`, `source`, `source_event_id`
- **Optional fields:** `round` (may be null if the schedule page omits round column)
- **Null scores:** Acceptable for games not yet played or games with "- " score. If score is null in a completed event, flag as a parser issue.

### Validation Queries

After extraction, confirm test data loaded correctly (run only if inserting into the test DB — skip if stdout-only mode):

```sql
-- Confirm test games are present
SELECT game_date, home_team, away_team, home_score, away_score, source, event_name
FROM games
WHERE source = 'snapsoccer'
  AND source_event_id = '[TARGET_TID]'
ORDER BY game_date, home_team;

-- Confirm no duplicates
SELECT home_team, away_team, game_date, COUNT(*) AS count
FROM games
WHERE source = 'snapsoccer'
  AND source_event_id = '[TARGET_TID]'
GROUP BY home_team, away_team, game_date
HAVING COUNT(*) > 1;
```

---

## Section 3: Pass/Fail Criteria

| Check | Pass | Fail |
|-------|------|------|
| Game count | Extracted count matches visible page count ± 5% | Count differs by > 10% or is 0 |
| Field completeness | `home_team`, `away_team`, `game_date`, `event_name` all non-null in ≥ 95% of rows | Any required field is null in > 5% of rows |
| Score parsing | `home_score` and `away_score` parse as integers for completed games | Scores garbled, negative, or null for completed games |
| Team name format | Names are legible; recognizable as soccer team names | Names truncated, HTML-escaped, or contain table row artifacts |
| Duplicate detection | Zero duplicate `home_team + away_team + game_date` combinations | Any duplicates present |
| Error rate | Script completes with 0 uncaught exceptions | Any uncaught exception (Cloudflare block, parse crash, timeout) |
| Accessibility (pre-test gate) | `soccer.sincsports.com/schedule.aspx?tid=TID` returns HTTP 200 with HTML content | HTTP 403 / Cloudflare challenge / CAPTCHA — stop immediately and escalate |

**If all checks pass: test PASS** — proceed to schedule full build session (post-DSS).

**If any check fails:** Document the failure. Update the Reuse Assessment verdict only if the failure indicates structural incompatibility beyond what was already assessed. Escalate to SENTINEL before scheduling a full build session.

---

## Section 4: Pre-DSS vs. Post-DSS Recommendation

**Reuse Assessment verdict:** REBUILD. The test itself requires writing the fetch + parse stack from scratch — the test infrastructure is not a trivial 30-minute adaptation. Estimated test infrastructure time: **1.5–2.5 hours**.

**Can this test run before May 16 (DSS)?**

Criteria:
- (a) Requires < 1 hour standalone: **No.** With REBUILD verdict, the test script is not adaptable — it must be written from scratch. Build + test = 1.5–2.5 hours minimum.
- (b) Existing SincSports parser requires ≤ 30 minutes of adaptation: **No.** No parser exists to adapt.
- (c) Does not touch production data: **Yes** — the test script outputs to stdout; no DB writes required.

**Criteria (a) and (b) are not met.**

**Recommended window: May 17+ post-DSS.** The test extraction should be the opening 1.5–2.5 hour block of the full SnapSoccer build session — it doubles as build validation and full session kickoff. There is no benefit to running it pre-DSS given it requires the same infrastructure setup as the beginning of the full build.

**Exception condition:** If Presten has a 3-hour window before May 16 and wants to derisk the SnapSoccer build, the test extraction can run pre-DSS as a standalone session. The only pre-condition is: (1) `soccer.sincsports.com` accessibility confirmed, and (2) SENTINEL authorizes the time investment pre-DSS.

**Risk if test is skipped entirely (going straight from Reuse Assessment to full build):** The Reuse Assessment documented the data model but could not verify actual HTML structure on `schedule.aspx` (codebase/site access limitation noted in Assessment Section 1). Going directly to full build without a test extraction means the HTML parser assumptions (table structure, division header format, score cell format) are unvalidated. Risk: the first 1–2 hours of the full build session may be spent debugging table structure assumptions. Low probability of total failure; medium probability of 1–2 hour delay.

---

## Section 5: Full Build Session Prerequisites

Before Presten schedules the full SnapSoccer build session:

- [ ] **Test extraction PASS confirmed** — all Section 3 criteria pass
- [ ] **`soccer.sincsports.com` accessibility confirmed** — Presten loads `soccer.sincsports.com/schedule.aspx?tid=[any valid TID]` in a browser and confirms: HTTP 200, no Cloudflare block, schedule table visible (30-second check)
- [ ] **Reuse Assessment filed** — `Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md` — REBUILD verdict filed ✅ (complete as of 2026-04-26)
- [ ] **Post-DSS sprint window open** — May 17+ (DSS is May 16)
- [ ] **SENTINEL has authorized SnapSoccer as first post-DSS ingestion priority** — confirm against Source Coverage Priority Matrix order

**Recommended build session scope:** 6 hours (REBUILD, mid-estimate). Use `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md` as the implementation spec. Use `Data Sources/SnapSoccer Source Research.md` as the scraper skeleton foundation. Reference the GotSport HTML scraper's architecture for the HTML parser build pattern.

---

## References

- `Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md` — REBUILD verdict and build scope (complete)
- `Data Sources/SnapSoccer Source Research.md` — scraper architecture and TID URL structure
- `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md` — full implementation design (use as full build spec)
- `Infrastructure/Non-GotSport Top Sources — Ingestion Design.md` — effort estimate source
- `Data Sources/Sinc Sports.md` — bulk import origin of existing 34,961 SincSports games

*FORGE — 2026-04-26*
