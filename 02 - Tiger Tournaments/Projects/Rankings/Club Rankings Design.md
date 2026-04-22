---
title: Club Rankings Design
aliases: [Club Rankings, Club Ratings, Club Elo]
tags: [rankings, clubs, elo, design, evo-draw]
created: 2026-04-23
updated: 2026-04-23
status: design-complete
author: ELO Agent
---

# Club Rankings Design

> Competitive response to USA Rank's club ranking feature. We have 144,676 teams with club affiliation data embedded in team names. This design establishes the canonical club identity model, ranking methodology, database schema, and computation logic.

---

## 1. Club Identity Resolution

### Current State of Club Data

The `teams` table (144,676 rows) has a `club` field and club names are also embedded in team name strings (e.g., "FC Dallas 2011G ECNL", "LAFC Nationally 2010B"). GotSport is our Priority 1 data source and does include organizational data in its structured API response — the teams table likely has a `club` column populated from GotSport org data for Priority 1 records.

**Diagnostic query — run first:**
```sql
SELECT
  club,
  COUNT(*) AS team_count
FROM teams
WHERE club IS NOT NULL AND club != ''
GROUP BY club
ORDER BY team_count DESC
LIMIT 50;
```

And check coverage:
```sql
SELECT
  COUNT(*) AS total_teams,
  COUNT(CASE WHEN club IS NOT NULL AND club != '' THEN 1 END) AS with_club,
  ROUND(
    COUNT(CASE WHEN club IS NOT NULL AND club != '' THEN 1 END) * 100.0 / COUNT(*), 1
  ) AS pct_with_club
FROM teams;
```

### The Three Options

| Option | Description | Verdict |
|--------|-------------|---------|
| A | Parse club name from team name string (NLP/fuzzy) | Fragile, expensive, last resort |
| B | Use GotSport org_id if stored in teams table | Best if available — high precision |
| C | New `clubs` table + `team_club_assignments` junction | Most robust long-term, needed regardless |

### Chosen Approach: B + C (Hybrid)

1. **Primary:** Use GotSport org_id from the `teams` table as the club identifier for all GotSport-sourced teams (Priority 1–2, which account for ~1.49M of 1.65M games)
2. **Fallback:** For non-GotSport teams, parse the `club` column or team name prefix
3. **Canonical store:** Create a `clubs` table to normalize and deduplicate club identities (e.g., "FC Dallas", "FC Dallas Academy", "FCD" → single canonical club)

### Name Normalization for Fuzzy Matching

Club names need normalization before matching. Apply:
```
lowercase → strip punctuation → remove common suffixes (FC, SC, Soccer Club, Academy, United)
→ trim whitespace
```

Example: "FC Dallas Academy" → "dallas" // "FC Dallas" → "dallas" — these merge.

> [!warning] Over-merging risk
> "Dallas Texans" and "Dallas Longhorns" both normalize to "dallas ..." but are different clubs. The normalized name should strip only generic suffixes, not the full name. Use GotSport org_id as the primary key to avoid this.

---

## 2. Coverage Estimate

### Estimated Coverage by Data Source

| Source | Teams Approx | Club Coverage Estimate |
|--------|-------------|----------------------|
| GotSport API (pri 1) | ~100,000 | ~85–95% (structured org data) |
| GotSport HTML (pri 2) | ~20,000 | ~60–70% (less structured) |
| TGS/AthleteOne (pri 3) | ~12,000 | ~40% (club in team name) |
| SincSports (pri 5) | ~8,000 | ~30% (team name only) |
| Others | ~5,000 | ~20% |
| **Total** | **~145,000** | **~70–80% estimated** |

**Target minimum:** 70% coverage (>100K teams mappable to a club) before launching club rankings. If GotSport org_id is populated, we likely exceed this on day one.

---

## 3. Ranking Methodology

### Eligible Age Groups

**U11, U12, U13, U14, U15, U16, U17** — 7 eligible age groups (matching USA Rank's U11–U17 scope). U18/U19 excluded (college transition makes club affiliation unstable at those ages).

Format note: U11/U12 are 9v9, U13–U17 are 11v11. Both are included — the Elo system handles format differences via calibration.

### Gender Handling

**Separate club rankings for Boys and Girls.** Rationale:
- Many clubs are gender-specific (LAFC = boys, many ECNL clubs = girls-only)
- A combined ranking would unfairly favor clubs with both strong boys and girls programs
- Coaches/parents search by gender — separate tables produce more useful results

### Top Team Selection Per Age Group

For each club × age group × gender:
- Take the **single highest-rated** team
- If a club has multiple U14G teams (e.g., "FC Dallas 2011G ECNL" and "FC Dallas 2011G NPL"), only the top one counts
- This rewards club investment in elite team development, not just having many teams

### Minimum Eligibility Threshold

- **5 of 7 eligible age groups** must have at least one ranked team (matching USA Rank)
- A team is "ranked" if it appears in the `rankings` table with ≥ 6 results against ranked opponents
- Inactive teams (no game in 7 months) do NOT count toward the minimum threshold

### Club Rating Formula

```
club_rating = AVG(top_team_rating per eligible age group)
```

Where eligible = age group has a ranked, active team for this club.

Example:
- FC Dallas Girls has ranked teams in U13G (1420), U14G (1510), U15G (1380), U16G (1450), U17G (1290)
- 5 of 7 eligible → qualifies
- club_rating = (1420 + 1510 + 1380 + 1450 + 1290) / 5 = **1410.0**

### Activity Requirement

A team must have played a game in the last **7 months** to count toward club ranking. This prevents clubs from gaming the system by counting stale elite teams.

---

## 4. SQL Schema — Production-Ready DDL

```sql
-- Canonical club registry
CREATE TABLE clubs (
  id                  SERIAL PRIMARY KEY,
  name                VARCHAR(200) NOT NULL,
  normalized_name     VARCHAR(200) NOT NULL,    -- lowercase, stripped for dedup matching
  state               CHAR(2),                  -- primary state (TX, CA, etc.)
  gotsport_org_id     VARCHAR(50),              -- GotSport org identifier (primary key for dedup)
  website             VARCHAR(300),
  is_verified         BOOLEAN DEFAULT false,    -- manually verified club identity
  created_at          TIMESTAMPTZ DEFAULT NOW(),
  updated_at          TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE (gotsport_org_id),
  UNIQUE (normalized_name, state)               -- dedup within state
);

-- Many-to-one: team → club assignment
CREATE TABLE team_club_assignments (
  team_id             VARCHAR(100) NOT NULL,    -- matches teams.team_id
  club_id             INT NOT NULL REFERENCES clubs(id) ON DELETE CASCADE,
  assignment_method   VARCHAR(20) NOT NULL,     -- 'gotsport_org_id', 'name_parse', 'manual'
  confidence          NUMERIC(3,2),             -- 0.00–1.00; 1.00 = verified
  assigned_at         TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (team_id)
);

-- Club ranking output (computed nightly)
CREATE TABLE club_rankings (
  club_id             INT NOT NULL REFERENCES clubs(id) ON DELETE CASCADE,
  gender              CHAR(1) NOT NULL CHECK (gender IN ('M', 'F')),
  computed_at         TIMESTAMPTZ NOT NULL,
  club_rating         NUMERIC(8,2) NOT NULL,
  club_rank           INT,                      -- rank within gender (1 = best)
  age_groups_counted  INT NOT NULL,             -- how many age groups contributed
  age_groups_eligible JSONB,                    -- which age groups qualified, e.g. ["U13","U14","U15","U16","U17"]
  top_teams           JSONB NOT NULL,           -- [{age_group, team_id, team_name, rating}]
  state               CHAR(2),                  -- for state-level filtering
  PRIMARY KEY (club_id, gender, computed_at)
);

-- Indexes for common query patterns
CREATE INDEX idx_club_rankings_gender_rank
  ON club_rankings (gender, club_rank)
  WHERE club_rank IS NOT NULL;

CREATE INDEX idx_club_rankings_state
  ON club_rankings (state, gender, club_rank);

CREATE INDEX idx_team_club_assignments_club
  ON team_club_assignments (club_id);
```

---

## 5. Computation SQL — Full CTE Chain

```sql
-- ============================================================
-- Club Rankings Computation
-- Run nightly after compute-rankings.js completes
-- Target: youth_soccer DB
-- ============================================================

WITH

-- Step 1: Get all ranked, active teams with their club assignments
ranked_teams AS (
  SELECT
    tca.club_id,
    c.name           AS club_name,
    c.state,
    r.team_id,
    r.age_group,
    r.gender,
    r.rating,
    r.last_game_date
  FROM rankings r
  JOIN team_club_assignments tca ON tca.team_id = r.team_id
  JOIN clubs c ON c.id = tca.club_id
  WHERE
    r.age_group IN ('U11','U12','U13','U14','U15','U16','U17')
    AND r.rating IS NOT NULL
    -- Only count active teams (game in last 7 months)
    AND r.last_game_date > NOW() - INTERVAL '7 months'
),

-- Step 2: For each club × gender × age_group, pick the top-rated team only
top_team_per_agegroup AS (
  SELECT DISTINCT ON (club_id, gender, age_group)
    club_id,
    club_name,
    state,
    gender,
    age_group,
    team_id,
    rating
  FROM ranked_teams
  ORDER BY club_id, gender, age_group, rating DESC
),

-- Step 3: Aggregate to club level — count qualifying age groups and compute avg
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
        'rating',    rating
      )
      ORDER BY age_group
    )                       AS top_teams_json,
    JSONB_AGG(age_group ORDER BY age_group) AS age_groups_eligible
  FROM top_team_per_agegroup
  GROUP BY club_id, club_name, state, gender
),

-- Step 4: Filter to clubs meeting the minimum 5-age-group threshold
eligible_clubs AS (
  SELECT *
  FROM club_aggregates
  WHERE age_groups_counted >= 5
),

-- Step 5: Rank clubs within gender
ranked_clubs AS (
  SELECT
    club_id,
    gender,
    ROUND(club_rating, 2)   AS club_rating,
    RANK() OVER (
      PARTITION BY gender
      ORDER BY club_rating DESC
    )                       AS club_rank,
    age_groups_counted,
    age_groups_eligible,
    top_teams_json          AS top_teams,
    state
  FROM eligible_clubs
)

-- Step 6: Insert results (or upsert with a timestamp partition)
INSERT INTO club_rankings
  (club_id, gender, computed_at, club_rating, club_rank,
   age_groups_counted, age_groups_eligible, top_teams, state)
SELECT
  club_id,
  gender,
  NOW()                   AS computed_at,
  club_rating,
  club_rank,
  age_groups_counted,
  age_groups_eligible,
  top_teams,
  state
FROM ranked_clubs;
```

> [!note] last_game_date in rankings table
> The computation above assumes `rankings.last_game_date` exists. If it doesn't, join to games:
> `AND EXISTS (SELECT 1 FROM games WHERE (home_team_id = r.team_id OR away_team_id = r.team_id) AND match_date > NOW() - INTERVAL '7 months' AND is_hidden = false)`

---

## 6. API Endpoint Specs

### Endpoint 1: Club Rankings List

```
GET /api/club-rankings?gender=F&state=TX&limit=50&offset=0
```

**Query parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `gender` | `M` \| `F` | required | Boys or Girls |
| `state` | 2-char | optional | Filter by club state |
| `limit` | int | 50 | Pagination |
| `offset` | int | 0 | Pagination |

**Response:**
```json
{
  "gender": "F",
  "state": "TX",
  "total": 312,
  "computed_at": "2026-04-23T02:14:00Z",
  "clubs": [
    {
      "club_rank": 1,
      "club_id": 42,
      "club_name": "FC Dallas",
      "state": "TX",
      "club_rating": 1478.5,
      "age_groups_counted": 7,
      "age_groups_eligible": ["U11","U12","U13","U14","U15","U16","U17"],
      "top_teams": [
        { "age_group": "U14", "team_id": "12345", "team_name": "FC Dallas 2011G ECNL", "rating": 1510.2 },
        { "age_group": "U15", "team_id": "12346", "team_name": "FC Dallas 2010G ECNL", "rating": 1490.1 }
      ]
    }
  ]
}
```

### Endpoint 2: Club Profile

```
GET /api/clubs/{club_id}
```

**Response:**
```json
{
  "club_id": 42,
  "club_name": "FC Dallas",
  "state": "TX",
  "gotsport_org_id": "org_1234",
  "rankings": {
    "boys": {
      "club_rank": 8,
      "club_rating": 1441.0,
      "age_groups_counted": 6,
      "top_teams": [ ... ]
    },
    "girls": {
      "club_rank": 1,
      "club_rating": 1478.5,
      "age_groups_counted": 7,
      "top_teams": [ ... ]
    }
  }
}
```

### Endpoint 3: Team's Club (for team pages)

```
GET /api/teams/{team_id}/club
→ Returns the club this team belongs to, with club's current rank
```

---

## 7. Bootstrap Plan — Seeding the Clubs Table

### Phase 1: GotSport Org ID Extraction (Week 1)

GotSport is Priority 1 and covers ~69% of all teams. The GotSport API likely returns an org/club ID for each team. Extract and store it:

```sql
-- After confirming gotsport_org_id column exists on teams table:
INSERT INTO clubs (name, normalized_name, gotsport_org_id, state)
SELECT DISTINCT
  club                                          AS name,
  LOWER(REGEXP_REPLACE(club, '[^a-zA-Z0-9 ]', '', 'g')) AS normalized_name,
  gotsport_org_id,
  NULL                                          AS state  -- populate manually or from team address
FROM teams
WHERE gotsport_org_id IS NOT NULL
  AND club IS NOT NULL
  AND club != ''
ON CONFLICT (gotsport_org_id) DO NOTHING;

-- Then create team_club_assignments from GotSport org_id matches:
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

### Phase 2: Name-Based Matching for Remaining Teams (Week 2)

For teams without a GotSport org_id, use the `club` column with normalized matching:

```sql
INSERT INTO team_club_assignments (team_id, club_id, assignment_method, confidence)
SELECT
  t.team_id,
  c.id,
  'name_parse',
  0.75
FROM teams t
JOIN clubs c ON c.normalized_name = LOWER(REGEXP_REPLACE(t.club, '[^a-zA-Z0-9 ]', '', 'g'))
WHERE t.team_id NOT IN (SELECT team_id FROM team_club_assignments)
  AND t.club IS NOT NULL
  AND t.club != ''
ON CONFLICT (team_id) DO NOTHING;
```

### Phase 3: Manual Verification (Ongoing)

- Priority verify clubs that rank in Top 100 nationally
- Build a simple admin UI for Presten to approve/reject fuzzy matches (confidence < 0.90)
- Set `clubs.is_verified = true` once reviewed

---

## 8. Implementation Timeline Estimate

| Week | Tasks | Effort |
|------|-------|--------|
| Week 1 | Confirm teams table schema, run coverage queries, create `clubs` + `team_club_assignments` + `club_rankings` tables | 4 hrs |
| Week 1 | Bootstrap clubs from GotSport org_id data | 2 hrs |
| Week 2 | Build and test computation CTE — validate results against known top clubs | 3 hrs |
| Week 2 | Schedule computation as nightly cron job after `compute-rankings.js` | 1 hr |
| Week 3 | Build `/api/club-rankings` and `/api/clubs/:id` endpoints | 3 hrs |
| Week 3 | Frontend: club rankings page (sortable table, gender toggle, state filter) | 4 hrs |
| Week 4 | Frontend: club profile page with age group breakdown | 3 hrs |
| Week 4 | QA, edge cases, top-club verification | 2 hrs |
| **Total** | | **~22 hrs** |

**Achievable within 4 weeks of implementation start.** If started week of April 28, launch by May 23 — one week after DSS.

---

## Related Notes
- [[Ranking Engine]] — team Elo ratings that feed club ranking computation
- [[Database Schema]] — `teams`, `rankings`, `team_merges` tables
- [[Dedup Strategy]] — team identity across sources (relevant for club assignment)
- [[Age Groups and Birth Years]] — U11–U17 scope definition
- [[USA Rank Competitive Intel]] — competitor club ranking methodology
