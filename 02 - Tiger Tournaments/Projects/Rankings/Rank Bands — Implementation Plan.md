---
title: Rank Bands — Implementation Plan
aliases: [Rank Bands Implementation, Band Implementation]
tags: [rankings, rank-bands, implementation, evo-draw]
created: 2026-04-23
updated: 2026-04-23
status: ready-to-implement
author: ELO Agent
design_doc: "[[Rank Bands Design]]"
target_completion: 2026-05-02
dss_deadline: 2026-05-16
estimated_hours: 5.5
---

# Rank Bands — Implementation Plan

> Design is approved. Thresholds: Gold ≥1500, Silver 1350–1499, Bronze 1200–1349, Red 1050–1199, Blue 900–1049, Green <900. This plan converts the design into an executable ~5.5-hour session. Zero design decisions left open. All code is copy-paste ready.

---

## Pre-Flight Checklist

Run these before writing a single line of feature code. Total time: ~20 minutes.

**1. Confirm `rankings` table columns:**
```sql
\d rankings
```
Expected columns: `team_id`, `rating`, `rank`, `age_group`, `gender`, `last_game_date`. If any are named differently, adjust the view SQL in Phase 1 Step 1 before running it.

**2. Confirm `teams` table has a name column:**
```sql
\d teams
```
The `rankings_with_bands` view joins to `games` for team names (since the design doc notes the teams table column name may differ). Confirm which column holds the display name: `team_name`, `name`, or other.

**3. Run the distribution query — confirm median is near ~1050:**
```sql
SELECT
  age_group,
  gender,
  COUNT(*) AS team_count,
  MIN(rating)::int AS min_rating,
  MAX(rating)::int AS max_rating,
  PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY rating)::int AS p25,
  PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY rating)::int AS p50_median,
  PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY rating)::int AS p75,
  PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY rating)::int AS p90,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY rating)::int AS p95
FROM rankings
GROUP BY age_group, gender
ORDER BY age_group, gender;
```
If the median across age groups is ≥ 975 and ≤ 1125 — proceed with thresholds as designed. If it deviates more than ±75 from 1050, see Threshold Contingency section before continuing.

**4. Run the per-band distribution check:**
```sql
SELECT
  CASE
    WHEN rating >= 1500 THEN 'Gold'
    WHEN rating >= 1350 THEN 'Silver'
    WHEN rating >= 1200 THEN 'Bronze'
    WHEN rating >= 1050 THEN 'Red'
    WHEN rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS band,
  COUNT(*) AS team_count,
  ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (), 1) AS pct
FROM rankings
GROUP BY band
ORDER BY MIN(rating) DESC;
```
Expected: Gold ~1%, Silver ~4%, Bronze ~20%, Red ~25%, Blue ~25%, Green ~25%. If distribution is wildly off (e.g., Gold is 15% of teams), thresholds need recalibration — see Threshold Contingency.

**5. Confirm the existing `/api/rankings` endpoint exists:**
```bash
curl "http://localhost:3000/api/rankings?age_group=U13G&limit=3"
```
If this returns JSON with team/rating/rank fields — the API exists. If 404 — locate the rankings query in `server.js` and identify the actual endpoint path before proceeding to Phase 2.

---

## Phase 1 — Database (~1 hour)

### Step 1: Create the `rankings_with_bands` view

```sql
-- Run in psql against youth_soccer
CREATE OR REPLACE VIEW rankings_with_bands AS
SELECT
  r.team_id,
  r.age_group,
  r.gender,
  r.rating,
  r.rank,
  -- Use most recent home_team_name from games as display name
  -- (adjust if teams table has a reliable team_name column)
  t.team_display_name AS team_name,
  CASE
    WHEN r.rating >= 1500 THEN 'Gold'
    WHEN r.rating >= 1350 THEN 'Silver'
    WHEN r.rating >= 1200 THEN 'Bronze'
    WHEN r.rating >= 1050 THEN 'Red'
    WHEN r.rating >= 900  THEN 'Blue'
    ELSE                       'Green'
  END AS rank_band,
  CASE
    WHEN r.rating >= 1500 THEN '#FFD700'
    WHEN r.rating >= 1350 THEN '#C0C0C0'
    WHEN r.rating >= 1200 THEN '#CD7F32'
    WHEN r.rating >= 1050 THEN '#DC2626'
    WHEN r.rating >= 900  THEN '#2563EB'
    ELSE                       '#16A34A'
  END AS band_color,
  -- Within-band rank for Gold, Silver, Bronze only
  CASE
    WHEN r.rating >= 1200 THEN
      RANK() OVER (
        PARTITION BY r.age_group, r.gender,
          CASE
            WHEN r.rating >= 1500 THEN 'Gold'
            WHEN r.rating >= 1350 THEN 'Silver'
            ELSE 'Bronze'
          END
        ORDER BY r.rating DESC
      )
    ELSE NULL
  END AS band_rank,
  r.last_game_date,
  (r.last_game_date > NOW() - INTERVAL '7 months') AS is_active
FROM rankings r
LEFT JOIN (
  -- Get most recent home_team_name for each team_id from games
  SELECT DISTINCT ON (home_team_id)
    home_team_id,
    home_team_name AS team_display_name
  FROM games
  WHERE is_hidden = false
  ORDER BY home_team_id, match_date DESC
) t ON t.home_team_id = r.team_id;
```

> **Note on team name source:** If the `teams` table has a reliable `team_name` column, replace the subquery with `JOIN teams t ON t.team_id = r.team_id` and use `t.team_name`. Confirm with `\d teams` in the pre-flight step. Using the games table as the name source is the safe fallback.

### Step 2: Verify the view

```sql
SELECT team_id, rating, rank_band, band_color, band_rank, is_active
FROM rankings_with_bands
LIMIT 20;
```

### Step 3: Spot-check band distribution

```sql
SELECT rank_band, COUNT(*) AS count
FROM rankings_with_bands
GROUP BY rank_band
ORDER BY MIN(rating) DESC;
```
Confirm at least some rows in each band. If Green is empty, the threshold may be too low — check pre-flight Step 3.

### Step 4: Confirm inactive teams show correctly

```sql
SELECT team_id, rating, rank_band, is_active, last_game_date
FROM rankings_with_bands
WHERE last_game_date < NOW() - INTERVAL '7 months'
LIMIT 5;
```
These rows should have `is_active = false`. The API will filter them out in active rankings display.

---

## Phase 2 — API (~1.5 hours)

The server is a plain Node.js HTTP server (`server.js`, port 3000, no Express). Find the section handling `/api/rankings` and replace the `rankings` table reference with `rankings_with_bands`, adding the new fields and the optional `band` filter.

### Step 5: Update the rankings endpoint

Locate the handler for `GET /api/rankings` in `server.js`. Replace the existing query with this:

```javascript
// In server.js — rankings endpoint handler
// (find the existing if/else block for the rankings path)

const url = new URL(req.url, `http://${req.headers.host}`);
const params = url.searchParams;

const ageGroup  = params.get('age_group');
const gender    = params.get('gender');
const band      = params.get('band');     // NEW
const limit     = parseInt(params.get('limit') || '100', 10);
const offset    = parseInt(params.get('offset') || '0', 10);

const conditions = ['is_active = true'];   // always exclude inactive
const values = [];
let idx = 1;

if (ageGroup) {
  conditions.push(`age_group = $${idx++}`);
  values.push(ageGroup);
}
if (gender) {
  conditions.push(`gender = $${idx++}`);
  values.push(gender);
}
if (band) {
  conditions.push(`rank_band = $${idx++}`);   // NEW band filter
  values.push(band);
}

values.push(limit, offset);

const where = conditions.length ? `WHERE ${conditions.join(' AND ')}` : '';

const query = `
  SELECT
    team_id,
    team_name,
    age_group,
    gender,
    rating,
    rank,
    rank_band      AS band,
    band_color,
    band_rank,
    is_active,
    last_game_date
  FROM rankings_with_bands
  ${where}
  ORDER BY rating DESC
  LIMIT $${idx++} OFFSET $${idx}
`;

const result = await db.query(query, values);
res.writeHead(200, { 'Content-Type': 'application/json' });
res.end(JSON.stringify(result.rows));
```

### Step 6: Test the updated endpoint

```bash
# Confirm band and band_color appear in response
curl "http://localhost:3000/api/rankings?age_group=U13G&limit=5"

# Confirm band filter works
curl "http://localhost:3000/api/rankings?band=Gold&age_group=U13G"

# Confirm inactive teams are not returned
curl "http://localhost:3000/api/rankings?age_group=U13G&limit=100" \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print(all(t['is_active'] for t in d))"
# Should print: True
```

**Expected response shape:**
```json
[
  {
    "team_id": "12345",
    "team_name": "FC Dallas 2011G",
    "age_group": "U13",
    "gender": "F",
    "rating": 1387,
    "rank": 142,
    "band": "Silver",
    "band_color": "#C0C0C0",
    "band_rank": 37,
    "is_active": true,
    "last_game_date": "2026-03-15"
  }
]
```

---

## Phase 3 — Frontend (~3 hours)

Frontend is React + Vite in `frontend-v2/`. Find the rankings display component (likely in `frontend-v2/src/components/` or `frontend-v2/src/pages/`).

### Step 7: Create the `BandBadge` component

Create `frontend-v2/src/components/BandBadge.jsx`:

```jsx
export default function BandBadge({ band, color, bandRank }) {
  if (!band) return null;

  const label = bandRank ? `${band} #${bandRank}` : band;

  // Silver needs dark text for contrast on light background
  const textColor = band === 'Silver' ? '#1f2937' : '#ffffff';

  return (
    <span
      className="inline-flex items-center px-2 py-0.5 rounded text-xs font-semibold"
      style={{ backgroundColor: color, color: textColor }}
    >
      {label}
    </span>
  );
}
```

### Step 8: Add `BandBadge` to the rankings table row

Find the component that renders a single row in the rankings table. It likely already renders `team_name`, `rating`, and `rank`. Add the badge:

```jsx
// In whatever component renders a rankings row — e.g. RankingRow.jsx
import BandBadge from './BandBadge';

// Inside the row JSX, next to the team name:
<td className="px-4 py-2">
  <div className="flex items-center gap-2">
    <span>{team.team_name}</span>
    <BandBadge
      band={team.band}
      color={team.band_color}
      bandRank={team.band_rank}
    />
  </div>
</td>
```

### Step 9: Add the band filter UI

Find the rankings filter bar (probably near the age group dropdown). Add a band dropdown:

```jsx
// In the rankings filter/control component
const BANDS = ['All', 'Gold', 'Silver', 'Bronze', 'Red', 'Blue', 'Green'];

// State
const [selectedBand, setSelectedBand] = useState('All');

// Filter UI
<select
  value={selectedBand}
  onChange={e => setSelectedBand(e.target.value)}
  className="border rounded px-2 py-1 text-sm"
>
  {BANDS.map(b => (
    <option key={b} value={b}>{b}</option>
  ))}
</select>

// Pass to API call — when selectedBand !== 'All', append ?band=Gold etc.
const bandParam = selectedBand !== 'All' ? `&band=${selectedBand}` : '';
const url = `/api/rankings?age_group=${ageGroup}${bandParam}`;
```

### Step 10: Test the frontend

1. Start dev server: `cd frontend-v2 && npm run dev`
2. Load the rankings page
3. Confirm badge pills render in correct colors next to team names
4. Confirm Gold/Silver badges show `Gold #N` (within-band rank)
5. Select "Gold" from the band filter — confirm the list filters to only Gold teams
6. Select "Green" — confirm lower-rated teams appear
7. Verify no existing ranking features are broken (age group filter, sorting, team search if present)

---

## Code Artifacts Summary

All four artifacts are complete above. Quick reference:

| Artifact | Location in Plan |
|----------|-----------------|
| A. `CREATE OR REPLACE VIEW rankings_with_bands` | Phase 1 Step 1 |
| B. Updated API handler (with band + band filter) | Phase 2 Step 5 |
| C. `BandBadge` React component | Phase 3 Step 7 |
| D. Distribution check query (pre-flight + per-band) | Pre-Flight Steps 3 & 4 |

---

## Threshold Contingency

If the pre-flight distribution query shows the overall median significantly differs from ~1050:

**Definition of "significantly different":** Median deviates more than ±75 points from 1050 (i.e., median < 975 or median > 1125).

**Recalibration formula — shift all thresholds by the same delta:**

```
delta = actual_median - 1050

Adjusted thresholds:
  Gold   ≥ 1500 + delta
  Silver ≥ 1350 + delta
  Bronze ≥ 1200 + delta
  Red    ≥ 1050 + delta   (= actual_median)
  Blue   ≥  900 + delta
  Green  < anything below Blue threshold
```

Example: if median is 950, delta = -100:
- Gold ≥ 1400, Silver ≥ 1250, Bronze ≥ 1100, Red ≥ 950, Blue ≥ 800, Green < 800

This keeps the proportional band sizes consistent regardless of absolute rating scale. Update the CASE statements in both the view and the BandBadge logic before proceeding.

Time to apply: < 10 minutes (2 CASE block replacements).

---

## Rollback Plan

**If the view causes query performance issues:**
```sql
DROP VIEW IF EXISTS rankings_with_bands;
```
API reverts to querying `rankings` directly (just remove `band`, `band_color`, `band_rank` from the SELECT and restore the old query).

**To remove band fields from API without breaking frontend:**
Remove `band`, `band_color`, `band_rank` from the SELECT. The frontend `BandBadge` component receives `null` for `band` and renders nothing (the `if (!band) return null` guard handles this).

**To revert frontend to pre-band state:**
Remove the `<BandBadge />` import and usage from the row component. Remove the band filter `<select>`. The `band` field returned by the API is simply ignored.

None of these rollbacks require a DB migration or schema change.

---

## Definition of Done

- [ ] `rankings_with_bands` view exists and returns correct bands for 20 spot-checked teams
- [ ] Pre-flight distribution query confirms median near ~1050
- [ ] API `GET /api/rankings?age_group=U13G` response includes `band` and `band_color`
- [ ] API `GET /api/rankings?band=Gold&age_group=U13G` returns only Gold teams
- [ ] Frontend displays band badges in correct colors (Gold = yellow, Silver = gray, etc.)
- [ ] Gold and Silver badges show within-band rank (`Gold #3`, `Silver #12`)
- [ ] Band filter dropdown works — selecting a band filters the rankings list
- [ ] Inactive teams (last_game_date > 7 months ago) do not appear in rankings
- [ ] No existing functionality broken (age group filter, pagination, team search)

---

## Related Notes

- [[Rank Bands Design]] — source of truth for thresholds, colors, and decisions
- [[Database Schema]] — `rankings` table structure
- [[Server and Frontend]] — server.js and React/Vite frontend details
- [[USA Rank Competitive Intel]] — competitor band system this closes the gap against
