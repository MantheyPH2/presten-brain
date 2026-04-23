---
title: Event Strength Ratings — Implementation Plan
aliases: [Event Strength Implementation, Tournament Strength Implementation]
tags: [rankings, events, strength, implementation, evo-draw]
created: 2026-04-23
updated: 2026-04-23
status: ready-to-implement
author: ELO Agent
design_doc: "[[Event Strength Ratings Design]]"
start_by: 2026-04-28
demo_ready: 2026-05-09
dss_deadline: 2026-05-16
estimated_hours: 13
---

# Event Strength Ratings — Implementation Plan

> Design is approved. Four metrics: Average Team Rating (+ percentile), Strength Classification (High/Average/Low), Competitive Spread (Tight/Mixed/Wide), Upset Rate (Low/Average/High). This plan is executable starting April 28 to hit the May 9 demo-ready deadline. **Competitive differentiator: we show the numbers, not just labels.**

---

## Pre-Flight Checklist

Run before any implementation. ~30 minutes total.

**1. Confirm `events` table (or confirm games-only approach):**
```sql
-- Check if a standalone events table exists
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
  AND table_name = 'events';
```
- **If exists:** run `\d events` — confirm columns `id`, `event_name`, `age_group`, `gender`
- **If does not exist:** proceed with `games.event_name` as the composite event key throughout. The computation SQL uses `event_name` as the identifier and handles both cases — see the design doc's fallback approach.

**2. Confirm games table foreign key:**
```sql
SELECT COUNT(*) FROM games WHERE event_id IS NOT NULL AND is_hidden = false;
```
If > 0: `games.event_id` is populated. Use the FK-based approach.
If 0 or column doesn't exist: use `games.event_name` as the event identifier throughout.

**3. Confirm ratings are populated:**
```sql
SELECT COUNT(*) FROM rankings WHERE rating IS NOT NULL;
```
Expected: 30,000–50,000+ rows. If near 0, rankings haven't been computed yet — run `compute-rankings.js` first.

**4. Confirm no `ratings_history` table:**
```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
  AND table_name LIKE '%rating%history%';
```
Expected: no results. Current-rating approximation is the accepted approach (per design doc). Note this in the UI.

**5. Count events below the 4-team minimum:**
```sql
SELECT COUNT(*) AS tiny_events
FROM (
  SELECT event_name, age_group, gender
  FROM games
  WHERE is_hidden = false
  GROUP BY event_name, age_group, gender
  HAVING COUNT(DISTINCT home_team_id) + COUNT(DISTINCT away_team_id) < 4
) sub;
```
Report this number for the briefing — it sets expectations on how many events will have `strength_label = 'Insufficient Data'`.

---

## Phase 1 — Database (~2 hours)

### Step 1: Create the `event_strength` table

```sql
CREATE TABLE event_strength (
  id                    SERIAL PRIMARY KEY,
  event_name            VARCHAR(300) NOT NULL,
  age_group             VARCHAR(10) NOT NULL,
  gender                CHAR(1) NOT NULL CHECK (gender IN ('M', 'F', 'U')),
  season                VARCHAR(10),
  event_id              INT,    -- FK to events.id if that table exists; otherwise NULL

  computed_at           TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  team_count            INT NOT NULL,
  avg_team_rating       NUMERIC(8,2) NOT NULL,
  rating_spread         NUMERIC(8,2),
  upset_rate            NUMERIC(5,4),
  decisive_games        INT,
  upset_count           INT,
  min_rating            NUMERIC(8,2),
  max_rating            NUMERIC(8,2),

  strength_percentile   NUMERIC(5,2),

  strength_label        VARCHAR(20) CHECK (strength_label IN ('High','Average','Low','Insufficient Data')),
  spread_label          VARCHAR(20) CHECK (spread_label IN ('Tight','Mixed','Wide','Unknown')),
  upset_label           VARCHAR(20) CHECK (upset_label IN ('Low','Average','High','Unknown')),

  is_rated              BOOLEAN NOT NULL DEFAULT true,
  ratings_are_current   BOOLEAN NOT NULL DEFAULT true,

  UNIQUE (event_name, age_group, gender, season, computed_at)
);

CREATE INDEX idx_es_lookup    ON event_strength (event_name, age_group, gender, season);
CREATE INDEX idx_es_label     ON event_strength (strength_label, age_group, gender);
CREATE INDEX idx_es_computed  ON event_strength (computed_at);
```

### Step 2: Verify the table

```bash
psql -d youth_soccer -c "\d event_strength"
```

### Step 3: Run the base metrics query on a sample of 10 events

```sql
-- Spot check: avg_team_rating for 10 events
SELECT
  event_name,
  age_group,
  gender,
  COUNT(DISTINCT team_id) AS team_count,
  ROUND(AVG(rating), 2)   AS avg_team_rating
FROM (
  SELECT DISTINCT g.event_name, g.age_group, g.gender,
    CASE WHEN g.home_score IS NOT NULL THEN g.home_team_id ELSE g.away_team_id END AS team_id,
    r.rating
  FROM games g
  LEFT JOIN rankings r ON r.team_id = g.home_team_id
    AND r.age_group = g.age_group AND r.gender = g.gender
  WHERE g.is_hidden = false AND r.rating IS NOT NULL
  UNION ALL
  SELECT DISTINCT g.event_name, g.age_group, g.gender,
    g.away_team_id AS team_id,
    r.rating
  FROM games g
  LEFT JOIN rankings r ON r.team_id = g.away_team_id
    AND r.age_group = g.age_group AND r.gender = g.gender
  WHERE g.is_hidden = false AND r.rating IS NOT NULL
) t
GROUP BY event_name, age_group, gender
HAVING COUNT(DISTINCT team_id) >= 4
LIMIT 10;
```

Results should show avg ratings in the 900–1400 range and team counts of 4–100+.

### Step 4: Run the upset rate query on the same sample

```sql
-- Spot check: upset rate for a known large event
SELECT
  event_name,
  COUNT(*) AS total_decisive,
  SUM(CASE
    WHEN home_score > away_score AND home_rating < away_rating
         AND ABS(home_rating - away_rating) >= 50 THEN 1
    WHEN away_score > home_score AND away_rating < home_rating
         AND ABS(home_rating - away_rating) >= 50 THEN 1
    ELSE 0
  END) AS upsets,
  ROUND(
    SUM(CASE
      WHEN home_score > away_score AND home_rating < away_rating
           AND ABS(home_rating - away_rating) >= 50 THEN 1
      WHEN away_score > home_score AND away_rating < home_rating
           AND ABS(home_rating - away_rating) >= 50 THEN 1
      ELSE 0
    END)::numeric /
    NULLIF(COUNT(*), 0), 3
  ) AS upset_rate
FROM (
  SELECT
    g.event_name,
    g.home_score,
    g.away_score,
    rh.rating AS home_rating,
    ra.rating AS away_rating
  FROM games g
  JOIN rankings rh ON rh.team_id = g.home_team_id
    AND rh.age_group = g.age_group AND rh.gender = g.gender
  JOIN rankings ra ON ra.team_id = g.away_team_id
    AND ra.age_group = g.age_group AND ra.gender = g.gender
  WHERE g.is_hidden = false
    AND g.game_status = 'completed'
    AND g.home_score IS NOT NULL
    AND g.away_score IS NOT NULL
    AND g.home_score != g.away_score   -- decisive only
) t
GROUP BY event_name
HAVING COUNT(*) >= 3
LIMIT 10;
```

Upset rates should mostly fall between 0.15 and 0.45.

### Step 5: Confirm `PERCENT_RANK()` window function works

```sql
SELECT
  event_name,
  age_group,
  avg_team_rating,
  PERCENT_RANK() OVER (
    PARTITION BY age_group, gender
    ORDER BY avg_team_rating
  ) AS strength_percentile
FROM (
  SELECT event_name, age_group, gender, AVG(r.rating) AS avg_team_rating
  FROM games g
  JOIN rankings r ON r.team_id = g.home_team_id
    AND r.age_group = g.age_group AND r.gender = g.gender
  WHERE g.is_hidden = false AND g.season = '2025-2026'
  GROUP BY event_name, age_group, gender
  HAVING COUNT(DISTINCT g.home_team_id) >= 4
) t
ORDER BY age_group, gender, avg_team_rating DESC
LIMIT 20;
```

---

## Phase 2 — Computation Script (~5 hours)

### Step 6: Write `compute-event-strength.js`

Create `scripts/compute-event-strength.js`:

```javascript
#!/usr/bin/env node
// compute-event-strength.js
// Computes event strength metrics for all events in the current season.
// Run after compute-rankings.js in the nightly pipeline.

const { Pool } = require('pg');

const pool = new Pool({
  host: 'localhost',
  port: 5432,
  database: 'youth_soccer',
  user: 'p',
});

const SEASON = '2025-2026';  // update each season or pass as CLI arg

async function run() {
  const client = await pool.connect();
  try {
    console.log(`[event-strength] Computing event strength for season ${SEASON}...`);
    const start = Date.now();

    // Clear stale rows for this season's current run
    // We insert a new snapshot each run; previous rows are preserved for history.
    // To avoid bloat, optionally delete rows older than 30 days:
    await client.query(`
      DELETE FROM event_strength
      WHERE season = $1
        AND computed_at < NOW() - INTERVAL '30 days'
    `, [SEASON]);

    const result = await client.query(computeSQL, [SEASON]);
    console.log(`[event-strength] Inserted ${result.rowCount} rows in ${Date.now() - start}ms`);
  } finally {
    client.release();
    await pool.end();
  }
}

const computeSQL = `
INSERT INTO event_strength (
  event_name, age_group, gender, season,
  team_count, avg_team_rating, rating_spread, min_rating, max_rating,
  upset_rate, decisive_games, upset_count,
  strength_percentile,
  strength_label, spread_label, upset_label,
  is_rated, ratings_are_current, computed_at
)
WITH event_teams AS (
  SELECT DISTINCT g.event_name, g.age_group, g.gender, g.season, g.home_team_id AS team_id, r.rating
  FROM games g
  JOIN rankings r ON r.team_id = g.home_team_id
    AND r.age_group = g.age_group AND r.gender = g.gender
  WHERE g.is_hidden = false AND g.season = $1
  UNION
  SELECT DISTINCT g.event_name, g.age_group, g.gender, g.season, g.away_team_id AS team_id, r.rating
  FROM games g
  JOIN rankings r ON r.team_id = g.away_team_id
    AND r.age_group = g.age_group AND r.gender = g.gender
  WHERE g.is_hidden = false AND g.season = $1
),
event_base AS (
  SELECT
    event_name, age_group, gender, season,
    COUNT(*)               AS team_count,
    AVG(rating)            AS avg_team_rating,
    STDDEV(rating)         AS rating_spread,
    MIN(rating)            AS min_rating,
    MAX(rating)            AS max_rating
  FROM event_teams
  GROUP BY event_name, age_group, gender, season
  HAVING COUNT(*) >= 4
),
game_results AS (
  SELECT
    g.event_name, g.age_group, g.gender, g.season,
    ABS(rh.rating - ra.rating) AS rating_gap,
    CASE
      WHEN g.home_score > g.away_score THEN 'home_win'
      WHEN g.away_score > g.home_score THEN 'away_win'
      ELSE 'draw'
    END AS result,
    rh.rating AS home_rating,
    ra.rating AS away_rating
  FROM games g
  JOIN rankings rh ON rh.team_id = g.home_team_id AND rh.age_group = g.age_group AND rh.gender = g.gender
  JOIN rankings ra ON ra.team_id = g.away_team_id AND ra.age_group = g.age_group AND ra.gender = g.gender
  WHERE g.is_hidden = false
    AND g.game_status = 'completed'
    AND g.home_score IS NOT NULL AND g.away_score IS NOT NULL
    AND g.season = $1
),
event_upsets AS (
  SELECT
    event_name, age_group, gender, season,
    COUNT(CASE WHEN result != 'draw' AND rating_gap >= 50 THEN 1 END)                                                AS decisive_games,
    COUNT(CASE
      WHEN result = 'home_win' AND home_rating < away_rating AND rating_gap >= 50 THEN 1
      WHEN result = 'away_win' AND away_rating < home_rating AND rating_gap >= 50 THEN 1
    END) AS upset_count
  FROM game_results
  GROUP BY event_name, age_group, gender, season
),
event_full AS (
  SELECT
    b.event_name, b.age_group, b.gender, b.season,
    b.team_count,
    ROUND(b.avg_team_rating::numeric, 2)   AS avg_team_rating,
    ROUND(b.rating_spread::numeric, 2)     AS rating_spread,
    ROUND(b.min_rating::numeric, 2)        AS min_rating,
    ROUND(b.max_rating::numeric, 2)        AS max_rating,
    CASE WHEN u.decisive_games > 0
      THEN ROUND(u.upset_count::numeric / u.decisive_games, 4)
      ELSE NULL END                        AS upset_rate,
    u.decisive_games,
    u.upset_count
  FROM event_base b
  LEFT JOIN event_upsets u USING (event_name, age_group, gender, season)
),
event_percentiled AS (
  SELECT *,
    ROUND((PERCENT_RANK() OVER (
      PARTITION BY age_group, gender
      ORDER BY avg_team_rating
    ) * 100)::numeric, 2) AS strength_percentile
  FROM event_full
)
SELECT
  event_name, age_group, gender, season,
  team_count, avg_team_rating, rating_spread, min_rating, max_rating,
  upset_rate, decisive_games, upset_count,
  strength_percentile,
  CASE
    WHEN strength_percentile >= 80 THEN 'High'
    WHEN strength_percentile >= 40 THEN 'Average'
    ELSE 'Low'
  END AS strength_label,
  CASE
    WHEN rating_spread IS NULL     THEN 'Unknown'
    WHEN rating_spread < 75        THEN 'Tight'
    WHEN rating_spread < 150       THEN 'Mixed'
    ELSE 'Wide'
  END AS spread_label,
  CASE
    WHEN decisive_games IS NULL OR decisive_games < 3 THEN 'Unknown'
    WHEN upset_rate < 0.20         THEN 'Low'
    WHEN upset_rate < 0.40         THEN 'Average'
    ELSE 'High'
  END AS upset_label,
  true AS is_rated,
  true AS ratings_are_current,
  NOW() AS computed_at
FROM event_percentiled
ON CONFLICT (event_name, age_group, gender, season, computed_at) DO NOTHING;
`;

run().catch(err => {
  console.error('[event-strength] Fatal error:', err);
  process.exit(1);
});
```

### Step 7: Test on current season events

```bash
node scripts/compute-event-strength.js
```

Expected output:
```
[event-strength] Computing event strength for season 2025-2026...
[event-strength] Inserted XXXX rows in XXXms
```

Verify:
```sql
SELECT COUNT(*) FROM event_strength WHERE season = '2025-2026';
```
Expected: ≥ 1,000 rows (many events in the current season).

### Step 8: Spot-check 5 events manually

```sql
SELECT event_name, age_group, gender, team_count,
       avg_team_rating, strength_label, spread_label, upset_label,
       strength_percentile
FROM event_strength
WHERE season = '2025-2026'
ORDER BY avg_team_rating DESC
LIMIT 10;
```

Check: do the top events (by avg_team_rating) have `strength_label = 'High'`? Do smaller local events have `strength_label = 'Low'`? Spot-check 1–2 known tournaments by name against your intuition of their field quality.

### Step 9: Add to nightly cron

Add to the nightly pipeline (after `compute-rankings.js`):

```bash
# In whatever file manages the nightly pipeline (crontab or pipeline script)
# Run AFTER compute-rankings completes

node /path/to/scripts/compute-event-strength.js >> /var/log/evo-draw/event-strength.log 2>&1
```

If using a direct crontab, add:
```
0 3 * * * node /path/to/scripts/compute-event-strength.js >> /var/log/event-strength.log 2>&1
```
(3 AM daily, after rankings pipeline completes)

---

## Phase 3 — API (~2 hours)

The server is a plain Node.js HTTP server in `server.js`. Add two new route handlers.

### Step 10: Add `GET /api/events/:event_name/strength`

In `server.js`, add a new route handler. Since there's no Express, this goes in the main request router `if/else` chain:

```javascript
// In server.js request handler — add after existing route checks

// GET /api/events/[event-name-url-encoded]/strength?age_group=U14G&season=2025-2026
if (pathname.startsWith('/api/events/') && pathname.endsWith('/strength')) {
  const parts = pathname.split('/');
  // parts: ['', 'api', 'events', '<encoded-name>', 'strength']
  const eventName = decodeURIComponent(parts[3]);
  const ageGroup  = params.get('age_group');
  const season    = params.get('season') || '2025-2026';

  const conditions = ['event_name = $1', 'season = $2'];
  const values     = [eventName, season];
  let idx = 3;

  if (ageGroup) {
    conditions.push(`age_group = $${idx++}`);
    values.push(ageGroup);
  }

  const query = `
    SELECT
      event_name, age_group, gender, season, computed_at,
      team_count, avg_team_rating, rating_spread, min_rating, max_rating,
      upset_rate, decisive_games, upset_count,
      strength_percentile,
      strength_label, spread_label, upset_label,
      ratings_are_current
    FROM event_strength
    WHERE ${conditions.join(' AND ')}
    ORDER BY computed_at DESC
    LIMIT 1
  `;

  const result = await db.query(query, values);
  if (result.rows.length === 0) {
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Event strength not computed for this event' }));
  } else {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify(result.rows[0]));
  }
  return;
}

// GET /api/events?strength=High&age_group=U15G&season=2025-2026
if (pathname === '/api/events') {
  const strength = params.get('strength');
  const spread   = params.get('spread');
  const ageGroup = params.get('age_group');
  const season   = params.get('season') || '2025-2026';
  const limit    = parseInt(params.get('limit') || '25', 10);
  const offset   = parseInt(params.get('offset') || '0', 10);

  const conditions = ['season = $1', 'is_rated = true'];
  const values = [season];
  let idx = 2;

  if (strength) { conditions.push(`strength_label = $${idx++}`); values.push(strength); }
  if (spread)   { conditions.push(`spread_label = $${idx++}`);   values.push(spread); }
  if (ageGroup) { conditions.push(`age_group = $${idx++}`);      values.push(ageGroup); }

  // Use latest computed_at snapshot per event
  const query = `
    WITH latest AS (
      SELECT DISTINCT ON (event_name, age_group, gender)
        event_name, age_group, gender, season,
        team_count, avg_team_rating, strength_percentile,
        strength_label, spread_label, upset_label, upset_rate
      FROM event_strength
      WHERE ${conditions.join(' AND ')}
      ORDER BY event_name, age_group, gender, computed_at DESC
    )
    SELECT *, COUNT(*) OVER () AS total_count
    FROM latest
    ORDER BY avg_team_rating DESC
    LIMIT $${idx++} OFFSET $${idx}
  `;
  values.push(limit, offset);

  const result = await db.query(query, values);
  const total = result.rows[0]?.total_count || 0;
  const events = result.rows.map(({ total_count, ...rest }) => rest);

  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ total, events }));
  return;
}
```

### Step 11: Test both endpoints

```bash
# Test event strength detail (URL-encode the event name)
curl "http://localhost:3000/api/events/Dallas%20Spring%20Showcase%202026/strength?age_group=U14G"

# Test event search with filter
curl "http://localhost:3000/api/events?strength=High&age_group=U15G&limit=5"

# Confirm response includes numeric values (the differentiator vs USA Rank)
curl "http://localhost:3000/api/events?strength=High&age_group=U13G&limit=3" | python3 -m json.tool
```

**Verify:** Response includes both `strength_label` (the label) AND `avg_team_rating`, `rating_spread`, `upset_rate` (the numbers). This is the competitive differentiator — USA Rank shows labels only; we show the math.

---

## Phase 4 — Frontend (~4 hours)

### Step 12: Create the `EventStrengthCard` component

Create `frontend-v2/src/components/EventStrengthCard.jsx`:

```jsx
// EventStrengthCard — displays all 4 strength metrics for an event
// Competitive differentiator: shows numbers AND labels (USA Rank shows labels only)

const LABEL_COLORS = {
  High:    { bg: '#16A34A', text: '#fff' },
  Average: { bg: '#2563EB', text: '#fff' },
  Low:     { bg: '#6B7280', text: '#fff' },
  Tight:   { bg: '#0891B2', text: '#fff' },
  Mixed:   { bg: '#D97706', text: '#fff' },
  Wide:    { bg: '#DC2626', text: '#fff' },
  Unknown: { bg: '#9CA3AF', text: '#fff' },
};

function LabelBadge({ label }) {
  if (!label || label === 'Insufficient Data') return null;
  const colors = LABEL_COLORS[label] || LABEL_COLORS.Unknown;
  return (
    <span
      className="inline-flex items-center px-2 py-0.5 rounded text-xs font-semibold"
      style={{ backgroundColor: colors.bg, color: colors.text }}
    >
      {label}
    </span>
  );
}

export default function EventStrengthCard({ strength }) {
  if (!strength || !strength.is_rated) return null;

  const pct = strength.strength_percentile != null
    ? `${strength.strength_percentile.toFixed(1)}th percentile`
    : null;

  return (
    <div className="rounded-lg border border-gray-200 p-4 space-y-3 bg-white">
      <div className="flex items-center justify-between">
        <h3 className="text-sm font-semibold text-gray-700">Event Strength</h3>
        {strength.ratings_are_current && (
          <span className="text-xs text-gray-400 italic">Based on current ratings</span>
        )}
      </div>

      {/* Row 1: Strength classification */}
      <div className="flex items-center justify-between">
        <span className="text-sm text-gray-600">Field Strength</span>
        <div className="flex items-center gap-2">
          <LabelBadge label={strength.strength_label} />
          <span className="text-xs text-gray-500">
            Avg {Math.round(strength.avg_team_rating)} rating
            {pct ? ` · ${pct}` : ''}
          </span>
        </div>
      </div>

      {/* Row 2: Competitive spread */}
      <div className="flex items-center justify-between">
        <span className="text-sm text-gray-600">Competitive Spread</span>
        <div className="flex items-center gap-2">
          <LabelBadge label={strength.spread_label} />
          {strength.rating_spread != null && (
            <span className="text-xs text-gray-500">
              σ = {Math.round(strength.rating_spread)} pts
            </span>
          )}
        </div>
      </div>

      {/* Row 3: Upset rate */}
      <div className="flex items-center justify-between">
        <span className="text-sm text-gray-600">Upset Rate</span>
        <div className="flex items-center gap-2">
          <LabelBadge label={strength.upset_label} />
          {strength.upset_rate != null && (
            <span className="text-xs text-gray-500">
              {(strength.upset_rate * 100).toFixed(1)}% of decisive games
            </span>
          )}
        </div>
      </div>

      {/* Footer: team count */}
      <div className="text-xs text-gray-400 pt-1 border-t border-gray-100">
        {strength.team_count} rated teams · {strength.decisive_games || 0} decisive games analyzed
      </div>
    </div>
  );
}
```

**Why this is better than USA Rank:** USA Rank shows `High` label only. Evo Draw shows `High · Avg 1321 rating · 91.2th percentile` + spread sigma + upset percentage. Coaches can see exactly how strong the field was, not just a tier label.

### Step 13: Add the card to event detail pages

Find the event detail component (likely `frontend-v2/src/pages/Event.jsx` or similar). Add:

```jsx
import EventStrengthCard from '../components/EventStrengthCard';
import { useEffect, useState } from 'react';

// In the event detail component:
const [strength, setStrength] = useState(null);

useEffect(() => {
  if (!eventName || !ageGroup) return;
  const encoded = encodeURIComponent(eventName);
  fetch(`/api/events/${encoded}/strength?age_group=${ageGroup}&season=${season}`)
    .then(r => r.ok ? r.json() : null)
    .then(data => setStrength(data))
    .catch(() => null);
}, [eventName, ageGroup, season]);

// In the JSX:
{strength && <EventStrengthCard strength={strength} />}
```

### Step 14: Add strength filter to event search/browse pages

If an events listing page exists:

```jsx
const STRENGTH_OPTIONS = ['All', 'High', 'Average', 'Low'];

const [selectedStrength, setSelectedStrength] = useState('All');

// Filter UI
<select value={selectedStrength} onChange={e => setSelectedStrength(e.target.value)}>
  {STRENGTH_OPTIONS.map(s => <option key={s} value={s}>{s}</option>)}
</select>

// API call
const strengthParam = selectedStrength !== 'All' ? `&strength=${selectedStrength}` : '';
const url = `/api/events?season=${season}${ageGroupParam}${strengthParam}`;
```

### Step 15: Test frontend

1. `cd frontend-v2 && npm run dev`
2. Load an event detail page — confirm the strength card appears with labels AND numbers
3. Load 3 different events — verify cards render with different labels (not all "High")
4. Test the event search filter — "High" should return fewer events than "All"
5. Verify the card is absent for events with `strength_label = 'Insufficient Data'`

---

## Code Artifacts Summary

| Artifact | Location in Plan |
|----------|-----------------|
| A. `CREATE TABLE event_strength` DDL | Phase 1 Step 1 |
| B. Full computation SQL (CTE chain) | Phase 2 Step 6, `computeSQL` const |
| C. `compute-event-strength.js` script | Phase 2 Step 6 |
| D. API handler stubs (both endpoints) | Phase 3 Steps 10–11 |
| E. `EventStrengthCard` component | Phase 4 Step 12 |

---

## Edge Case Handling Confirmation

| Edge Case | Where Handled |
|-----------|--------------|
| Event with < 4 rated teams | `HAVING COUNT(*) >= 4` in `event_base` CTE; `strength_label = 'Insufficient Data'` |
| All games are draws | `decisive_games = 0`, `upset_rate = NULL`, `upset_label = 'Unknown'` |
| Event with no teams in rankings | Excluded by JOIN to rankings — team_count = 0, not inserted |
| Multi-age-group events | Separate rows per `(event_name, age_group, gender)` tuple |
| Re-runs (nightly) | New row with new `computed_at`; UI uses `ORDER BY computed_at DESC LIMIT 1` |
| Events with < 3 decisive games | `upset_label = 'Unknown'` |
| Rating spread = NULL (single team) | `spread_label = 'Unknown'` |

---

## Rollback Plan

**Drop the table:**
```sql
DROP TABLE IF EXISTS event_strength;
```

**Remove API endpoints:**
Remove the two route handlers from `server.js`. No other routes are affected.

**Nightly cron:**
Comment out the `compute-event-strength.js` line in the pipeline script.

**Frontend:**
Remove `EventStrengthCard` from event pages. No other components reference it.

---

## Definition of Done

- [ ] `event_strength` table exists with correct schema
- [ ] `compute-event-strength.js` runs without errors and inserts ≥ 1,000 rows for 2025-2026 season
- [ ] At least 100 events have strength ratings (not 'Insufficient Data')
- [ ] `GET /api/events/[name]/strength` returns correct data for 5 manually tested events
- [ ] `GET /api/events?strength=High&age_group=U15G` returns only High-strength events
- [ ] API response includes numeric values (`avg_team_rating`, `rating_spread`, `upset_rate`) — not just labels
- [ ] Event detail page shows `EventStrengthCard` with labels AND numbers
- [ ] Nightly computation is scheduled after `compute-rankings.js`
- [ ] All edge cases confirmed handled per table above

---

## Timeline to Demo-Ready (May 9, 2026)

| Date | Milestone |
|------|-----------|
| April 28 | Start — complete Phase 1 (DB) and Phase 2 (computation script) |
| April 29–30 | Complete Phase 3 (API) — both endpoints working |
| May 1–2 | Complete Phase 4 (Frontend) — strength card on event pages |
| May 3–7 | QA and edge case testing; spot-check 20+ events |
| May 9 | Demo-ready — Presten can demo event strength cards |
| May 16 | DSS demo |

---

## Related Notes

- [[Event Strength Ratings Design]] — source of truth for all decisions and SQL
- [[Database Schema]] — `games`, `rankings` table structure
- [[Server and Frontend]] — server.js and React/Vite integration
- [[USA Rank Competitive Intel]] — competitor event strength for reference
- [[Ranking Engine]] — Phase 7 SOS context
