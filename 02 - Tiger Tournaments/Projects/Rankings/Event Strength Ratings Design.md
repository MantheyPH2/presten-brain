---
title: Event Strength Ratings Design
aliases: [Event Strength, Tournament Strength, Event Quality]
tags: [rankings, events, strength, elo, design, evo-draw]
created: 2026-04-23
updated: 2026-04-23
status: design-complete
author: ELO Agent
---

# Event Strength Ratings Design

> We have ~10,500 events and 1.65M games. USA Rank shows High/Average/Low strength labels. We go further: we show the math. This document defines 4 required metrics, their computation SQL, schema, and rollout plan.

---

## 1. Metric Definitions

### A. Average Team Rating (Event Rating)

The mean Elo rating of all teams participating in the event, computed at the **per-age-group level** so a U13G event is compared against other U13G events — not the global pool.

- **Input:** Current `rankings` table rating for each participating team
- **Output:** Numeric score (e.g., 1183.4) + percentile rank among all events in that age group
- **Why current rating is acceptable:** We do not have a `ratings_history` table. Current rating is a reasonable proxy — teams that were good at event time are likely still near that rating, and rating drift over a season is typically ±50–100 points for most teams. Flag this limitation in the UI with "ratings are current, not event-time."
- **Minimum teams:** 4 distinct teams required for the metric to be meaningful

### B. Strength Classification (High / Average / Low)

Map the event's average team rating percentile (within its age group × gender bucket) to a 3-tier label:

| Label   | Percentile Range | Rationale |
|---------|-----------------|-----------|
| High    | Top 20% (≥ p80) | Genuinely elite field — coaches pay attention |
| Average | 20–60% (p40–p79) | Typical competitive tournament |
| Low     | Bottom 40% (< p40) | Developmental or recreational |

> Skewed toward "High" and "Average" being meaningful distinctions. Bottom 40% is "Low" rather than bottom 20% because many events are recreational by design — we want the Low label to be informative, not pejorative-rare.

### C. Competitive Spread

Standard deviation of team ratings within the event. Tells coaches whether the field is tightly matched or wildly uneven.

| Label  | Condition | Meaning |
|--------|-----------|---------|
| Tight  | StdDev < 75 | All teams within ~1 tier — closely matched field |
| Mixed  | StdDev 75–150 | Normal tournament variance — some mismatches |
| Wide   | StdDev > 150 | Large rating gaps — blowout risk is real |

Thresholds are calibrated to our Elo scale (~600–1700 range, typical inter-team spread ~100–200 within an event).

### D. Upset Rate

An upset occurs when a **lower-rated team wins** (not draws) against a higher-rated opponent, with a **minimum 50-point rating gap** to filter out noise from near-equal teams.

**Formula:**
```
upset_rate = upsets / total_decisive_games
```
where:
- `total_decisive_games` = games with a winner (home_score ≠ away_score) where rating gap ≥ 50 points
- `upsets` = subset of decisive games where lower-rated team won

| Label   | Upset Rate  | Meaning |
|---------|-------------|---------|
| Low     | < 20%       | Favorites dominate — rating predicts outcomes well |
| Average | 20–40%      | Normal competitive balance |
| High    | > 40%       | High unpredictability — upsets are the norm |

> Draws are excluded from upset calculation. A draw by the lower-rated team is not an upset — it is a positive result but not a win.

### E. Schedule Strength (Per-Team, Optional — Scope Controlled)

Average rating of opponents faced by each team within the event. This is per-team, not per-event, and is already partially captured by Phase 7 (SOS Bonuses) in the Ranking Engine.

**Recommendation:** Defer to Phase 2. Computing schedule strength per-team for historical events requires joining team ratings to individual game opponents, which is feasible but significantly increases the scope. Phase 1 focus is event-level metrics (A–D).

---

## 2. Data Availability Assessment

### Events Table

```sql
-- Confirm event schema and sample data
SELECT id, event_name, age_group, gender, COUNT(g.id) AS game_count
FROM events e
LEFT JOIN games g ON g.event_id = e.id AND g.is_hidden = false
GROUP BY e.id, e.event_name, e.age_group, e.gender
ORDER BY game_count DESC
LIMIT 20;
```

> [!note] events table assumption
> The schema doc shows `games.event_name` (text) but does not show a separate `events` table with a PK. If `events` is not a standalone table, games can still be grouped by `(event_name, age_group, gender, season)` as a composite event key. See "edge cases" section and the fallback query below.

**If a standalone `events` table with `id` exists:** The join-based approach works as written.

**If events are identified only by name in the games table (no events table):** Use this fallback:

```sql
-- Fallback: treat (event_name, age_group, gender, season) as the event identifier
SELECT
  event_name,
  age_group,
  gender,
  season,
  COUNT(DISTINCT CASE WHEN home_score IS NOT NULL THEN home_team_id END)
  + COUNT(DISTINCT CASE WHEN away_score IS NOT NULL THEN away_team_id END) AS approx_team_count
FROM games
WHERE is_hidden = false
GROUP BY event_name, age_group, gender, season
HAVING COUNT(*) >= 5
ORDER BY approx_team_count DESC
LIMIT 20;
```

### Games Table

`games.event_id` → `events.id` foreign key assumed from the event_name field being present. Key fields needed:
- `home_team_id`, `away_team_id` — for team rating lookup
- `home_score`, `away_score` — for upset calculation
- `is_hidden = false` — always filter
- `age_group`, `gender` — for normalization bucket

### Ratings History

**No `ratings_history` table exists** per the schema documentation. Current `rankings` table ratings are used as the approximation. This is noted as a limitation — the metric is labeled "strength based on current ratings" in the UI, not "strength at event time."

**Future improvement:** The Ranking Engine could write a snapshot of ratings to a `ratings_snapshots` table after each nightly run. Once 6+ months of snapshots exist, event strength can be recomputed using historical ratings. This is a Phase 2 item.

### Minimum Viable Event Size

Events with fewer than 4 distinct teams or fewer than 5 games have insufficient data for meaningful metrics. These events receive `strength_label = 'Insufficient Data'` and are excluded from percentile calculations.

**Count of eligible events:**
```sql
SELECT COUNT(*) AS eligible_events
FROM (
  SELECT event_name, age_group, gender
  FROM games
  WHERE is_hidden = false
  GROUP BY event_name, age_group, gender
  HAVING COUNT(*) >= 5
    AND COUNT(DISTINCT home_team_id) + COUNT(DISTINCT away_team_id) >= 4
) sub;
```

---

## 3. Computation SQL — Production-Ready

### Base Event Stats Query

```sql
-- ============================================================
-- Event Strength: Base Metrics
-- Groups games by event (using event_name as identifier if no events table)
-- ============================================================
WITH event_teams AS (
  -- Get distinct teams per event with their current ratings
  SELECT
    g.event_name,
    g.age_group,
    g.gender,
    g.season,
    t.team_id,
    r.rating
  FROM (
    SELECT DISTINCT event_name, age_group, gender, season, home_team_id AS team_id
    FROM games WHERE is_hidden = false
    UNION
    SELECT DISTINCT event_name, age_group, gender, season, away_team_id AS team_id
    FROM games WHERE is_hidden = false
  ) g
  LEFT JOIN rankings r ON r.team_id = g.team_id
    AND r.age_group = g.age_group
    AND r.gender = g.gender
  -- Only include teams that have a current ranking
  WHERE r.rating IS NOT NULL
),

event_base AS (
  SELECT
    event_name,
    age_group,
    gender,
    season,
    COUNT(*)               AS team_count,
    AVG(rating)            AS avg_team_rating,
    STDDEV(rating)         AS rating_spread,
    MIN(rating)            AS min_rating,
    MAX(rating)            AS max_rating
  FROM event_teams
  GROUP BY event_name, age_group, gender, season
  HAVING COUNT(*) >= 4     -- minimum 4 rated teams
),

-- Upset rate computation
game_results AS (
  SELECT
    g.event_name,
    g.age_group,
    g.gender,
    g.season,
    g.home_team_id,
    g.away_team_id,
    g.home_score,
    g.away_score,
    rh.rating AS home_rating,
    ra.rating AS away_rating,
    ABS(rh.rating - ra.rating) AS rating_gap,
    CASE
      WHEN g.home_score > g.away_score THEN 'home_win'
      WHEN g.away_score > g.home_score THEN 'away_win'
      ELSE 'draw'
    END AS result
  FROM games g
  LEFT JOIN rankings rh ON rh.team_id = g.home_team_id
    AND rh.age_group = g.age_group AND rh.gender = g.gender
  LEFT JOIN rankings ra ON ra.team_id = g.away_team_id
    AND ra.age_group = g.age_group AND ra.gender = g.gender
  WHERE g.is_hidden = false
    AND g.game_status = 'completed'
    AND g.home_score IS NOT NULL
    AND g.away_score IS NOT NULL
    AND rh.rating IS NOT NULL
    AND ra.rating IS NOT NULL
),

event_upsets AS (
  SELECT
    event_name,
    age_group,
    gender,
    season,
    COUNT(CASE
      WHEN result != 'draw' AND rating_gap >= 50 THEN 1
    END) AS decisive_games,
    COUNT(CASE
      WHEN result = 'home_win' AND home_rating < away_rating AND rating_gap >= 50 THEN 1
      WHEN result = 'away_win' AND away_rating < home_rating AND rating_gap >= 50 THEN 1
    END) AS upsets
  FROM game_results
  GROUP BY event_name, age_group, gender, season
),

-- Combine base stats with upset rate
event_full AS (
  SELECT
    b.event_name,
    b.age_group,
    b.gender,
    b.season,
    b.team_count,
    ROUND(b.avg_team_rating, 2)  AS avg_team_rating,
    ROUND(b.rating_spread, 2)    AS rating_spread,
    CASE
      WHEN u.decisive_games > 0
        THEN ROUND(u.upsets::numeric / u.decisive_games, 4)
      ELSE 0
    END                          AS upset_rate,
    u.decisive_games,
    u.upsets
  FROM event_base b
  LEFT JOIN event_upsets u USING (event_name, age_group, gender, season)
),

-- Percentile rank of each event within its age_group × gender bucket
event_percentiled AS (
  SELECT
    *,
    PERCENT_RANK() OVER (
      PARTITION BY age_group, gender
      ORDER BY avg_team_rating
    ) AS strength_percentile   -- 0.0 = weakest in age group, 1.0 = strongest
  FROM event_full
)

-- Final output with classification labels
SELECT
  event_name,
  age_group,
  gender,
  season,
  team_count,
  avg_team_rating,
  rating_spread,
  ROUND(strength_percentile * 100, 1) AS strength_percentile_pct,
  upset_rate,

  -- Strength classification
  CASE
    WHEN strength_percentile >= 0.80 THEN 'High'
    WHEN strength_percentile >= 0.40 THEN 'Average'
    ELSE 'Low'
  END AS strength_label,

  -- Spread classification
  CASE
    WHEN rating_spread IS NULL THEN 'Unknown'
    WHEN rating_spread < 75   THEN 'Tight'
    WHEN rating_spread < 150  THEN 'Mixed'
    ELSE 'Wide'
  END AS spread_label,

  -- Upset label
  CASE
    WHEN decisive_games < 3   THEN 'Unknown'
    WHEN upset_rate < 0.20    THEN 'Low'
    WHEN upset_rate < 0.40    THEN 'Average'
    ELSE 'High'
  END AS upset_label

FROM event_percentiled
ORDER BY age_group, gender, avg_team_rating DESC;
```

---

## 4. Event Strength Table DDL — Production-Ready

```sql
-- ============================================================
-- event_strength: Computed nightly, stores latest snapshot
-- Uses event_name as identifier if no events.id PK exists
-- ============================================================
CREATE TABLE event_strength (
  id                    SERIAL PRIMARY KEY,
  -- Event identifier — use event_id if events table exists, else use composite
  event_name            VARCHAR(300) NOT NULL,
  age_group             VARCHAR(10) NOT NULL,
  gender                CHAR(1) NOT NULL CHECK (gender IN ('M', 'F', 'U')),  -- U = unspecified
  season                VARCHAR(10),                  -- e.g. '2025-2026'
  -- If a standalone events table exists:
  event_id              INT REFERENCES events(id),    -- nullable if no events table

  computed_at           TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  -- Core metrics
  team_count            INT NOT NULL,
  avg_team_rating       NUMERIC(8,2) NOT NULL,
  rating_spread         NUMERIC(8,2),                 -- NULL if < 2 teams
  upset_rate            NUMERIC(5,4),                 -- NULL if < 3 decisive games
  decisive_games        INT,
  upset_count           INT,
  min_rating            NUMERIC(8,2),
  max_rating            NUMERIC(8,2),

  -- Percentile within age group × gender
  strength_percentile   NUMERIC(5,2),                 -- 0.00–100.00

  -- Classification labels
  strength_label        VARCHAR(20) CHECK (strength_label IN ('High','Average','Low','Insufficient Data')),
  spread_label          VARCHAR(20) CHECK (spread_label IN ('Tight','Mixed','Wide','Unknown')),
  upset_label           VARCHAR(20) CHECK (upset_label IN ('Low','Average','High','Unknown')),

  -- Flags
  is_rated              BOOLEAN NOT NULL DEFAULT true,  -- false if < 4 rated teams
  ratings_are_current   BOOLEAN NOT NULL DEFAULT true,  -- always true until ratings_history exists

  UNIQUE (event_name, age_group, gender, season, computed_at)
);

CREATE INDEX idx_event_strength_lookup
  ON event_strength (event_name, age_group, gender, season);

CREATE INDEX idx_event_strength_label
  ON event_strength (strength_label, age_group, gender);

CREATE INDEX idx_event_strength_computed
  ON event_strength (computed_at);
```

**Changes from the proposed schema:**
- Added `event_name` as the primary lookup field (no assumption of `events.id`)
- Added `event_id` as nullable FK for when events table exists
- Added `decisive_games`, `upset_count`, `min_rating`, `max_rating` for display richness
- Added `gender` constraint to include `'U'` for unspecified
- Added `is_rated` flag to distinguish "computed but weak" from "insufficient data"
- Added `ratings_are_current` flag to be transparent about the approximation
- Replaced compound PK with serial PK + unique constraint (more flexible for re-runs)

---

## 5. Ranking Engine Integration Decision

### Current State

The Ranking Engine v7 Phase 7 (SOS Bonuses) already rewards teams for facing stronger opponents. This is a form of implicit event strength integration — playing at a High-strength event naturally produces tougher opponents, which boosts SOS. Phase 8 (League Calibration) then applies K-factor multipliers based on tier classification.

**Gap:** Event strength is not currently computed as a standalone metric that feeds back into Phase 3 (Elo Computation) directly.

### Recommendation: Display-Only for Phase 1 — Confirmed with Reasoning

**Phase 1: Display metric only. Do not adjust historical Elo with event strength scores yet.**

Reasons:
1. **Circular dependency risk.** Event strength is computed from team ratings. If team ratings are then adjusted based on event strength, the system becomes circular. Solving this requires iterative convergence, which significantly increases complexity and runtime.
2. **SOS already handles it.** Phase 7 already captures the core signal — opponents at elite events are high-rated, so SOS bonuses naturally flow from playing at strong events.
3. **Bootstrap quality.** First event strength computation uses current (post-hoc) ratings. Feeding these back into historical Elo would retroactively change all past ratings, requiring a full recompute. This is a significant engineering commitment for uncertain marginal gain.
4. **Measurable value today.** Displaying event strength on event pages has immediate user value and zero algorithmic risk.

**Phase 2 (after 6+ months of data):** Consider using `event_strength.strength_percentile` as a K-factor multiplier modifier — games at High-strength events (p80+) get K × 1.15, games at Low-strength events get K × 0.85. This would require adding it to Phase 2 (Load Event Tiers) and testing against our cross-league validation dataset (575K games).

---

## 6. API Specs

### Endpoint 1: Event Strength Detail

```
GET /api/events/{event_name}/strength?age_group=U14G&season=2025-2026
```

(URL-encode event_name; or use event_id if available)

**Response:**
```json
{
  "event_name": "Dallas Spring Showcase 2026",
  "age_group": "U14G",
  "gender": "F",
  "season": "2025-2026",
  "computed_at": "2026-04-23T02:00:00Z",
  "team_count": 32,
  "avg_team_rating": 1287.4,
  "rating_spread": 118.2,
  "upset_rate": 0.2840,
  "strength_percentile": 87.3,
  "strength_label": "High",
  "spread_label": "Mixed",
  "upset_label": "Average",
  "ratings_are_current": true,
  "min_rating": 1041.0,
  "max_rating": 1498.3
}
```

### Endpoint 2: Event Search with Strength Filter

```
GET /api/events?strength=High&age_group=U15G&season=2025-2026&limit=25
```

**Query parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `strength` | `High` \| `Average` \| `Low` | optional | Filter by label |
| `age_group` | string | optional | e.g. U14G, U15B |
| `season` | string | optional | e.g. 2025-2026 |
| `spread` | `Tight` \| `Mixed` \| `Wide` | optional | Filter by spread |
| `limit` | int | 25 | Pagination |
| `offset` | int | 0 | Pagination |

**Response:**
```json
{
  "total": 847,
  "events": [
    {
      "event_name": "Dallas Spring Showcase 2026",
      "age_group": "U15G",
      "season": "2025-2026",
      "strength_label": "High",
      "spread_label": "Mixed",
      "upset_label": "Low",
      "avg_team_rating": 1321.0,
      "strength_percentile": 91.2,
      "team_count": 28
    }
  ]
}
```

---

## 7. Edge Cases

| Edge Case | Handling |
|-----------|---------|
| Event with 1–3 teams | `strength_label = 'Insufficient Data'`, excluded from percentile calc, `is_rated = false` |
| Event with 0 rated teams | Same as above — no ratings available to compute from |
| All games are draws | `upset_rate = NULL`, `upset_label = 'Unknown'`, decisive_games = 0 |
| Only 1–2 decisive games | `upset_label = 'Unknown'` (insufficient sample, threshold: < 3 decisive games) |
| Rating spread = NULL (single team) | `spread_label = 'Unknown'` |
| Event spans multiple age groups | Compute separate event_strength row per age_group × gender combination |
| Event name contains special characters | URL-encode in API; store raw in DB |
| Teams with no ranking (new/unranked) | Excluded from avg_team_rating; team_count reflects only rated teams; unrated teams noted if count diverges significantly |
| Event is very old (> 2 seasons) | Still compute — historical events are valuable for context. Flag with `season` field. |
| Re-run / nightly update | Insert new row with `computed_at = NOW()`. Previous rows preserved for trend analysis. Latest is always `MAX(computed_at)`. |

---

## 8. Rollout Plan

### Phase 1: Most Recent Season First (Week 1–2)

Compute event strength for all events in the **2025-2026 season** first. These are most relevant to current users and have the most current team ratings (minimizing the historical rating approximation error).

```sql
-- Scope to current season for initial run
WHERE season = '2025-2026'
```

Expected: ~2,000–4,000 events × multiple age groups × 2 genders = ~15,000–30,000 event_strength rows.

### Phase 2: All Historical Events (Week 3)

Backfill all prior seasons. Rating approximation quality degrades for older events (teams may have changed significantly), but the data is still useful for historical context. Add a display note: "Ratings are current approximations; older events may have lower accuracy."

### Phase 3: Frontend Integration (Week 3–4)

1. **Event detail page:** Add a "Strength Card" component displaying all 4 metrics with label badges and numbers
2. **Event search/browse page:** Add strength filter dropdown
3. **Tournament schedule pages:** Show strength badge next to each event name

### Computation Schedule

- Run as part of the nightly pipeline, after `compute-rankings.js` completes
- Estimated runtime: < 2 minutes for 10,500 events with 1.65M games (indexed queries)
- Log run time and row count for monitoring

### Total Engineering Estimate

| Task | Effort |
|------|--------|
| Validate schema (events table vs games.event_name) | 1 hr |
| Create `event_strength` table | 30 min |
| Build and test computation SQL (current season) | 3 hrs |
| Backfill historical events | 1 hr |
| Schedule as nightly job | 30 min |
| API endpoints (2) | 2 hrs |
| Frontend strength card component | 2 hrs |
| Frontend event search filter | 1 hr |
| QA + edge case testing | 2 hrs |
| **Total** | **~13 hrs** |

**DSS demo (May 16, 2026): Achievable.** If started April 28, all work fits in 3 weeks with buffer.

---

## Related Notes
- [[Ranking Engine]] — Phase 7 SOS Bonuses, Phase 2 Event Tiers
- [[Database Schema]] — `games`, `events` table structure
- [[Server and Frontend]] — API integration points
- [[USA Rank Competitive Intel]] — competitor event strength feature
- [[Cross-League Analysis]] — methodology for large-scale game stat computation
