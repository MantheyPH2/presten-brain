---
title: Source Submission MVP — Implementation Plan
tags: [product, pipeline, implementation-plan, mvp, evo-draw]
created: 2026-04-23
updated: 2026-04-23
status: ready-to-execute
author: FORGE
estimated_hours: 4–6
target_completion: Before DSS (May 16, 2026)
---

# Source Submission MVP — Implementation Plan

**Designed by:** FORGE  
**Date:** 2026-04-23  
**Design doc:** [[Source Submission Framework Design]]  
**Goal:** Zero ambiguity. Execute this plan in a single 4–6 hour session. All decisions are locked.

---

## Pre-Flight Checklist

Before writing a single line of code:

- [ ] **PostgreSQL access:** `psql -U p -d youth_soccer` connects without error
- [ ] **Server is running:** `curl http://localhost:3000/api/managed-events/dss2026` returns a 200 (or known 404) — server is up
- [ ] **Existing endpoints work:** Verify GET `/api/h2h-violations` returns JSON (not a server crash)
- [ ] **source_submissions table does NOT exist yet:** `psql -U p -d youth_soccer -c "\d source_submissions"` should return "Did not find any relation named..."
- [ ] **server.js is editable:** Confirm you know the path to `server.js` and it's not in a read-only location
- [ ] **No active pipeline run:** Check that no scraper is currently writing to the DB (check cron log or ps aux)

---

## Step 1 — Create the Database Table

**Time estimate: 5–10 minutes**

Run this against the `youth_soccer` database:

```sql
-- Run in psql or via: psql -U p -d youth_soccer < create-source-submissions.sql

CREATE TABLE source_submissions (
  id                SERIAL PRIMARY KEY,
  submitted_by_name TEXT,
  submitted_by_email TEXT,
  url               TEXT NOT NULL,
  source_type       TEXT,
  event_name        TEXT,
  age_groups        TEXT,
  date_range_start  DATE,
  date_range_end    DATE,
  notes             TEXT,
  status            TEXT NOT NULL DEFAULT 'pending',
  rejection_reason  TEXT,
  scraper_script    TEXT,
  games_imported    INTEGER,
  created_at        TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  processed_at      TIMESTAMPTZ,
  processed_by      TEXT
);

CREATE INDEX idx_submissions_status ON source_submissions (status, created_at);
CREATE INDEX idx_submissions_url ON source_submissions (url);
```

**Verify:**
```sql
\d source_submissions
-- Should show all columns listed above
```

Expected output: table description with 17 columns. If you see "relation does not exist" error on the CREATE, check you're connected to `youth_soccer` (`\c youth_soccer`).

---

## Step 2 — Add `source-detector.js` Module

**Time estimate: 15–20 minutes**

Create a new file `source-detector.js` in the same directory as `server.js`. Paste the following exactly:

```javascript
// source-detector.js
// Maps a submitted URL to its source family for scraper routing.

const SOURCE_PATTERNS = [
  { pattern: /system\.gotsport\.com\/api/i, family: 'gotsport-api',  scraper: 'crawl-gotsport-api-parallel.js', priority: 1 },
  { pattern: /gotsport\.com/i,              family: 'gotsport',       scraper: 'crawl-gotsport-html.js',         priority: 2 },
  { pattern: /soccer\.sincsports\.com/i,    family: 'sincsports',     scraper: 'crawl-sincsports.js',            priority: 5 },
  { pattern: /sincsports\.com/i,            family: 'sincsports',     scraper: 'crawl-sincsports.js',            priority: 5 },
  { pattern: /snapsoccer\.com/i,            family: 'snapsoccer',     scraper: 'crawl-snapsoccer.js',            priority: 5 },
  { pattern: /affinitysoccer\.com/i,        family: 'affinity',       scraper: null,                             priority: 6 },
  { pattern: /gotsoccer\.com/i,             family: 'gotsoccer',      scraper: null,                             priority: 6 },
  { pattern: /tourneymachine\.com/i,        family: 'tourneymachine', scraper: null,                             priority: 6 },
  { pattern: /usarank\.com/i,               family: 'usarank',        scraper: null,                             priority: null },
];

function detectSource(url) {
  try {
    const normalized = new URL(url).href.toLowerCase();
    for (const { pattern, family, scraper, priority } of SOURCE_PATTERNS) {
      if (pattern.test(normalized)) {
        return { family, scraper, priority, autoRoutable: scraper !== null };
      }
    }
    return { family: 'unknown', scraper: null, priority: null, autoRoutable: false };
  } catch {
    return { family: 'invalid-url', scraper: null, priority: null, autoRoutable: false };
  }
}

module.exports = { detectSource };
```

**Verify (Node.js REPL):**
```javascript
const { detectSource } = require('./source-detector');
detectSource('https://gotsport.com/games/schedule?id=12345');
// Expected: { family: 'gotsport', scraper: 'crawl-gotsport-html.js', priority: 2, autoRoutable: true }

detectSource('https://usarank.com/teamlist/123/2026/U13G');
// Expected: { family: 'usarank', scraper: null, priority: null, autoRoutable: false }

detectSource('not-a-url');
// Expected: { family: 'invalid-url', scraper: null, priority: null, autoRoutable: false }
```

---

## Step 3 — Add `POST /api/submit-source` to `server.js`

**Time estimate: 45–60 minutes**

Open `server.js`. At the top (with other requires), add:
```javascript
const { detectSource } = require('./source-detector');
```

Then find the request routing block (the `if/else if` chain that matches `req.method` and `pathname`). Add the following block **before** the final `else` (the 404 handler):

```javascript
// ── Source Submission ─────────────────────────────────────────
if (req.method === 'POST' && pathname === '/api/submit-source') {
  let body = '';
  req.on('data', chunk => { body += chunk; });
  req.on('end', async () => {
    try {
      const data = JSON.parse(body);

      // Validate: url is required and must be a valid URL
      if (!data.url) {
        return sendJSON(res, 400, { success: false, error: 'url is required.', field: 'url' });
      }
      try { new URL(data.url); } catch {
        return sendJSON(res, 400, { success: false, error: 'Invalid URL format.', field: 'url' });
      }

      // Reject competitor URLs
      if (/usarank\.com/i.test(data.url)) {
        return sendJSON(res, 400, { success: false, error: 'That URL cannot be submitted.', field: 'url' });
      }

      // Check for duplicate already-imported URL
      const existing = await db.query(
        'SELECT id, status, games_imported FROM source_submissions WHERE url = $1 LIMIT 1',
        [data.url]
      );
      if (existing.rows.length > 0 && existing.rows[0].status === 'imported') {
        return sendJSON(res, 200, {
          success: true,
          submission_id: existing.rows[0].id,
          status: 'imported',
          message: `Good news — this event is already in our database! We imported ${existing.rows[0].games_imported} games from it.`
        });
      }

      // Detect source family
      const detection = detectSource(data.url);

      // Insert submission
      const result = await db.query(`
        INSERT INTO source_submissions
          (submitted_by_name, submitted_by_email, url, source_type, event_name,
           age_groups, date_range_start, date_range_end, notes, status)
        VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, 'pending')
        RETURNING id
      `, [
        data.name        || null,
        data.email       || null,
        data.url,
        detection.family,
        data.event_name  || null,
        data.age_groups  || null,
        data.date_range_start || null,
        data.date_range_end   || null,
        data.notes       || null
      ]);

      const submissionId = result.rows[0].id;

      const message = detection.autoRoutable
        ? `Thanks! We detected this as a ${detection.family} event and will import results within 24 hours.`
        : `Received! We'll review this source manually within 2–3 business days.`;

      return sendJSON(res, 202, {
        success: true,
        submission_id: submissionId,
        status: 'pending',
        source_family: detection.family,
        message
      });

    } catch (err) {
      console.error('[submit-source]', err);
      return sendJSON(res, 500, { success: false, error: 'Server error. Please try again.' });
    }
  });
}

// ── Submissions Admin (no auth — MVP only) ────────────────────
else if (req.method === 'GET' && pathname === '/api/submissions') {
  try {
    const rows = await db.query(
      `SELECT id, submitted_by_name, submitted_by_email, url, source_type,
              event_name, status, games_imported, created_at, processed_at
       FROM source_submissions
       ORDER BY created_at DESC
       LIMIT 100`
    );
    return sendJSON(res, 200, { success: true, submissions: rows.rows });
  } catch (err) {
    console.error('[submissions-admin]', err);
    return sendJSON(res, 500, { success: false, error: 'Server error.' });
  }
}
```

> **Security note:** `GET /api/submissions` has no authentication in MVP. This is a known gap — it exposes submitter emails to anyone who knows the URL. Add auth in Phase 2 before publicly advertising the feature. For now, the URL is obscure enough and the data is low-sensitivity.

**Verify (restart server first: `node server.js`):**

```bash
# Test valid GotSport submission
curl -X POST http://localhost:3000/api/submit-source \
  -H "Content-Type: application/json" \
  -d '{"url":"https://gotsport.com/games/schedule?id=99999","name":"Test User","email":"test@test.com","event_name":"Test Event"}'
# Expected: 202 with source_family: "gotsport", submission_id: 1

# Test invalid URL
curl -X POST http://localhost:3000/api/submit-source \
  -H "Content-Type: application/json" \
  -d '{"url":"not-a-url"}'
# Expected: 400 with error: "Invalid URL format."

# Test admin endpoint
curl http://localhost:3000/api/submissions
# Expected: 200 with submissions array containing the row you just inserted
```

Also verify in psql:
```sql
SELECT id, url, source_type, status, created_at FROM source_submissions;
-- Should show 1 row with status='pending', source_type='gotsport'
```

---

## Step 4 — Build the Web Form

**Time estimate: 45–60 minutes**

Create the submission form. The cleanest approach depends on the frontend:

**If the frontend is React (frontend-v2/):** Create `frontend-v2/src/pages/SubmitSource.jsx` and add a route in the router. See code below.

**If you want it live immediately without a rebuild:** Create a static HTML file at `public/submit-source.html` (or wherever the server serves static files) that posts directly to the API.

### Static HTML Option (fastest to ship):

Save as `public/submit-source.html` (adjust path to match where server.js serves static files):

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Submit a Source — Evo Draw</title>
  <style>
    body { font-family: system-ui, sans-serif; max-width: 600px; margin: 60px auto; padding: 0 20px; color: #1a1a1a; }
    h1 { background: linear-gradient(to right, #34d399, #06b6d4); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
    label { display: block; margin-top: 16px; font-weight: 600; font-size: 14px; }
    input, textarea { width: 100%; margin-top: 6px; padding: 10px; border: 1px solid #d1d5db; border-radius: 6px; font-size: 14px; box-sizing: border-box; }
    button { margin-top: 24px; width: 100%; padding: 12px; background: linear-gradient(to right, #34d399, #06b6d4); color: white; border: none; border-radius: 6px; font-size: 16px; font-weight: 600; cursor: pointer; }
    button:disabled { opacity: 0.6; cursor: not-allowed; }
    #submit-result { margin-top: 20px; padding: 14px; border-radius: 6px; display: none; }
    #submit-result.success { background: #d1fae5; color: #065f46; }
    #submit-result.error   { background: #fee2e2; color: #991b1b; }
    p.subtitle { color: #6b7280; margin-top: 8px; }
  </style>
</head>
<body>
  <h1>Submit a Tournament Source</h1>
  <p class="subtitle">Know of game results we're missing? Share the link — we'll import them and update our rankings.</p>

  <form id="submit-form">
    <label for="url">Results URL <span style="color:#ef4444">*</span></label>
    <input type="url" id="url" name="url" placeholder="https://gotsport.com/games/schedule?id=..." required>

    <label for="event_name">Event / Tournament Name</label>
    <input type="text" id="event_name" name="event_name" placeholder="Spring Classic 2026">

    <label for="age_groups">Age Groups</label>
    <input type="text" id="age_groups" name="age_groups" placeholder="U13 Girls, U14 Boys">

    <label for="name">Your Name</label>
    <input type="text" id="name" name="name" placeholder="Jane Smith">

    <label for="email">Your Email (optional — for confirmation)</label>
    <input type="email" id="email" name="email" placeholder="jane@fcstars.com">

    <label for="notes">Notes</label>
    <textarea id="notes" name="notes" rows="3" placeholder="Anything we should know about this event..."></textarea>

    <button type="submit" id="submit-btn">Submit Source</button>
  </form>

  <div id="submit-result"></div>

  <script>
    document.getElementById('submit-form').addEventListener('submit', async (e) => {
      e.preventDefault();
      const btn = document.getElementById('submit-btn');
      const resultDiv = document.getElementById('submit-result');
      btn.disabled = true;
      btn.textContent = 'Submitting...';
      resultDiv.style.display = 'none';

      const payload = {
        url:        document.getElementById('url').value.trim(),
        event_name: document.getElementById('event_name').value.trim() || undefined,
        age_groups: document.getElementById('age_groups').value.trim() || undefined,
        name:       document.getElementById('name').value.trim() || undefined,
        email:      document.getElementById('email').value.trim() || undefined,
        notes:      document.getElementById('notes').value.trim() || undefined,
      };

      try {
        const response = await fetch('/api/submit-source', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload),
        });
        const data = await response.json();

        resultDiv.style.display = 'block';
        if (data.success) {
          resultDiv.className = 'success';
          resultDiv.textContent = data.message;
          document.getElementById('submit-form').reset();
        } else {
          resultDiv.className = 'error';
          resultDiv.textContent = data.error || 'Something went wrong. Please try again.';
        }
      } catch (err) {
        resultDiv.style.display = 'block';
        resultDiv.className = 'error';
        resultDiv.textContent = 'Network error. Please check your connection and try again.';
      } finally {
        btn.disabled = false;
        btn.textContent = 'Submit Source';
      }
    });
  </script>
</body>
</html>
```

**Verify:**
- Open `http://localhost:3000/submit-source.html` (or the appropriate path)
- Submit the form with a valid GotSport URL
- Confirm: success message appears, row appears in `source_submissions` with `status='pending'`
- Submit again with the same URL → should show the duplicate message (or a second pending row if not yet imported)
- Submit with an invalid URL → should show the error message (from client-side `required` + `type=url`, then server-side if bypassed)

---

## Step 5 — Full Flow Test

**Time estimate: 15–20 minutes**

Run through this complete checklist:

1. **Happy path:** Submit a valid GotSport URL via the form → 202 success message → row in DB
2. **Duplicate imported:** Manually update a row to `status='imported', games_imported=50`, then submit the same URL again → should receive the "already imported" message
3. **Invalid URL:** Submit `not-a-url` → 400 error shown in form
4. **SincSports URL:** Submit `https://soccer.sincsports.com/schedule.aspx?tid=TEST` → source_family = 'sincsports' in response
5. **Competitor URL:** Submit `https://usarank.com/teamlist/123` → 400 rejection
6. **Admin endpoint:** `curl http://localhost:3000/api/submissions` → returns all rows as JSON
7. **Existing endpoints still work:** `curl http://localhost:3000/api/h2h-violations` → normal response (not a 500)

---

## Rollback Plan

If something breaks:

**Drop the table (if DDL was wrong):**
```sql
DROP TABLE IF EXISTS source_submissions;
```

**Remove the new routes from server.js:**
Find and delete the two blocks added in Step 3 (from `// ── Source Submission ──` to the closing `}` of the admin route). The file should revert to exactly its state before Step 3.

**Verify existing endpoints:**
```bash
curl http://localhost:3000/api/managed-events/dss2026
curl http://localhost:3000/api/h2h-violations
```
Both should return their normal responses.

**source-detector.js:** If it causes a require error, delete the file and remove the `require('./source-detector')` line from server.js top. The server will start without it.

---

## Post-Launch Verification (Definition of Done)

Run these checks before calling it shipped:

- [ ] `psql -U p -d youth_soccer -c "SELECT COUNT(*) FROM source_submissions WHERE status='pending'"` returns at least 1 (your test submission)
- [ ] `GET http://localhost:3000/api/submissions` returns that row with correct `source_type`
- [ ] `POST /api/submit-source` with GotSport URL returns 202 and `source_family: "gotsport"`
- [ ] The web form at `/submit-source.html` shows a success message after submission
- [ ] All 5 original endpoints still return correct data (not 500)
- [ ] No `console.error` lines appear in the server log for any of the above tests

---

## What Is Explicitly Out of Scope (MVP)

Do NOT build these now:

| Feature | Phase |
|---------|-------|
| Email confirmation to submitter | Phase 2 |
| Auto-routing to existing scrapers | Phase 2 |
| Admin UI for reviewing submissions | Phase 2 (Presten uses `psql` directly for now) |
| Authentication on `/api/submissions` | Phase 2 |
| Community credit system | Phase 3 |
| Automated triage scoring | Phase 2 |

**Presten's MVP workflow for processing submissions:**
```sql
-- Morning review: what came in?
SELECT id, submitted_by_email, url, source_type, event_name, created_at
FROM source_submissions
WHERE status = 'pending'
ORDER BY created_at DESC;

-- After running a scraper manually:
UPDATE source_submissions
SET status = 'imported', games_imported = 143, processed_at = NOW(), processed_by = 'presten'
WHERE id = 5;

-- Reject a submission:
UPDATE source_submissions
SET status = 'rejected', rejection_reason = 'Already captured via GotSport API', processed_at = NOW(), processed_by = 'presten'
WHERE id = 6;
```

---

## Timeline

| Step | Estimated Time |
|------|---------------|
| Pre-flight checklist | 10 min |
| Step 1: Create DB table | 10 min |
| Step 2: source-detector.js | 20 min |
| Step 3: API endpoints | 60 min |
| Step 4: Web form | 60 min |
| Step 5: Full flow test | 20 min |
| Buffer / debugging | 30 min |
| **Total** | **~3.5–4 hours** |

---

## Related Notes
- [[Source Submission Framework Design]] — the design doc this implements
- [[Server and Frontend]] — server.js structure and existing endpoints
- [[Database Schema]] — `youth_soccer` DB connection details
- [[Data Sources Overview]] — source priority system context

---

*Implementation plan by FORGE — 2026-04-23*
