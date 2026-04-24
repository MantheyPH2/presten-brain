---
title: Club Rankings — Implementation Plan
aliases: [Club Rankings Implementation]
tags: [rankings, clubs, implementation, evo-draw]
created: 2026-04-23
updated: 2026-04-23
status: ready-to-implement
author: ELO Agent
design_doc: "[[Club Rankings Design]]"
dss_deadline: 2026-05-16
start_by: 2026-05-02
estimated_hours: 10
---

# Club Rankings — Implementation Plan

> Design is complete (`[[Club Rankings Design]]`). Methodology: AVG of top team rating per age group (U11–U17), 5+ of 7 age groups required, separate boys/girls rankings. This plan converts the design into a sequenced ~10-hour session. All 5 code artifacts are complete. No design decisions left open.
>
> **Competitive differentiator:** Club directors search for their club's standing. We cover 40% more teams than USA Rank — club rankings built on our deeper game coverage will surface clubs USA Rank misses entirely.

---

## Pre-Flight Checklist

Run these before writing any feature code. ~30 minutes total.

**1. Confirm club identifier on `teams` table:**
```sql
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'teams'
ORDER BY ordinal_position;
```
Look for: `club`, `gotsport_org_id`, `org_id`, or similar. The design uses GotSport org_id as the primary club key.

**2. Run coverage estimate:**
```sql
SELECT
  COUNT(*) AS total_teams,
  COUNT(CASE WHEN club IS NOT NULL AND club != '' THEN 1 END) AS with_club_name,
  COUNT(CASE WHEN gotsport_org_id IS NOT NULL THEN 1 END) AS with_org_id,
  ROUND(COUNT(CASE WHEN club IS NOT NULL AND club != '' THEN 1 END) * 100.0 / COUNT(*), 1) AS pct_club_name,
  ROUND(COUNT(CASE WHEN gotsport_org_id IS NOT NULL THEN 1 END) * 100.0 / COUNT(*), 1) AS pct_org_id
FROM teams;
```
**If org_id coverage ≥ 60% → proceed with Case A (clean path).**
**If org_id coverage < 60% → proceed with Case B (name-parse path) — see Bootstrap Contingency.**

**3. Confirm `rankings` table has age_group and gender:**
```sql
SELECT DISTINCT age_group, gender FROM rankings ORDER BY age_group, gender;
```
Expected: U11–U17, M/F entries. If U11 or U12 are missing, the club ranking will use fewer age groups — note this.

**4. Confirm neither `clubs` nor `club_rankings` table exists:**
```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
  AND table_name IN ('clubs', 'club_rankings', 'team_club_assignments');
```
Expected: 0 rows. If any exist, review before proceeding (may be from a prior attempt).

**5. Get a count of active ranked teams (activity baseline):**
```sql
SELECT age_group, gender, COUNT(*) AS ranked_teams
FROM rankings
WHERE age_group IN ('U11','U12','U13','U14','U15','U16','U17')
  AND last_game_date > NOW() - INTERVAL '7 months'
GROUP BY age_group, gender
ORDER BY age_group, gender;
```
This sets the expectation for how many teams will feed into club ranking computation.

---

## Phase 1 — Bootstrap the `clubs` Table (~2 hours)

### Step 1: Create the three tables

```sql
-- Canonical club registry
CREATE TABLE clubs (
  id                  SERIAL PRIMARY KEY,
  name                VARCHAR(200) NOT NULL,
  normalized_name     VARCHAR(200) NOT NULL,
  state               CHAR(2),
  gotsport_org_id     VARCHAR(50),
  website             VARCHAR(300),
  is_verified         BOOLEAN DEFAULT false,
  created_at          TIMESTAMPTZ DEFAULT NOW(),
  updated_at          TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE (gotsport_org_id),
  UNIQUE (normalized_name, state)
);

-- Many-to-one: team → club assignment
CREATE TABLE team_club_assignments (
  team_id             VARCHAR(100) NOT NULL,
  club_id             INT NOT NULL REFERENCES clubs(id) ON DELETE CASCADE,
  assignment_method   VARCHAR(20) NOT NULL,  -- 'gotsport_org_id', 'name_parse', 'manual'
  confidence          NUMERIC(3,2),          -- 0.00–1.00
  assigned_at         TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (team_id)
);

-- Club ranking output (computed nightly)
CREATE TABLE club_rankings (
  club_id             INT NOT NULL REFERENCES clubs(id) ON DELETE CASCADE,
  gender              CHAR(1) NOT NULL CHECK (gender IN ('M', 'F')),
  computed_at         TIMESTAMPTZ NOT NULL,
  club_rating         NUMERIC(8,2) NOT NULL,
  club_rank           INT,
  age_groups_counted  INT NOT NULL,
  age_groups_eligible JSONB,
  top_teams           JSONB NOT NULL,
  state               CHAR(2),
  PRIMARY KEY (club_id, gender, computed_at)
);

-- Indexes
CREATE INDEX idx_club_rankings_gender_rank ON club_rankings (gender, club_rank)
  WHERE club_rank IS NOT NULL;
CREATE INDEX idx_club_rankings_state ON club_rankings (state, gender, club_rank);
CREATE INDEX idx_team_club_assignments_club ON team_club_assignments (club_id);
```

Verify:
```bash
psql -d youth_soccer -c "\d clubs"
psql -d youth_soccer -c "\d club_rankings"
psql -d youth_soccer -c "\d team_club_assignments"
```

### Step 2: Seed clubs from GotSport org_id (Case A — primary path)

**Use this if pre-flight Step 2 shows org_id coverage ≥ 60%:**

```sql
-- Insert distinct clubs from GotSport org_id data
INSERT INTO clubs (name, normalized_name, gotsport_org_id, state)
SELECT DISTINCT
  club                                                                          AS name,
  LOWER(TRIM(REGEXP_REPLACE(club, '[^a-zA-Z0-9 ]', '', 'g')))                  AS normalized_name,
  gotsport_org_id,
  NULL                                                                          AS state
FROM teams
WHERE gotsport_org_id IS NOT NULL
  AND club IS NOT NULL
  AND club != ''
ON CONFLICT (gotsport_org_id) DO NOTHING;
```

Then create team assignments:
```sql
INSERT INTO team_club_assignments (team_id, club_id, assignment_method, confidence)
SELECT
  t.team_id,
  c.id,
  'gotsport_org_id',
  1.0
FROM teams t
JOIN clubs c ON c.gotsport_org_id = t.gotsport_org_id
WHERE t.gotsport_org_id IS NOT NULL
ON CONFLICT (team_id) DO NOTHING;
```

### Step 3: Fill remaining teams via name-parse (both paths)

For teams without a GotSport org_id but with a populated `club` column:
```sql
-- First: insert any new clubs not yet in the clubs table via name-parse
INSERT INTO clubs (name, normalized_name, state)
SELECT DISTINCT
  club,
  LOWER(TRIM(REGEXP_REPLACE(club, '[^a-zA-Z0-9 ]', '', 'g'))),
  NULL
FROM teams
WHERE team_id NOT IN (SELECT team_id FROM team_club_assignments)
  AND club IS NOT NULL
  AND club != ''
ON CONFLICT (normalized_name, state) DO NOTHING;

-- Then: assign those teams
INSERT INTO team_club_assignments (team_id, club_id, assignment_method, confidence)
SELECT
  t.team_id,
  c.id,
  'name_parse',
  0.75
FROM teams t
JOIN clubs c ON c.normalized_name = LOWER(TRIM(REGEXP_REPLACE(t.club, '[^a-zA-Z0-9 ]', '', 'g')))
WHERE t.team_id NOT IN (SELECT team_id FROM team_club_assignments)
  AND t.club IS NOT NULL
  AND t.club != ''
ON CONFLICT (team_id) DO NOTHING;
```

### Step 4: Verify seeding

```sql
SELECT COUNT(*) AS club_count FROM clubs;
SELECT COUNT(*) AS assignment_count FROM team_club_assignments;
SELECT assignment_method, COUNT(*) AS count FROM team_club_assignments GROUP BY assignment_method;
```

Expected: clubs table has 5,000–20,000 rows; assignments cover 70%+ of teams.

### Step 5: Spot-check 10 club assignments

```sql
SELECT c.name AS club_name, t.team_name, tca.assignment_method, tca.confidence
FROM team_club_assignments tca
JOIN clubs c ON c.id = tca.club_id
JOIN teams t ON t.team_id = tca.team_id
WHERE c.name ILIKE '%FC Dallas%'
  OR c.name ILIKE '%Eclipse%'
  OR c.name ILIKE '%PA Classics%'
ORDER BY c.name, t.team_name
LIMIT 30;
```

Confirm: teams assigned to "FC Dallas" actually look like FC Dallas teams (not a different club with a similar name).

---

## Phase 2 — Computation Script (~3 hours)

### Step 6: Write `compute-club-rankings.js`

Create `scripts/compute-club-rankings.js`:

```javascript
#!/usr/bin/env node
// compute-club-rankings.js
// Computes club rankings for boys and girls separately.
// Run after compute-rankings.js in the nightly pipeline.

const { Pool } = require('pg');

const pool = new Pool({
  host: 'localhost',
  port: 5432,
  database: 'youth_soccer',
  user: 'p',
});

async function run() {
  const client = await pool.connect();
  try {
    console.log('[club-rankings] Starting computation...');
    const start = Date.now();

    const result = await client.query(computeSQL);
    console.log(`[club-rankings] Inserted ${result.rowCount} rows in ${Date.now() - start}ms`);
  } finally {
    client.release();
    await pool.end();
  }
}

const computeSQL = `
INSERT INTO club_rankings
  (club_id, gender, computed_at, club_rating, club_rank, age_groups_counted, age_groups_eligible, top_teams, state)

WITH

ranked_teams AS (
  SELECT
    tca.club_id,
    c.name           AS club_name,
    c.state,
    r.team_id,
    t.team_name,
    r.age_group,
    r.gender,
    r.rating,
    r.last_game_date
  FROM rankings r
  JOIN team_club_assignments tca ON tca.team_id = r.team_id
  JOIN clubs c ON c.id = tca.club_id
  JOIN teams t ON t.team_id = r.team_id
  WHERE
    r.age_group IN ('U11','U12','U13','U14','U15','U16','U17')
    AND r.rating IS NOT NULL
    AND r.last_game_date > NOW() - INTERVAL '7 months'
),

top_team_per_agegroup AS (
  SELECT DISTINCT ON (club_id, gender, age_group)
    club_id, club_name, state, gender, age_group, team_id, team_name, rating
  FROM ranked_teams
  ORDER BY club_id, gender, age_group, rating DESC
),

club_aggregates AS (
  SELECT
    club_id,
    club_name,
    state,
    gender,
    COUNT(*)                AS age_groups_counted,
    AVG(rating)             AS club_rating,
    JSONB_AGG(
      JSONB_BUILD_OBJECT(
        'age_group', age_group,
        'team_id',   team_id,
        'team_name', team_name,
        'rating',    ROUND(rating::numeric, 2)
      )
      ORDER BY age_group
    )                       AS top_teams,
    JSONB_AGG(age_group ORDER BY age_group) AS age_groups_eligible
  FROM top_team_per_agegroup
  GROUP BY club_id, club_name, state, gender
),

eligible_clubs AS (
  SELECT * FROM club_aggregates WHERE age_groups_counted >= 5
),

ranked_clubs AS (
  SELECT
    club_id,
    gender,
    ROUND(club_rating::numeric, 2) AS club_rating,
    RANK() OVER (PARTITION BY gender ORDER BY club_rating DESC) AS club_rank,
    age_groups_counted,
    age_groups_eligible,
    top_teams,
    state
  FROM eligible_clubs
)

SELECT
  club_id, gender, NOW() AS computed_at, club_rating, club_rank,
  age_groups_counted, age_groups_eligible, top_teams, state
FROM ranked_clubs;
`;

run().catch(err => {
  console.error('[club-rankings] Fatal error:', err);
  process.exit(1);
});
```

### Step 7: Test on current data

```bash
node scripts/compute-club-rankings.js
```

Verify:
```sql
SELECT gender, COUNT(*) AS club_count, AVG(club_rating) AS avg_rating
FROM club_rankings
GROUP BY gender;
```

Expected: 50+ qualifying clubs per gender (clubs with 5+ active age groups).

### Step 8: Spot-check known clubs

```sql
SELECT c.name, cr.gender, cr.club_rank, cr.club_rating, cr.age_groups_counted
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE c.name ILIKE ANY(ARRAY['%Eclipse%', '%FC Dallas%', '%PA Classics%', '%De Anza%', '%Surf%'])
ORDER BY gender, club_rank;
```

Confirm: national powerhouse clubs appear in the top 100 for their gender. If they're not present, check whether they have 5+ active age groups in the data.

### Step 9: Add to nightly cron

Add after `compute-rankings.js` (and after `compute-event-strength.js`) in the nightly pipeline:

```bash
node /path/to/scripts/compute-club-rankings.js >> /var/log/evo-draw/club-rankings.log 2>&1
```

Or as a crontab line (runs at 4 AM, after the 3 AM event strength job):
```
0 4 * * * node /path/to/scripts/compute-club-rankings.js >> /var/log/club-rankings.log 2>&1
```

---

## Phase 3 — API (~2 hours)

The server is a plain Node.js HTTP server in `server.js`. Add two new route handlers.

### Step 10: Add `GET /api/club-rankings`

In `server.js`, add:

```javascript
// GET /api/club-rankings?gender=F&state=TX&limit=50&offset=0
if (pathname === '/api/club-rankings') {
  const gender = params.get('gender');
  const state  = params.get('state');
  const limit  = parseInt(params.get('limit') || '50', 10);
  const offset = parseInt(params.get('offset') || '0', 10);

  if (!gender || !['M', 'F'].includes(gender)) {
    res.writeHead(400, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'gender parameter required: M or F' }));
    return;
  }

  const conditions = ['cr.gender = $1', 'cr.club_rank IS NOT NULL'];
  const values = [gender];
  let idx = 2;

  if (state) {
    conditions.push(`cr.state = $${idx++}`);
    values.push(state.toUpperCase());
  }

  // Use latest computed_at snapshot per club
  const query = `
    WITH latest AS (
      SELECT DISTINCT ON (club_id, gender)
        club_id, gender, computed_at, club_rating, club_rank,
        age_groups_counted, age_groups_eligible, top_teams, state
      FROM club_rankings
      WHERE ${conditions.join(' AND ')}
      ORDER BY club_id, gender, computed_at DESC
    )
    SELECT
      l.*,
      c.name AS club_name,
      c.gotsport_org_id,
      COUNT(*) OVER () AS total_count
    FROM latest l
    JOIN clubs c ON c.id = l.club_id
    ORDER BY l.club_rank
    LIMIT $${idx++} OFFSET $${idx}
  `;
  values.push(limit, offset);

  const result = await db.query(query, values);
  const total = result.rows[0]?.total_count || 0;
  const clubs = result.rows.map(({ total_count, ...rest }) => rest);

  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ gender, total, computed_at: clubs[0]?.computed_at, clubs }));
  return;
}
```

### Step 11: Add `GET /api/clubs/:club_id`

```javascript
// GET /api/clubs/:club_id
const clubMatch = pathname.match(/^\/api\/clubs\/(\d+)$/);
if (clubMatch) {
  const clubId = parseInt(clubMatch[1], 10);

  const clubQuery = `SELECT * FROM clubs WHERE id = $1`;
  const clubResult = await db.query(clubQuery, [clubId]);

  if (clubResult.rows.length === 0) {
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Club not found' }));
    return;
  }

  const club = clubResult.rows[0];

  // Get latest rankings for both genders
  const rankingsQuery = `
    SELECT DISTINCT ON (gender)
      gender, club_rank, club_rating, age_groups_counted, age_groups_eligible, top_teams, computed_at
    FROM club_rankings
    WHERE club_id = $1 AND club_rank IS NOT NULL
    ORDER BY gender, computed_at DESC
  `;
  const rankingsResult = await db.query(rankingsQuery, [clubId]);

  const rankings = {};
  for (const row of rankingsResult.rows) {
    rankings[row.gender === 'F' ? 'girls' : 'boys'] = {
      club_rank: row.club_rank,
      club_rating: row.club_rating,
      age_groups_counted: row.age_groups_counted,
      age_groups_eligible: row.age_groups_eligible,
      top_teams: row.top_teams,
      computed_at: row.computed_at,
    };
  }

  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ ...club, rankings }));
  return;
}
```

### Step 12: Test both endpoints

```bash
# Club rankings list — girls
curl "http://localhost:3000/api/club-rankings?gender=F&limit=5"

# Club rankings list — boys, Texas filter
curl "http://localhost:3000/api/club-rankings?gender=M&state=TX&limit=5"

# Individual club profile (use a real club_id from the above)
curl "http://localhost:3000/api/clubs/1"
```

Confirm: `top_teams` array is present in responses and contains age_group/team_name/rating breakdown.

### Step 13: Confirm `top_teams` JSONB structure

```bash
curl "http://localhost:3000/api/club-rankings?gender=F&limit=1" | python3 -m json.tool
```

Expected structure:
```json
{
  "gender": "F",
  "total": 312,
  "clubs": [
    {
      "club_rank": 1,
      "club_name": "Eclipse Select SC",
      "state": "IL",
      "club_rating": 1478.50,
      "age_groups_counted": 7,
      "age_groups_eligible": ["U11","U12","U13","U14","U15","U16","U17"],
      "top_teams": [
        { "age_group": "U13", "team_id": "12345", "team_name": "Eclipse Select 2012G ECNL", "rating": 1510.20 },
        { "age_group": "U14", "team_id": "12346", "team_name": "Eclipse Select 2011G ECNL", "rating": 1499.80 }
      ]
    }
  ]
}
```

---

## Phase 4 — Frontend (~3 hours)

### Step 14: Add club rankings page (`/club-rankings`)

Create `frontend-v2/src/pages/ClubRankings.jsx`:

```jsx
import { useState, useEffect } from 'react';

const STATES = ['All','AL','AK','AZ','AR','CA','CO','CT','DE','FL','GA','HI','ID','IL','IN','IA',
  'KS','KY','LA','ME','MD','MA','MI','MN','MS','MO','MT','NE','NV','NH','NJ','NM','NY','NC','ND',
  'OH','OK','OR','PA','RI','SC','SD','TN','TX','UT','VT','VA','WA','WV','WI','WY'];

export default function ClubRankings() {
  const [gender, setGender] = useState('F');
  const [state, setState] = useState('All');
  const [clubs, setClubs] = useState([]);
  const [total, setTotal] = useState(0);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    setLoading(true);
    const stateParam = state !== 'All' ? `&state=${state}` : '';
    fetch(`/api/club-rankings?gender=${gender}${stateParam}&limit=100`)
      .then(r => r.json())
      .then(data => { setClubs(data.clubs || []); setTotal(data.total || 0); })
      .finally(() => setLoading(false));
  }, [gender, state]);

  return (
    <div className="p-4 max-w-5xl mx-auto">
      <h1 className="text-2xl font-bold mb-4">Club Rankings</h1>

      {/* Controls */}
      <div className="flex gap-3 mb-4">
        <div className="flex rounded overflow-hidden border">
          {['F','M'].map(g => (
            <button key={g}
              className={`px-4 py-2 text-sm font-medium ${gender === g ? 'bg-blue-600 text-white' : 'bg-white text-gray-700'}`}
              onClick={() => setGender(g)}
            >
              {g === 'F' ? 'Girls' : 'Boys'}
            </button>
          ))}
        </div>
        <select value={state} onChange={e => setState(e.target.value)}
          className="border rounded px-2 py-1 text-sm">
          {STATES.map(s => <option key={s} value={s}>{s === 'All' ? 'All States' : s}</option>)}
        </select>
        <span className="text-sm text-gray-500 self-center">{total} clubs</span>
      </div>

      {/* Rankings table */}
      {loading ? <p className="text-gray-500">Loading...</p> : (
        <table className="w-full text-sm border-collapse">
          <thead>
            <tr className="border-b bg-gray-50">
              <th className="py-2 px-3 text-left font-medium text-gray-600">Rank</th>
              <th className="py-2 px-3 text-left font-medium text-gray-600">Club</th>
              <th className="py-2 px-3 text-left font-medium text-gray-600">State</th>
              <th className="py-2 px-3 text-right font-medium text-gray-600">Rating</th>
              <th className="py-2 px-3 text-left font-medium text-gray-600">Age Groups</th>
            </tr>
          </thead>
          <tbody>
            {clubs.map(club => (
              <tr key={club.club_id} className="border-b hover:bg-gray-50 cursor-pointer"
                onClick={() => window.location.href = `/clubs/${club.club_id}`}>
                <td className="py-2 px-3 font-semibold text-gray-700">#{club.club_rank}</td>
                <td className="py-2 px-3 font-medium">{club.club_name}</td>
                <td className="py-2 px-3 text-gray-500">{club.state || '—'}</td>
                <td className="py-2 px-3 text-right tabular-nums">{club.club_rating?.toFixed(1)}</td>
                <td className="py-2 px-3">
                  <AgeGroupCoverage
                    eligible={club.age_groups_eligible}
                    count={club.age_groups_counted}
                  />
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      )}
    </div>
  );
}

function AgeGroupCoverage({ eligible, count }) {
  const ALL = ['U11','U12','U13','U14','U15','U16','U17'];
  return (
    <div className="flex gap-0.5">
      {ALL.map(ag => (
        <span key={ag}
          className={`text-xs px-1 rounded ${eligible?.includes(ag) ? 'bg-blue-100 text-blue-700' : 'bg-gray-100 text-gray-400'}`}>
          {ag.replace('U','')}
        </span>
      ))}
    </div>
  );
}
```

### Step 15: Add club detail page (`/clubs/:id`)

Create `frontend-v2/src/pages/ClubDetail.jsx`:

```jsx
import { useState, useEffect } from 'react';
import { useParams } from 'react-router-dom';

export default function ClubDetail() {
  const { id } = useParams();
  const [club, setClub] = useState(null);
  const [activeGender, setActiveGender] = useState('F');

  useEffect(() => {
    fetch(`/api/clubs/${id}`)
      .then(r => r.json())
      .then(setClub);
  }, [id]);

  if (!club) return <p className="p-4">Loading...</p>;

  const rankings = club.rankings || {};
  const genderKey = activeGender === 'F' ? 'girls' : 'boys';
  const data = rankings[genderKey];

  return (
    <div className="p-4 max-w-3xl mx-auto">
      <h1 className="text-2xl font-bold mb-1">{club.name}</h1>
      <p className="text-gray-500 mb-4">{club.state || 'State unknown'}</p>

      {/* Gender toggle */}
      <div className="flex rounded overflow-hidden border inline-flex mb-6">
        {['F','M'].map(g => (
          <button key={g}
            className={`px-4 py-2 text-sm font-medium ${activeGender === g ? 'bg-blue-600 text-white' : 'bg-white text-gray-700'}`}
            onClick={() => setActiveGender(g)}
          >
            {g === 'F' ? 'Girls' : 'Boys'}
          </button>
        ))}
      </div>

      {data ? (
        <div>
          <div className="mb-4 p-4 bg-gray-50 rounded-lg flex gap-6">
            <div>
              <div className="text-3xl font-bold">#{data.club_rank}</div>
              <div className="text-sm text-gray-500">National Rank</div>
            </div>
            <div>
              <div className="text-3xl font-bold">{data.club_rating?.toFixed(1)}</div>
              <div className="text-sm text-gray-500">Club Rating</div>
            </div>
            <div>
              <div className="text-3xl font-bold">{data.age_groups_counted}/7</div>
              <div className="text-sm text-gray-500">Age Groups</div>
            </div>
          </div>

          <h2 className="text-lg font-semibold mb-3">Top Teams by Age Group</h2>
          <table className="w-full text-sm">
            <thead>
              <tr className="border-b bg-gray-50">
                <th className="py-2 px-3 text-left font-medium text-gray-600">Age Group</th>
                <th className="py-2 px-3 text-left font-medium text-gray-600">Team</th>
                <th className="py-2 px-3 text-right font-medium text-gray-600">Rating</th>
              </tr>
            </thead>
            <tbody>
              {(data.top_teams || []).map(t => (
                <tr key={t.age_group} className="border-b">
                  <td className="py-2 px-3 font-medium text-blue-600">{t.age_group}</td>
                  <td className="py-2 px-3">{t.team_name}</td>
                  <td className="py-2 px-3 text-right tabular-nums">{t.rating?.toFixed(1)}</td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      ) : (
        <p className="text-gray-500">No {activeGender === 'F' ? 'girls' : 'boys'} club ranking available (requires 5+ active age groups).</p>
      )}
    </div>
  );
}
```

### Step 16: Add gender toggle and state filter

The gender toggle and state filter are built into `ClubRankings.jsx` above (Step 14).

### Step 17: Wire up routing

In the React router config (likely `App.jsx`), add:
```jsx
<Route path="/club-rankings" element={<ClubRankings />} />
<Route path="/clubs/:id" element={<ClubDetail />} />
```

Add a nav link to `/club-rankings` in the main navigation.

### Step 18: Test frontend

1. `cd frontend-v2 && npm run dev`
2. Load `/club-rankings` — confirm table renders with rank, name, state, rating, age group coverage
3. Toggle Boys / Girls — confirm different data loads
4. Select Texas in state filter — confirm only TX clubs appear
5. Click a club row — confirm it navigates to `/clubs/:id`
6. On club detail page, confirm top-teams table shows age group breakdown
7. Toggle gender on club detail — confirm data switches

---

## Code Artifacts Summary

| Artifact | Location in Plan |
|----------|-----------------|
| A. DDL (`clubs`, `team_club_assignments`, `club_rankings`) | Phase 1 Step 1 |
| B. Bootstrap SQL (org_id path + name-parse path) | Phase 1 Steps 2–3 |
| C. Computation SQL (full CTE chain) | Phase 2 Step 6, `computeSQL` const |
| D. `compute-club-rankings.js` | Phase 2 Step 6 |
| E. API handlers (both endpoints) | Phase 3 Steps 10–11 |

---

## Bootstrap Contingency

### Case A: GotSport org_id available on `teams` table (the clean path)

Pre-flight Step 2 shows `pct_org_id ≥ 60%`. Proceed with Phase 1 Step 2 exactly as written. The `gotsport_org_id` column is the primary key for club identity — high confidence, low false positive rate.

### Case B: org_id NOT on `teams`, or coverage < 60%

Use the `club` column with name normalization as the only identifier. Steps 2–3 still apply, but skip the org_id INSERT and go straight to the name-parse path.

**Parsing rule for team name → club name (when `club` column is empty):**

```sql
-- Strip age/gender/year suffixes to extract club name from team name
-- Pattern: "FC Dallas 2011G ECNL" → "FC Dallas"
SELECT
  team_name,
  TRIM(REGEXP_REPLACE(
    team_name,
    '\s+(U\d{1,2}[BG]?|20\d{2}[BG]?|Boys|Girls|ECNL|NPL|GA|MLS|Academy|Elite|Premier|Black|Red|Blue|White|Gold)\b.*$',
    '', 'i'
  )) AS parsed_club_name
FROM teams
LIMIT 20;
```

Run this query and inspect the output. If more than 20% of parsed names look wrong (e.g., "2012" or "ECNL" appearing as the club name), adjust the regex before running the INSERT.

**False positive rate for name-parse:** ~10–20% expected. Priority-review the top 100 clubs by ranked team count:
```sql
SELECT c.name, COUNT(DISTINCT tca.team_id) AS assigned_teams
FROM clubs c
JOIN team_club_assignments tca ON tca.club_id = c.id
WHERE tca.assignment_method = 'name_parse'
GROUP BY c.name
ORDER BY assigned_teams DESC
LIMIT 100;
```
Manually review any club name that looks wrong. Set `is_verified = false` by default; Presten marks top-ranked clubs as verified before DSS.

---

## Rollback Plan

**Drop all three tables:**
```sql
DROP TABLE IF EXISTS club_rankings;
DROP TABLE IF EXISTS team_club_assignments;
DROP TABLE IF EXISTS clubs;
```

**Remove API endpoints:** Remove the two route handlers from `server.js`. No other routes are affected.

**Remove frontend pages:** Remove `ClubRankings.jsx` and `ClubDetail.jsx` and the two routes from `App.jsx`.

**Disable nightly cron:** Comment out the `compute-club-rankings.js` line in the pipeline script.

None of these rollbacks require a DB schema migration beyond table drops.

---

## Definition of Done

- [ ] `clubs` table seeded with coverage ≥ 60% of ranked teams (pre-flight Step 2 confirms)
- [ ] `club_rankings` table populated for both boys and girls
- [ ] At least 50 clubs qualify (5+ age groups) per gender
- [ ] `GET /api/club-rankings?gender=F` returns correct data with `top_teams` JSONB
- [ ] `GET /api/clubs/:id` returns correct team breakdown per age group
- [ ] `/club-rankings` page renders with sortable table, gender toggle, state filter
- [ ] `/clubs/:id` page shows age group breakdown with ratings
- [ ] Known-club spot check passes (Eclipse, FC Dallas, PA Classics appear at plausible ranks)
- [ ] Nightly recompute scheduled after `compute-rankings.js`
- [ ] Age group coverage indicator renders correctly (green for present, gray for absent)

---

## Timeline to DSS (May 16, 2026)

| Date | Milestone |
|------|-----------|
| May 2 | Start — complete Phase 1 (tables + bootstrap) |
| May 3–4 | Complete Phase 2 (computation script + cron) |
| May 5–6 | Complete Phase 3 (API endpoints) |
| May 7–10 | Complete Phase 4 (frontend pages) |
| May 11–14 | QA, spot-checks, top-club verification |
| May 14 | Demo-ready |
| May 16 | DSS demo |

**Total estimate: ~10 hours.** If implementation starts May 2, club rankings are demoable by May 14 — 2 days before DSS.

---

## Related Notes

- [[Club Rankings Design]] — source of truth for all decisions and methodology
- [[Database Schema]] — `teams`, `rankings` table structure
- [[Server and Frontend]] — server.js and React/Vite frontend details
- [[Rank Bands — Implementation Plan]] — reference format
- [[Event Strength Ratings — Implementation Plan]] — reference format
- [[USA Rank Competitive Intel]] — competitor club ranking methodology
