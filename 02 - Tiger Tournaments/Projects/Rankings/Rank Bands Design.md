---
title: Rank Bands Design
aliases: [Rank Bands, Rating Bands, Gold Silver Bronze]
tags: [rankings, rank-bands, elo, design, evo-draw]
created: 2026-04-23
updated: 2026-04-23
status: design-complete
author: ELO Agent
---

# Rank Bands Design

> Competitive response to USA Rank's Gold/Silver/Bronze/Red/Blue/Green band system. Our implementation uses **rating-value thresholds** (Approach B) rather than rank-position bands, grounding each tier in our actual Elo distribution rather than arbitrary team counts.

---

## 1. Rating Distribution Analysis

### Distribution Query

Run this against `youth_soccer` to get per-age-group statistics before finalizing thresholds:

```sql
SELECT
  age_group,
  gender,
  COUNT(*) AS team_count,
  MIN(rating)::int AS min_rating,
  MAX(rating)::int AS max_rating,
  PERCENTILE_CONT(0.10) WITHIN GROUP (ORDER BY rating)::int AS p10,
  PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY rating)::int AS p25,
  PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY rating)::int AS p50_median,
  PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY rating)::int AS p75,
  PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY rating)::int AS p90,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY rating)::int AS p95,
  PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY rating)::int AS p99
FROM rankings
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

### Estimated Distribution (based on Glicko v7 calibration values and league hierarchy)

Our Elo/Glicko system starts teams at a baseline near **1000** and adjusts via K-factor games plus calibration bonuses. League calibration values range from +25 (Pre-ECNL) to +160 (MLS NEXT boys), and Phase 7 SOS bonuses add further separation at the top. Expected distribution across all age groups:

| Statistic | Estimated Rating |
|-----------|-----------------|
| Min       | ~600–700        |
| P10       | ~900            |
| P25       | ~1000           |
| Median    | ~1050           |
| P75       | ~1150           |
| P90       | ~1250           |
| P95       | ~1350           |
| P99       | ~1500           |
| Max       | ~1700+          |

> [!note] Confirm with actual DB run
> Thresholds below are calibrated to these estimates. Presten should run the distribution query above, then validate thresholds against actual percentiles. If median deviates from ~1050, adjust threshold boundaries proportionally.

### Distribution Consistency Across Age Groups

- U17B and U15B likely have the widest rating spread (most games, strongest top teams with MLS NEXT calibration at +160)
- U11G and U12G likely have narrower spread (9v9 format, fewer cross-league games)
- Rating-based bands (Approach B) naturally accommodate this — the same threshold means the same quality regardless of age group, which is the correct semantics

---

## 2. Band Threshold Design

### Approach A: Rank-Position Bands (like USA Rank)

| Band   | Rank Range   | Pros | Cons |
|--------|-------------|------|------|
| Gold   | 1–100       | Easy to explain | "Gold" means different things when there are 200 teams vs 1,200 teams |
| Silver | 101–250     | Self-balancing | Bands shift every ranking run as teams enter/exit |
| Bronze | 251–400     | Familiar to users | U11G with 300 total teams has no Green band |
| Red    | 401–700     |      | Artificially privileges small age groups |
| Blue   | 701–1050    |      |      |
| Green  | 1051+       |      |      |

**Verdict:** Approach A is what USA Rank does. It works for their use case (simple rank count display). We reject it as primary because our multi-age-group system makes fixed rank-count bands inconsistent — "Top 100" in U14G (large pool) is not equivalent to "Top 100" in U11B (small pool).

### Approach B: Rating-Value Bands (Recommended)

Thresholds are absolute Elo values, the same for every age group and gender. A team with a 1500+ rating is Gold regardless of whether they play U13G or U17B.

| Band   | Rating Range | Hex Color   | Rationale |
|--------|-------------|-------------|-----------|
| Gold   | 1500+       | `#FFD700`   | Elite — MLS NEXT / top ECNL, top ~1% of rated teams |
| Silver | 1350–1499   | `#C0C0C0`   | High — ECNL regulars, top ECNL RL, ~p95–p99 |
| Bronze | 1200–1349   | `#CD7F32`   | Competitive — strong ECNL RL, top NPL, ~p75–p95 |
| Red    | 1050–1199   | `#DC2626`   | Average — mid NPL/DPL, ~p50–p75 |
| Blue   | 900–1049    | `#2563EB`   | Below average — Pre-ECNL, lower NPL, ~p25–p50 |
| Green  | <900        | `#16A34A`   | Entry — recreational, minimal ranked games, <p25 |

### Estimated Team Count Per Band (all age groups combined, ~144K teams total, subset ranked)

Assuming ~30,000–50,000 teams have sufficient games to be ranked:

| Band   | Approx % of Ranked | Approx Team Count |
|--------|-------------------|-------------------|
| Gold   | ~1%               | 300–500           |
| Silver | ~4%               | 1,200–2,000       |
| Bronze | ~20%              | 6,000–10,000      |
| Red    | ~25%              | 7,500–12,500      |
| Blue   | ~25%              | 7,500–12,500      |
| Green  | ~25%              | 7,500–12,500      |

> [!note] Run this to validate per-band counts
> ```sql
> SELECT
>   CASE
>     WHEN rating >= 1500 THEN 'Gold'
>     WHEN rating >= 1350 THEN 'Silver'
>     WHEN rating >= 1200 THEN 'Bronze'
>     WHEN rating >= 1050 THEN 'Red'
>     WHEN rating >= 900  THEN 'Blue'
>     ELSE 'Green'
>   END AS band,
>   COUNT(*) AS team_count
> FROM rankings
> GROUP BY band
> ORDER BY MIN(rating) DESC;
> ```

---

## 3. Band Display Design

### Color Palette

| Band   | Hex       | CSS Name Suggestion | Usage |
|--------|-----------|--------------------|----|
| Gold   | `#FFD700` | `band-gold`        | Text + badge background |
| Silver | `#C0C0C0` | `band-silver`      | Text + badge background |
| Bronze | `#CD7F32` | `band-bronze`      | Text + badge background |
| Red    | `#DC2626` | `band-red`         | Text + badge background |
| Blue   | `2563EB`  | `band-blue`        | Text + badge background |
| Green  | `#16A34A` | `band-green`       | Text + badge background |

> [!tip] Evo Draw Brand Integration
> Apply band colors as badge pill components using the existing Tailwind color system. Emerald/cyan gradient (`#34d399` to `#06b6d4`) remains the primary brand color — band colors are accent/indicator only.

### Rank Within Band

Yes — display numeric rank within the band for Gold, Silver, and Bronze. Example: `Gold #47`. This gives elite teams a precise signal without exposing raw rank numbers to all tiers. Red/Blue/Green bands do not need within-band rank (user value is lower there).

### Inactivity Handling

- **7 months** of no games = band is hidden, team shows as "Inactive" (matching USA Rank's threshold)
- Band is still stored in DB; it just does not display in the UI
- The `rankings` table query must filter by `last_game_date > NOW() - INTERVAL '7 months'`

### Unranked Teams

- Teams with fewer than **6 results** against ranked opponents = "Unranked" label, no band badge
- Display: gray pill labeled `Unranked` — not hidden, visible in team search but excluded from band counts
- This matches USA Rank's 6–10 result threshold; we use 6 as the lower bound

---

## 4. Database Implementation

### Band Computation: SQL CASE Statement

```sql
-- Inline band mapping (use anywhere a rating is available)
CASE
  WHEN rating >= 1500 THEN 'Gold'
  WHEN rating >= 1350 THEN 'Silver'
  WHEN rating >= 1200 THEN 'Bronze'
  WHEN rating >= 1050 THEN 'Red'
  WHEN rating >= 900  THEN 'Blue'
  ELSE                     'Green'
END AS rank_band,

CASE
  WHEN rating >= 1500 THEN '#FFD700'
  WHEN rating >= 1350 THEN '#C0C0C0'
  WHEN rating >= 1200 THEN '#CD7F32'
  WHEN rating >= 1050 THEN '#DC2626'
  WHEN rating >= 900  THEN '#2563EB'
  ELSE                     '#16A34A'
END AS band_color
```

### Storage Decision: Computed View (Not Stored Column)

**Recommendation: SQL view, not stored column.**

- Band thresholds may need tuning in the first few months — a view means zero migration cost to adjust
- Ratings are recomputed nightly anyway; band is always fresh from the view
- View query is trivial and adds no meaningful query latency

### `rankings_with_bands` View

```sql
CREATE OR REPLACE VIEW rankings_with_bands AS
SELECT
  r.team_id,
  r.age_group,
  r.gender,
  r.rating,
  r.rank,
  t.home_team_name AS team_name,
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
  -- Within-band rank for Gold/Silver/Bronze
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
  -- Active = game in last 7 months
  (r.last_game_date > NOW() - INTERVAL '7 months') AS is_active
FROM rankings r
LEFT JOIN (
  SELECT DISTINCT ON (home_team_id) home_team_id, home_team_name
  FROM games
  WHERE is_hidden = false
  ORDER BY home_team_id, match_date DESC
) t ON t.home_team_id = r.team_id;
```

> [!warning] teams table name resolution
> The `teams` table stores team_name. If the column name differs from `home_team_name`, adjust accordingly. Presten should verify `teams` schema: `\d teams` in psql.

---

## 5. API Design

### Response Schema Change

**Before (current):**
```json
{
  "team_id": "12345",
  "team_name": "FC Dallas 2011G",
  "rating": 1387,
  "rank": 142,
  "age_group": "U14G",
  "gender": "Girls"
}
```

**After (with bands):**
```json
{
  "team_id": "12345",
  "team_name": "FC Dallas 2011G",
  "rating": 1387,
  "rank": 142,
  "age_group": "U14G",
  "gender": "Girls",
  "band": "Silver",
  "band_color": "#C0C0C0",
  "band_rank": 37,
  "is_active": true
}
```

### Endpoint Additions

**Existing endpoint update:**
```
GET /api/rankings?age_group=U14G
→ Now includes band, band_color, band_rank, is_active in every record
```

**New filter endpoint:**
```
GET /api/rankings?band=Gold&age_group=U14G
→ Returns only teams in the Gold band for U14G, sorted by rating DESC
```

**Implementation note:** Filter on the view `rankings_with_bands WHERE rank_band = $band AND age_group = $age_group AND is_active = true`.

---

## 6. Color Palette Summary

| Band   | Hex       | Tailwind Equivalent | Dark Mode Adjust |
|--------|-----------|--------------------|----|
| Gold   | `#FFD700` | `yellow-400`       | Same |
| Silver | `#C0C0C0` | `gray-300`         | `gray-400` for visibility |
| Bronze | `#CD7F32` | `orange-700`       | Same |
| Red    | `#DC2626` | `red-600`          | Same |
| Blue   | `#2563EB` | `blue-600`         | Same |
| Green  | `#16A34A` | `green-600`        | Same |

---

## 7. Inactivity Handling Rule

| Condition | Behavior |
|-----------|---------|
| Last game < 7 months ago | `is_active = true`, band displays normally |
| Last game ≥ 7 months ago | `is_active = false`, UI shows "Inactive" — band badge hidden |
| Fewer than 6 ranked results | `is_active = null`, UI shows "Unranked" gray pill |
| Team not in rankings table | Not displayed |

---

## 8. Frontend Integration (Presten's Scope)

### React Component Changes

1. **RankingRow component** — add `<BandBadge band={band} color={band_color} bandRank={band_rank} />` pill after team name
2. **BandBadge component** (new, ~20 lines) — renders a colored pill. Props: `band`, `color`, `bandRank`
3. **Rankings filter bar** — add band dropdown: All / Gold / Silver / Bronze / Red / Blue / Green
4. **Team profile page** — display band badge prominently near rating number
5. **API calls** — no URL changes needed; response already includes new fields

### BandBadge component sketch:
```jsx
function BandBadge({ band, color, bandRank }) {
  const label = bandRank ? `${band} #${bandRank}` : band;
  return (
    <span
      className="inline-flex items-center px-2 py-0.5 rounded text-xs font-semibold text-white"
      style={{ backgroundColor: color }}
    >
      {label}
    </span>
  );
}
```

---

## 9. Implementation Order and Timeline

| Step | Task | Owner | Estimate |
|------|------|-------|---------|
| 1 | Run distribution query, validate thresholds | Presten | 30 min |
| 2 | Create `rankings_with_bands` view | Presten | 1 hr |
| 3 | Update `/api/rankings` response to include band fields | Presten | 1 hr |
| 4 | Add `?band=` filter param to API | Presten | 30 min |
| 5 | Build `BandBadge` React component | Presten | 1 hr |
| 6 | Add band filter to rankings UI | Presten | 1 hr |
| 7 | Team profile page badge | Presten | 30 min |
| **Total** | | | **~5.5 hrs** |

**DSS deadline (May 16, 2026):** Achievable in a single focused session. Recommend targeting implementation week of May 5.

---

## Related Notes
- [[Ranking Engine]] — Elo/Glicko v7 that produces the ratings bands are derived from
- [[Database Schema]] — rankings table structure
- [[Server and Frontend]] — API and React integration points
- [[USA Rank Competitive Intel]] — competitor band system reference
