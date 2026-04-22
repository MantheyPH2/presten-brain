---
title: Source Submission Framework Design
tags: [product, pipeline, data-sources, competitive-response, evo-draw]
created: 2026-04-23
updated: 2026-04-23
status: design-complete
priority: high
---

# Source Submission Framework Design

**Designed by:** FORGE  
**Date:** 2026-04-23  
**Context:** Competitive response to USA Rank's 400+ source community-sourced data moat.  
**Goal:** Let tournament directors and users bring events to us — we don't need to build 400 scrapers.

---

## 1. User Flow

```
Submitter (tournament director or parent/coach)
  │
  ▼
POST /api/submit-source
  [web form on evo draw site]
  │
  ▼
submissions table (status: 'pending')
  │
  ▼
Automated triage (source detection classifies URL)
  │
  ├─── Known source (GotSport, SincSports, etc.)
  │       ▼
  │    status: 'queued' → routed to existing scraper
  │       ▼
  │    Scraper runs, games inserted → status: 'imported'
  │       ▼
  │    Email confirmation sent to submitter
  │
  └─── Unknown source
          ▼
       status: 'manual-review' (Presten reviews)
          ▼
       If buildable: new scraper built → status: 'imported'
       If not: status: 'rejected' + reason logged
```

---

## 2. Database Schema — Submissions Table

```sql
-- Run against youth_soccer database
CREATE TABLE source_submissions (
  id                SERIAL PRIMARY KEY,
  submitted_by_name TEXT,
  submitted_by_email TEXT,
  url               TEXT NOT NULL,
  source_type       TEXT,            -- auto-detected: 'gotsport', 'sincsports', 'snapsoccer', 'unknown', etc.
  event_name        TEXT,            -- submitter-provided event name (optional)
  age_groups        TEXT,            -- submitter-provided: e.g. "U13, U14 Boys"
  date_range_start  DATE,
  date_range_end    DATE,
  notes             TEXT,            -- free-text from submitter
  status            TEXT NOT NULL DEFAULT 'pending',
                                     -- pending | queued | scraping | imported | manual-review | rejected
  rejection_reason  TEXT,
  scraper_script    TEXT,            -- e.g. 'crawl-gotsport-html.js'
  games_imported    INTEGER,         -- populated after scrape completes
  created_at        TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  processed_at      TIMESTAMPTZ,
  processed_by      TEXT            -- 'auto' or 'presten'
);

-- Index for status queue processing
CREATE INDEX idx_submissions_status ON source_submissions (status, created_at);

-- Index for duplicate URL detection
CREATE INDEX idx_submissions_url ON source_submissions (url);
```

**Status lifecycle:**
```
pending → queued      (auto-triage identified a known scraper)
pending → manual-review (unknown source, Presten reviews)
queued  → scraping    (scraper job started)
scraping → imported   (scraper completed, games in DB)
scraping → rejected   (scraper failed, unrecoverable)
manual-review → queued   (Presten approves, assigns scraper)
manual-review → rejected (Presten declines, logs reason)
```

---

## 3. Source Detection Logic

```javascript
// source-detector.js
// Maps submitted URL → source family for scraper routing

const SOURCE_PATTERNS = [
  { pattern: /gotsport\.com/i,           family: 'gotsport',      scraper: 'crawl-gotsport-html.js',   priority: 2 },
  { pattern: /system\.gotsport\.com\/api/i, family: 'gotsport-api', scraper: 'crawl-gotsport-api-parallel.js', priority: 1 },
  { pattern: /soccer\.sincsports\.com/i, family: 'sincsports',    scraper: 'crawl-sincsports.js',      priority: 5 },
  { pattern: /sincsports\.com/i,         family: 'sincsports',    scraper: 'crawl-sincsports.js',      priority: 5 },
  { pattern: /snapsoccer\.com/i,         family: 'snapsoccer',    scraper: 'crawl-snapsoccer.js',      priority: 5 },
  { pattern: /affinitysoccer\.com/i,     family: 'affinity',      scraper: null,                       priority: 6 },
  { pattern: /gotsoccer\.com/i,          family: 'gotsoccer',     scraper: null,                       priority: 6 },
  { pattern: /tourneymachine\.com/i,     family: 'tourneymachine',scraper: null,                       priority: 6 },
  { pattern: /usarank\.com/i,            family: 'usarank',       scraper: null,                       priority: null }, // competitor, reject
];

/**
 * Detects source family from a submitted URL.
 * @param {string} url - The submitted event/results URL
 * @returns {{ family: string, scraper: string|null, priority: number|null, autoRoutable: boolean }}
 */
function detectSource(url) {
  try {
    const normalized = new URL(url).href.toLowerCase();
    for (const { pattern, family, scraper, priority } of SOURCE_PATTERNS) {
      if (pattern.test(normalized)) {
        return {
          family,
          scraper,
          priority,
          autoRoutable: scraper !== null,
        };
      }
    }
    return { family: 'unknown', scraper: null, priority: null, autoRoutable: false };
  } catch {
    return { family: 'invalid-url', scraper: null, priority: null, autoRoutable: false };
  }
}

module.exports = { detectSource };
```

---

## 4. Scraper Routing Table

| Source Family | Detected From | Scraper Script | Priority | Auto-Routable |
|---------------|---------------|----------------|----------|---------------|
| `gotsport` | `gotsport.com` in URL | `crawl-gotsport-html.js` | 2 | Yes |
| `gotsport-api` | `system.gotsport.com/api` | `crawl-gotsport-api-parallel.js` | 1 | Yes |
| `sincsports` | `sincsports.com` in URL | `crawl-sincsports.js` | 5 | Yes (once scraper built) |
| `snapsoccer` | `snapsoccer.com` in URL | `crawl-snapsoccer.js` | 5 | Yes (once scraper built) |
| `affinity` | `affinitysoccer.com` | TBD | 6 | No — manual review |
| `gotsoccer` | `gotsoccer.com` | TBD | 6 | No — manual review |
| `tourneymachine` | `tourneymachine.com` | TBD | 6 | No — manual review |
| `unknown` | No pattern match | None | None | No — manual review |
| `usarank` | `usarank.com` | Rejected | None | No — reject with message |

**Routing logic:**
```javascript
// route-submission.js
const { detectSource } = require('./source-detector');

async function routeSubmission(submissionId, url) {
  const detection = detectSource(url);

  if (detection.family === 'usarank') {
    await updateStatus(submissionId, 'rejected', 'URL points to a competitor site.');
    return;
  }

  if (detection.autoRoutable) {
    await updateStatus(submissionId, 'queued');
    await setScraperScript(submissionId, detection.scraper, detection.family);
    await enqueueScrapeJob(submissionId, detection.scraper, url);
  } else {
    await updateStatus(submissionId, 'manual-review');
    await notifyPresten(submissionId, url, detection.family);
  }
}
```

---

## 5. API Endpoint Design

### `POST /api/submit-source`

**Purpose:** Accept a source submission from a tournament director or user.

**Request Body:**
```json
{
  "name": "Jane Smith",
  "email": "jane@fcstars.com",
  "url": "https://soccer.sincsports.com/schedule.aspx?tid=COASTALINV",
  "event_name": "Coastal Soccer Invitational 2026",
  "age_groups": "U13, U14 Boys",
  "date_range_start": "2026-04-11",
  "date_range_end": "2026-04-13",
  "notes": "We run this every spring, results should be here"
}
```

**Validation Rules:**
- `url` — required, must be a valid URL (`new URL(url)` must not throw), max 1000 chars
- `email` — optional but must be valid format if provided (for confirmation emails)
- `name` — optional, max 200 chars
- `event_name` — optional, max 500 chars
- `url` must not already exist in `source_submissions` with status `imported` (duplicate check)
- `url` must not point to `usarank.com` (competitor rejection)

**Success Response (202 Accepted):**
```json
{
  "success": true,
  "submission_id": 42,
  "status": "pending",
  "message": "Thanks! We detected this as a SincSports event and will import results within 24 hours.",
  "source_family": "sincsports"
}
```

**Error Response (400 Bad Request):**
```json
{
  "success": false,
  "error": "Invalid URL format.",
  "field": "url"
}
```

**Duplicate Response (200 OK):**
```json
{
  "success": true,
  "submission_id": 17,
  "status": "imported",
  "message": "Good news — this event is already in our database! We imported 143 games from it."
}
```

**Server.js handler stub (plain Node.js, no Express):**
```javascript
// In server.js — add to request routing block
if (req.method === 'POST' && pathname === '/api/submit-source') {
  let body = '';
  req.on('data', chunk => body += chunk);
  req.on('end', async () => {
    try {
      const data = JSON.parse(body);

      // Validate
      if (!data.url) return sendJSON(res, 400, { success: false, error: 'url is required.' });
      try { new URL(data.url); } catch {
        return sendJSON(res, 400, { success: false, error: 'Invalid URL format.', field: 'url' });
      }

      // Check duplicate
      const existing = await db.query(
        'SELECT id, status, games_imported FROM source_submissions WHERE url = $1 LIMIT 1',
        [data.url]
      );
      if (existing.rows.length > 0 && existing.rows[0].status === 'imported') {
        return sendJSON(res, 200, {
          success: true,
          submission_id: existing.rows[0].id,
          status: 'imported',
          message: `This event is already in our database — ${existing.rows[0].games_imported} games imported.`
        });
      }

      // Detect source
      const detection = detectSource(data.url);

      // Insert submission
      const result = await db.query(`
        INSERT INTO source_submissions
          (submitted_by_name, submitted_by_email, url, source_type, event_name,
           age_groups, date_range_start, date_range_end, notes, status)
        VALUES ($1,$2,$3,$4,$5,$6,$7,$8,$9,'pending')
        RETURNING id
      `, [data.name, data.email, data.url, detection.family, data.event_name,
          data.age_groups, data.date_range_start || null, data.date_range_end || null, data.notes]);

      const submissionId = result.rows[0].id;

      // Route async (don't await — return immediately)
      routeSubmission(submissionId, data.url).catch(console.error);

      const message = detection.autoRoutable
        ? `We detected this as a ${detection.family} event and will import results within 24 hours.`
        : `Received! We'll review this source manually within 2–3 business days.`;

      return sendJSON(res, 202, {
        success: true,
        submission_id: submissionId,
        status: 'pending',
        source_family: detection.family,
        message
      });

    } catch (err) {
      console.error('submit-source error:', err);
      return sendJSON(res, 500, { success: false, error: 'Server error.' });
    }
  });
}
```

---

## 6. Priority and Effort Estimates

| Component | Effort | Notes |
|-----------|--------|-------|
| `source_submissions` table (SQL) | **Low** — 30 min | Run `CREATE TABLE` script against youth_soccer DB |
| `detectSource()` function | **Low** — 1–2 hours | Pure JS, no DB required |
| `POST /api/submit-source` endpoint | **Medium** — 3–4 hours | Add to server.js, wire validation + DB insert |
| `routeSubmission()` routing logic | **Medium** — 2–3 hours | Status updates, async job enqueue |
| Email confirmation to submitter | **Medium** — 2–3 hours | Nodemailer or simple SMTP; skip for MVP |
| Admin view for `manual-review` queue | **Medium** — 4–6 hours | Simple HTML table at `/admin/submissions`; skip for MVP |
| Duplicate URL detection | **Low** — 1 hour | One `SELECT` before insert |
| GotSport HTML auto-routing | **Low** — 1–2 hours | Wire existing scraper to accept URL param |

**Total estimate:** 14–21 hours full build. MVP: 6–8 hours.

---

## 7. MVP Scope — Start Accepting Submissions Before DSS

**MVP = What Presten can build in one focused session (4–6 hours) to open submissions before Dallas Spring Showcase (May 16, 2026).**

### MVP Includes:
1. **`CREATE TABLE source_submissions`** — run the SQL above. Done in 5 minutes.
2. **`POST /api/submit-source` endpoint** — validation, duplicate check, DB insert, `detectSource()` classification. Returns 202 with detected source family. No email, no auto-scraping yet.
3. **Simple web form** — one page on the Evo Draw site: URL field, event name, email, submit button. POST to the endpoint. Show success/error message.
4. **`detectSource()` function** — classify submitted URLs. Pure JS, self-contained.

### MVP Excludes (Phase 2):
- Email confirmation to submitter
- Automatic scraper routing (Presten manually processes the queue)
- Admin UI (Presten queries `source_submissions` directly via `psql`)
- Community credit system

### MVP Presten Workflow (no admin UI needed):
```sql
-- Check pending submissions each morning
SELECT id, submitted_by_email, url, source_type, event_name, created_at
FROM source_submissions
WHERE status = 'pending'
ORDER BY created_at DESC;

-- After manually running a scraper:
UPDATE source_submissions
SET status = 'imported', games_imported = 143, processed_at = NOW(), processed_by = 'presten'
WHERE id = 42;
```

### Timeline
| Milestone | Target |
|-----------|--------|
| `source_submissions` table created | Day 1 (30 min) |
| `/api/submit-source` endpoint live | Day 1 (3–4 hours) |
| Web form live on evo draw site | Day 1–2 (2 hours) |
| First tournament director submission | Within 1 week of launch |
| Auto-routing for GotSport URLs | Phase 2 (post-DSS) |

---

## Related Notes
- [[Data Sources Overview]] — current 7 sources
- [[SnapSoccer Source Research]] — first source to add via this framework
- [[Data Pipeline]] — Step 0 scraper intake
- [[Database Schema]] — `games` table that submissions ultimately feed
- [[Server and Frontend]] — Node.js server where endpoint lives

---

*Designed by FORGE — 2026-04-23*
