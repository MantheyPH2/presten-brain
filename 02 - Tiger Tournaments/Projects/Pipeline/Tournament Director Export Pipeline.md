---
title: Tournament Director Export Pipeline
aliases: [Export Pipeline, Event Upload Pipeline, TD Export]
tags: [pipeline, gotsport, resilience, tournament-director, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: FORGE
status: design spec — ready for sprint planning
priority: medium
---

# Tournament Director Export Pipeline — Design Spec

> Part of [[Data Pipeline]] | Companion to [[Webhook Fallback Scope]]

**Author:** FORGE  
**Date:** 2026-04-23  
**Scope:** Full design specification for the voluntary event export ingestion pipeline — the primary GotSport API dependency mitigation identified in `Pipeline/Webhook Fallback Scope.md`

> [!info] Why This Pipeline Exists
> GotSport does not support outbound webhooks. The v2 spec webhook approach is not buildable without GotSport's cooperation. The tournament director export pipeline is the highest-value buildable alternative: it creates a human-mediated data channel that works under all three risk scenarios, and it creates a strategic relationship with tournament directors that supports the division placement-as-a-service business model.

---

## Fallback Scenario Coverage

This pipeline directly addresses the scenarios from [[Webhook Fallback Scope]]:

| Scenario | Coverage | Notes |
|---|---|---|
| A: API rate-limited (current) | Partial | Supplements API data; does not replace team discovery |
| B: API moves to auth-only | Yes | Tournament directors can export + upload regardless of API auth state |
| C: Full Cloudflare block | Yes | Manual exports are human-triggered downloads — not affected by bot protection |

**What this pipeline does NOT cover:** Team discovery. The 107,528 team IDs from the API rankings endpoint cannot be replicated via manual exports. If the API becomes inaccessible, team coverage narrows to teams that have participated in uploaded events. This is a known and accepted limitation — the tournament director pipeline is a resilience measure, not a full API replacement.

---

## Architecture Overview

```
Tournament Director
        │
        │ exports CSV from GotSport dashboard
        ▼
POST /api/ingest/event-export
   (multipart/form-data)
        │
        ▼
Upload Validation
  ├── file size ≤ 10MB
  ├── CSV format check
  ├── required fields present
  └── auth check (TD account)
        │
        ▼
source_submissions record created
  (status: queued, job_id returned)
        │
        ▼
Async: parse-event-export.js
  ├── CSV parse
  ├── row filtering (Final only)
  ├── division → age/gender
  ├── team name → team_id resolution
  └── dedup check vs API records
        │
        ▼
games table upsert
  (source: gs_export, priority: 3)
        │
        ▼
Confirmation screen
  ├── games ingested count
  ├── teams updated count
  ├── link to rankings filtered to event
  └── DSS pitch CTA
```

---

## Component 1: Upload Endpoint

### Route

```
POST /api/ingest/event-export
```

### Authentication

Required. The uploading user must have a tournament director account on Evo Draw. This is deliberate — it creates the relationship and establishes that the data was contributed by an authenticated, accountable party.

For the MVP, "authentication" can be an API key issued to tournament directors manually by Presten. This avoids building a full auth system for MVP. Key format: `td_key_{uuid}` in the `Authorization: Bearer` header.

### Request Format

```
Content-Type: multipart/form-data

Fields:
  file          CSV file — the GotSport event export
  tournament_name   string — human-readable event name (required)
  event_id      integer — GotSport event ID if known (optional but strongly encouraged)
  date_start    date — event start date (required, format: YYYY-MM-DD)
  date_end      date — event end date (required, format: YYYY-MM-DD)
  submitter_name  string — name of the tournament director (required for accountability)
```

### Validation

```javascript
// File size
if (file.size > 10 * 1024 * 1024) {
  return res.status(400).json({ error: 'File too large. Maximum 10MB.' });
}

// CSV format check — must have expected headers
const requiredHeaders = ['Match ID', 'Home Team', 'Away Team', 'Home Score', 'Away Score', 'Division', 'Status'];
const fileHeaders = parseCSVHeaders(file);
const missingHeaders = requiredHeaders.filter(h => !fileHeaders.includes(h));
if (missingHeaders.length > 0) {
  return res.status(400).json({ 
    error: 'Invalid CSV format', 
    missing_fields: missingHeaders,
    hint: 'Please export using GotSport\'s Results CSV format from Event Management > Export.'
  });
}

// Date range sanity check
if (new Date(date_start) > new Date(date_end)) {
  return res.status(400).json({ error: 'date_start must be before date_end' });
}
```

### Response

```json
{
  "status": "queued",
  "job_id": "export-job-u4892",
  "message": "Your event export is being processed. Check back in 60 seconds.",
  "status_url": "/api/ingest/status/export-job-u4892"
}
```

Processing is asynchronous. The endpoint returns immediately with a job ID. The client polls `/api/ingest/status/{job_id}` for completion.

### Node.js Handler Stub

```javascript
// In server.js — add this handler to the existing route list
// No Express required — uses plain Node.js http module with busboy for multipart

const busboy = require('busboy');
const { v4: uuidv4 } = require('uuid');

if (req.method === 'POST' && req.url === '/api/ingest/event-export') {
  const apiKey = (req.headers['authorization'] || '').replace('Bearer ', '');
  
  // Validate API key
  const user = await db.query(
    `SELECT submitter_id, name FROM tournament_directors WHERE api_key = $1`,
    [apiKey]
  );
  if (user.rowCount === 0) {
    res.writeHead(401); res.end(JSON.stringify({ error: 'Invalid API key' })); return;
  }
  
  const jobId = `export-job-${uuidv4().split('-')[0]}`;
  const bb = busboy({ headers: req.headers });
  let csvBuffer = Buffer.alloc(0);
  let metadata = {};
  
  bb.on('file', (name, file) => {
    file.on('data', (data) => { csvBuffer = Buffer.concat([csvBuffer, data]); });
  });
  
  bb.on('field', (name, value) => { metadata[name] = value; });
  
  bb.on('finish', async () => {
    if (csvBuffer.length > 10 * 1024 * 1024) {
      res.writeHead(400); res.end(JSON.stringify({ error: 'File too large' })); return;
    }
    
    // Write upload to source_submissions
    await db.query(
      `INSERT INTO source_submissions 
       (job_id, submitter_id, tournament_name, gotsport_event_id, date_start, date_end, csv_data, status, submitted_at)
       VALUES ($1, $2, $3, $4, $5, $6, $7, 'queued', NOW())`,
      [jobId, user.rows[0].submitter_id, metadata.tournament_name, 
       metadata.event_id || null, metadata.date_start, metadata.date_end, csvBuffer.toString('utf8')]
    );
    
    // Queue async processing (use setImmediate or a simple queue — no worker infrastructure needed for MVP)
    setImmediate(() => processExportJob(jobId));
    
    res.writeHead(202); 
    res.end(JSON.stringify({ 
      status: 'queued', 
      job_id: jobId,
      status_url: `/api/ingest/status/${jobId}`
    }));
  });
  
  req.pipe(bb);
  return;
}
```

---

## Component 2: Database Schema

### `source_submissions` Table

```sql
CREATE TABLE source_submissions (
  id                  SERIAL PRIMARY KEY,
  job_id              VARCHAR(64) UNIQUE NOT NULL,
  submitter_id        INTEGER REFERENCES tournament_directors(id),
  tournament_name     VARCHAR(255) NOT NULL,
  gotsport_event_id   INTEGER,                  -- NULL if not provided
  date_start          DATE NOT NULL,
  date_end            DATE NOT NULL,
  csv_data            TEXT NOT NULL,            -- raw CSV content
  status              VARCHAR(32) DEFAULT 'queued',  -- queued | processing | complete | failed
  games_ingested      INTEGER DEFAULT 0,
  teams_updated       INTEGER DEFAULT 0,
  provisional_teams   INTEGER DEFAULT 0,
  error_message       TEXT,
  submitted_at        TIMESTAMP NOT NULL,
  processed_at        TIMESTAMP
);

CREATE INDEX idx_source_submissions_job_id ON source_submissions (job_id);
CREATE INDEX idx_source_submissions_status ON source_submissions (status);
CREATE INDEX idx_source_submissions_submitter ON source_submissions (submitter_id);
```

### `tournament_directors` Table

```sql
CREATE TABLE tournament_directors (
  id           SERIAL PRIMARY KEY,
  name         VARCHAR(255) NOT NULL,
  email        VARCHAR(255) UNIQUE NOT NULL,
  api_key      VARCHAR(64) UNIQUE NOT NULL,  -- td_key_{uuid}
  organization VARCHAR(255),
  created_at   TIMESTAMP DEFAULT NOW()
);
```

---

## Component 3: Parser — `scripts/parse-event-export.js`

### Input / Output

```javascript
// Input:
{
  jobId: string,          // source_submissions.job_id
  csvData: string,        // raw CSV content
  uploadId: string,       // same as jobId — used in source_id
  gotSportEventId: integer | null  // from upload metadata
}

// Output:
{
  games: [...],           // array of game objects in Evo Draw schema
  stats: {
    rows_parsed: N,
    rows_skipped_non_final: N,
    rows_skipped_no_score: N,
    team_resolutions: { exact: N, alias: N, provisional: N },
    games_deduped: N      // games hidden because API version already exists
  }
}
```

### Field Mapping: GotSport Export → `games` Table

| Export Column | `games` column | Transform |
|---|---|---|
| `Match ID` | `source_id` | `gs_export_{uploadId}_{matchId}` |
| `Event ID` | `event_id` | Direct if present; else from upload metadata |
| `Match Date` | `game_date` | Parse `MM/DD/YYYY` → `YYYY-MM-DD` |
| `Home Team` | `home_team_id` | Name → ID resolution (3-step) |
| `Away Team` | `away_team_id` | Name → ID resolution (3-step) |
| `Home Score` | `home_score` | Parse integer; skip if null/empty |
| `Away Score` | `away_score` | Parse integer; skip if null/empty |
| `Home PKS` | — | If present + non-null: scores are tied, PKS discarded |
| `Away PKS` | — | (same as above) |
| `Division` | `division_name` | Store raw; parse for age/gender |
| `Division` | `age_group` | Parse using `backfill-age-gender.js` logic |
| `Division` | `gender` | Parse using `backfill-age-gender.js` logic |
| `Round` | `round` | Normalize: Pool Play, Semifinal, Final, etc. |
| `Status` | — | Filter: Final only; skip Pending/Cancelled/Forfeit |
| — | `source` | Hardcoded: `'gs_export'` |
| — | `source_priority` | Hardcoded: `3` |
| — | `is_hidden` | Initially `false`; dedup pass may set to `true` |

### Team Name Resolution

See `Reports/gotsport-export-format-2026-04-23.md` for full resolution logic. Three-step fallback: exact match → alias lookup → provisional creation.

### Dedup Integration

After parsing all rows, run the dedup check against existing API records:

```javascript
for (const game of parsedGames) {
  if (game.gotsport_event_id && game.match_id) {
    // Check if API-scraped version already exists
    const apiVersion = await db.query(
      `SELECT id FROM games 
       WHERE source = 'gotsport_api' 
       AND source_id = $1 
       AND is_hidden = false`,
      [`gs_api_${game.gotsport_event_id}_${game.match_id}`]
    );
    
    if (apiVersion.rowCount > 0) {
      // API version exists and is canonical — hide the export version
      game.is_hidden = true;
      game.duplicate_of = apiVersion.rows[0].id;
      stats.games_deduped++;
    }
  }
}
```

**Source priority:** Export games are priority 3. API games are priority 1. When both exist for the same match, the API version is canonical and the export version is hidden. This prevents double-counting when a tournament director uploads data for an event the scraper already covered.

---

## Component 4: Confirmation Screen Design

After a successful upload and processing, the tournament director should see:

### What to Display

```
Event: [tournament_name]
Successfully processed: [date_end]

─────────────────────────────────────────
  [games_ingested]        [teams_updated]
  Games Ingested          Teams Updated

  [provisional_teams]     [games_deduped]  
  New Teams Added         Already in Evo Draw
─────────────────────────────────────────

[View Rankings for this Event →]
  (links to /rankings?event_id={gotsport_event_id} or /rankings?event={tournament_name_slug})

─────────────────────────────────────────
  Want precision division placement for your next event?
  
  Evo Draw uses rankings from 1.65M games across every 
  youth soccer league in the country to place teams into 
  the right divisions — no sandbagging, no blowouts.
  
  [Get Division Recommendations →]   [Contact Presten →]
─────────────────────────────────────────
```

### Data Required by Confirmation Screen

| Element | Data Source |
|---|---|
| `games_ingested` | `source_submissions.games_ingested` |
| `teams_updated` | `source_submissions.teams_updated` |
| `provisional_teams` | `source_submissions.provisional_teams` |
| `games_deduped` | `source_submissions.games_deduped` (add this column) |
| Rankings link | `gotsport_event_id` from submission or `tournament_name` slug |

### Strategic Importance

This screen is the relationship moment. A tournament director who uploads data and sees their event's teams appearing in national youth soccer rankings has an immediate, tangible reason to come back. The DSS pitch on the screen converts a data contributor into a potential paid customer in a single step. The confirmation screen should be polished even in MVP.

---

## Implementation Estimate (Validated)

| Task | Estimate | Notes |
|---|---|---|
| `source_submissions` + `tournament_directors` SQL tables | 0.5 days | Simple schema, no migrations complexity |
| Upload endpoint (`POST /api/ingest/event-export`) | 1 day | Auth, multipart parsing, validation, async queue |
| Status polling endpoint (`GET /api/ingest/status/:jobId`) | 0.5 days | Simple db query + JSON response |
| `parse-event-export.js` | 2 days | Team name resolution is the hard part; rest is straightforward |
| Dedup integration in parser | 0.5 days | Reuses existing dedup logic patterns |
| Tournament director web form | 1.5 days | File upload form, metadata fields, status polling UI |
| Confirmation screen | 0.5 days | Static HTML + JSON data from status endpoint |
| End-to-end testing with real GotSport export | 1 day | Need a real export file; edge cases in team name resolution |
| **Total** | **7.5 days** | Consistent with original estimate |

**Adjusted estimate note:** The original 7-day estimate was correct in aggregate. The team name resolution step is more complex than a naive CSV parse (2 days vs an initial mental model of 1 day), but the dedup integration is simpler than expected (0.5 days) because the existing dedup logic handles the pattern already. Net: 7.5 days vs 7.

### Complexity Adjustments from Format Research

The GotSport export does NOT include internal team IDs — only team name strings. This was the key risk from the format research. Impact: the team name → ID resolution step is the highest-complexity component. Mitigations:
1. Existing `team_merges` alias table handles most known name variants
2. Provisional team creation ensures no row is dropped due to name mismatch
3. Provisional teams are flagged in the confirmation screen for awareness

The parser does NOT need to handle ambiguous merges at ingestion time. If a team name matches multiple `team_merges` entries, use the highest-game-count match. Flag it in the stats as `resolution: ambiguous_used_best` for post-processing review.

---

## MVP Scope

**MVP = table + endpoint + parser + web form.** No email system, no admin UI. Presten processes the `source_submissions` queue manually via `psql` to issue API keys to tournament directors. Auto-routing and admin UI are Phase 2.

**Target:** Shipped and tested before DSS (May 16, 2026). Even if no tournament directors use it at DSS, having the endpoint live means it can be mentioned in the DSS post-event outreach as a feature available to future event partners.

---

## References

- [[Webhook Fallback Scope]] — the scoping document that established this pipeline as the primary recommendation
- [[Dedup Strategy]] — source priority and dedup logic
- [[Data Sources Overview]] — source priority table (export = priority 3, same slot as TGS)
- [[GotSport HTML Scraper]] — existing HTML parsing logic that shares patterns with the export parser
- [[Strategic Insights]] — Risk 1 (GotSport API dependency), Action Item 10 (GotSport partnership), Insight 11 (tournament director relationship)
- `Reports/gotsport-export-format-2026-04-23.md` — companion document with full field-level format documentation

---

*FORGE — 2026-04-23*
