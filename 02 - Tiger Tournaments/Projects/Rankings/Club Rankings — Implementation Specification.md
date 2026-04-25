---
title: Club Rankings — Implementation Specification
type: implementation-spec
author: ELO
date: 2026-04-24
status: ready-for-may2
ordering_constraint: "Must not run before April 28 GA ASPIRE fix + 24h stabilization. Target session: May 2–3."
related: "[[Club Rankings — Implementation Plan]]", "[[Club Rankings — Calibration Pre-Assessment]]", "[[DSS Data Readiness Validation Plan — May 14-15]]"
tags: [club-rankings, implementation, sql, evo-draw, dss]
---

# Club Rankings — Implementation Specification

> **Summary:** Step-by-step execution guide for the May 2–3 club rankings implementation session. Depends on the April 28 GA ASPIRE fix (material dependency — GA ASPIRE overcalibration would corrupt Girls club ratings by 30–50 points for ASPIRE-heavy clubs). All aggregation methodology is specified explicitly. PA Classics (DSS demo club) must appear at rank ≤30 Girls to confirm demo readiness.

---

## Section 0: Pre-Flight Checks

Run before writing any code or creating any tables. ~30 minutes total.

**Gate A — GA ASPIRE fix has stabilized (≥24h since April 28 run)**

```sql
-- Confirm GA ASPIRE events are correctly tagged post-fix
SELECT COUNT(*) AS aspire_still_tagged_ga
FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga';
-- Expected: 0. If >0, the April 28 fix did not run. Stop.
```

Also confirm ranking recompute has run post-fix:
```sql
-- Check that rankings were updated after April 28
SELECT MAX(last_game_date) AS most_recent_game_in_rankings
FROM rankings;
-- Expected: a date on or after April 28. If this shows April 27 or earlier, the recompute has not yet run.
```

If either condition fails, do not proceed. The GA ASPIRE fix is a material dependency — club ratings for ASPIRE-heavy clubs will be 30–50 points too high without it.

**Gate B — Boys pre-flight (only needed if computing Boys club rankings)**

```sql
-- Boys Option A pre-flight: sufficient cross-tier Boys GA ASPIRE games?
SELECT g.gender, COUNT(DISTINCT g.id) AS game_count
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_tier = 'ga_aspire'
  AND g.gender = 'M'
GROUP BY g.gender;
```

- If Boys game_count < 300: Boys club rankings are not supported at this time. Compute Girls only. Flag to SENTINEL.
- If Boys game_count ≥ 300: proceed with Boys club rankings.

**Gate C — Confirm tables don't already exist**

```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
  AND table_name IN ('clubs', 'club_rankings', 'team_club_assignments');
```

Expected: 0 rows. If any tables exist, review before proceeding — they may be from a prior attempt. If they look correct and populated, skip Phase 1 and go directly to Phase 2 validation.

**Gate D — Confirm club identifier on `teams` table**

```sql
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'teams'
  AND column_name IN ('club', 'gotsport_org_id', 'org_id', 'team_id', 'last_game_date')
ORDER BY column_name;
```

- If `gotsport_org_id` is present AND coverage ≥60% (check Gate D-2): proceed with Case A (org_id path).
- Otherwise: proceed with Case B (name-parse path).

```sql
-- Gate D-2: org_id coverage check
SELECT
  COUNT(*) AS total_teams,
  COUNT(CASE WHEN gotsport_org_id IS NOT NULL THEN 1 END) AS with_org_id,
  ROUND(COUNT(CASE WHEN gotsport_org_id IS NOT NULL THEN 1 END) * 100.0 / COUNT(*), 1) AS pct_org_id
FROM teams;
-- If pct_org_id >= 60: Case A. Otherwise: Case B.
```

---

## Section 1: Schema — Three Tables

```sql
-- Canonical club registry
CREATE TABLE clubs (
  id               SERIAL PRIMARY KEY,
  name             VARCHAR(200) NOT NULL,
  normalized_name  VARCHAR(200) NOT NULL,
  state            CHAR(2),
  gotsport_org_id  VARCHAR(50),
  website          VARCHAR(300),
  is_verified      BOOLEAN DEFAULT false,
  created_at       TIMESTAMPTZ DEFAULT NOW(),
  updated_at       TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE (gotsport_org_id),
  UNIQUE (normalized_name, state)
);

-- Many-to-one: team → club assignment
CREATE TABLE team_club_assignments (
  team_id           VARCHAR(100) NOT NULL,
  club_id           INT NOT NULL REFERENCES clubs(id) ON DELETE CASCADE,
  assignment_method VARCHAR(20) NOT NULL,  -- 'gotsport_org_id', 'name_parse'
  confidence        NUMERIC(3,2),          -- 1.00 = org_id, 0.75 = name_parse
  assigned_at       TIMESTAMPTZ DEFAULT NOW(),
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

CREATE INDEX idx_club_rankings_gender_rank ON club_rankings (gender, club_rank)
  WHERE club_rank IS NOT NULL;
CREATE INDEX idx_club_rankings_state ON club_rankings (state, gender, club_rank);
CREATE INDEX idx_team_club_assignments_club ON team_club_assignments (club_id);
```

---

## Section 2: Club Aggregation Methodology

The following methodology applies to the computation SQL in Section 3. All decisions are stated explicitly.

**Which teams are included:**
Active teams only — `last_game_date > NOW() - INTERVAL '7 months'` AND `rank IS NOT NULL`. Stale teams (no games in 7 months) do not contribute to club ratings. This prevents clubs that still have legacy teams in the DB from appearing inflated.

**Age groups covered:**
U11, U12, U13, U14, U15, U16, U17. These are the 7 qualifying age groups. Teams in other age groups (U8, U10, U18, U19) are excluded.

**How ratings aggregate:**
Average of the top team rating per age group per gender per club. One team per age group contributes — the highest-rated active team in that age group. This is straight average (not weighted by game count) because game count is already embedded in the rating via Glicko-2. A team with more games has a more stable, reliable rating; a team with fewer games is closer to the initialization value and will naturally contribute a lower rating.

**Minimum team threshold:**
A club must have ≥5 qualifying age groups (active, ranked, within 7 months) to appear in club rankings. Clubs with 4 or fewer age groups are excluded. This threshold ensures club rankings reflect multi-age-group programs, not single-team clubs that happen to be highly rated.

**Gender split:**
Girls and Boys are computed separately. A club gets a Girls rank and a Boys rank independently. The Girls rank is based on Girls teams only; the Boys rank is based on Boys teams only.

**Age group weighting:**
All 7 age groups contribute equally to the club_rating average. No age group bonus or weighting for competitive tier — the team's rating already reflects competition quality through calibration values.

---

## Section 3: Bootstrap SQL — Seed the `clubs` Table

### Case A: GotSport org_id available (preferred path, if Gate D-2 shows ≥60%)

```sql
-- Insert distinct clubs from GotSport org_id
INSERT INTO clubs (name, normalized_name, gotsport_org_id, state)
SELECT DISTINCT
  club AS name,
  LOWER(TRIM(REGEXP_REPLACE(club, '[^a-zA-Z0-9 ]', '', 'g'))) AS normalized_name,
  gotsport_org_id,
  NULL AS state
FROM teams
WHERE gotsport_org_id IS NOT NULL
  AND club IS NOT NULL
  AND club != ''
ON CONFLICT (gotsport_org_id) DO NOTHING;

-- Create team assignments via org_id
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

### Case B: Name-parse path (if org_id coverage <60%)

```sql
-- Insert clubs from team name parsing
INSERT INTO clubs (name, normalized_name, state)
SELECT DISTINCT
  club,
  LOWER(TRIM(REGEXP_REPLACE(club, '[^a-zA-Z0-9 ]', '', 'g'))),
  NULL
FROM teams
WHERE club IS NOT NULL AND club != ''
ON CONFLICT (normalized_name, state) DO NOTHING;

-- Assign teams to clubs via name match
INSERT INTO team_club_assignments (team_id, club_id, assignment_method, confidence)
SELECT
  t.team_id,
  c.id,
  'name_parse',
  0.75
FROM teams t
JOIN clubs c ON c.normalized_name = LOWER(TRIM(REGEXP_REPLACE(t.club, '[^a-zA-Z0-9 ]', '', 'g')))
WHERE t.club IS NOT NULL AND t.club != ''
ON CONFLICT (team_id) DO NOTHING;
```

### Verify seeding

```sql
SELECT COUNT(*) AS club_count FROM clubs;
SELECT COUNT(*) AS assignment_count FROM team_club_assignments;
SELECT assignment_method, COUNT(*) AS count
FROM team_club_assignments
GROUP BY assignment_method;
```

Expected: clubs table has 5,000–20,000 rows; assignments cover ≥60% of ranked teams.

### Spot-check 10 assignments

```sql
SELECT c.name AS club_name, t.team_name, tca.assignment_method, tca.confidence
FROM team_club_assignments tca
JOIN clubs c ON c.id = tca.club_id
JOIN teams t ON t.team_id = tca.team_id
WHERE c.name ILIKE '%FC Dallas%'
   OR c.name ILIKE '%PA Classics%'
   OR c.name ILIKE '%Eclipse%'
ORDER BY c.name, t.team_name
LIMIT 30;
```

Confirm: teams look like they belong to the clubs they're assigned to.

---

## Section 4: Computation SQL

The full computation runs via `compute-club-rankings.js` (see [[Club Rankings — Implementation Plan]] Phase 2). The core SQL within that script:

```sql
INSERT INTO club_rankings
  (club_id, gender, computed_at, club_rating, club_rank,
   age_groups_counted, age_groups_eligible, top_teams, state)

WITH

ranked_teams AS (
  SELECT
    tca.club_id,
    c.name      AS club_name,
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
    AND r.rank IS NOT NULL
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
    COUNT(*)                   AS age_groups_counted,
    AVG(rating)                AS club_rating,
    JSONB_AGG(
      JSONB_BUILD_OBJECT(
        'age_group', age_group,
        'team_id',   team_id,
        'team_name', team_name,
        'rating',    ROUND(rating::numeric, 2)
      )
      ORDER BY age_group
    )                          AS top_teams,
    JSONB_AGG(age_group ORDER BY age_group) AS age_groups_eligible
  FROM top_team_per_agegroup
  GROUP BY club_id, club_name, state, gender
),

eligible_clubs AS (
  SELECT * FROM club_aggregates
  WHERE age_groups_counted >= 5
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
```

**Expected runtime:** 1–5 minutes on standard DB size. If >10 minutes, add index hint or batch by gender.

---

## Section 5: Validation Queries

Run all 4 after the INSERT completes.

**V1 — Counts by gender**

```sql
SELECT gender, COUNT(*) AS club_count, ROUND(AVG(club_rating), 1) AS avg_rating
FROM club_rankings
WHERE computed_at = (SELECT MAX(computed_at) FROM club_rankings)
GROUP BY gender;
```

Expected: ≥50 qualifying clubs per gender. If <20 clubs per gender, the `age_groups_counted >= 5` threshold is filtering too aggressively — check whether the 7-month activity filter is cutting off legitimate clubs (may need to extend to 8 months if the DB is thin on recent games).

**V2 — Top 10 Girls clubs**

```sql
SELECT cr.club_rank, c.name, cr.club_rating, cr.age_groups_counted
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE cr.gender = 'F'
  AND cr.computed_at = (SELECT MAX(computed_at) FROM club_rankings)
ORDER BY cr.club_rank
LIMIT 10;
```

Sanity check: recognizable national programs should appear (Eclipse, FC Dallas, PDA, PA Classics, NYCFC, etc.). If the top 5 are obscure regional clubs, the aggregation logic has an error — likely the bootstrap assigned club names incorrectly. Investigate the top 5 clubs' team assignments.

**V3 — PA Classics presence (DSS demo club)**

```sql
SELECT c.name, cr.club_rank, cr.club_rating, cr.age_groups_counted
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE (c.name ILIKE '%PA Classics%' OR c.name ILIKE '%Pennsylvania Classics%')
  AND cr.gender = 'F'
  AND cr.computed_at = (SELECT MAX(computed_at) FROM club_rankings);
```

Expected: present, rank ≤30, age_groups_counted ≥5.

- If not present: the club was not seeded correctly or none of PA Classics' teams have 5+ qualifying age groups. Check which PA Classics teams exist in the DB and whether they have 7-month activity.
- If rank >30: flag to SENTINEL — may need to pivot to PDA or NJ Wildcats as the demo club. Do not adjust the ranking methodology to make PA Classics look better.

**V4 — Boys club rankings baseline**

```sql
SELECT cr.club_rank, c.name, cr.club_rating, cr.age_groups_counted
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE cr.gender = 'M'
  AND cr.computed_at = (SELECT MAX(computed_at) FROM club_rankings)
ORDER BY cr.club_rank
LIMIT 10;
```

Cross-reference with Gate B result. If Gate B showed <300 Boys GA ASPIRE games and Boys club rankings were computed anyway, flag this as a potential data quality issue. Boys club rankings in that case should carry a caveat at DSS.

---

## Section 6: Boys-Only Pre-Flight Failure Protocol

If Gate B (Boys pre-flight) returns game_count < 300 at run time:

1. Compute Girls club rankings only (run the Section 4 SQL with `WHERE r.gender = 'F'` added to `ranked_teams` CTE).
2. Flag to SENTINEL: "Boys club rankings not computed — Boys GA ASPIRE game volume < 300. Boys club ranks should not be presented at DSS."
3. Do NOT lower the threshold to 200 or 100 to make Boys work — the threshold exists because <300 cross-tier Boys games means calibration values for Boys ASPIRE are noise-dominated, and club rankings built on those ratings are unreliable.

---

## Section 7: Rollback

```sql
-- Clear club rankings data (keeps tables in place for recompute)
TRUNCATE club_rankings;

-- OR drop all three tables entirely
DROP TABLE IF EXISTS club_rankings;
DROP TABLE IF EXISTS team_club_assignments;
DROP TABLE IF EXISTS clubs;
```

No downstream tables depend on `clubs`, `team_club_assignments`, or `club_rankings` at Phase 1 implementation time. Rollback is clean.

For code rollback:
- Remove `compute-club-rankings.js` from the nightly pipeline script
- Remove the two API route handlers from `server.js`
- Remove `ClubRankings.jsx` and `ClubDetail.jsx` from the frontend

---

## Execution Timeline (May 2–3 Session)

| Session Block | Task | Time |
|---|---|---|
| May 2 AM | Pre-flight checks (Section 0) | 30 min |
| May 2 AM | Phase 1: Bootstrap clubs table (Section 1–3) | 2 hr |
| May 2 PM | Phase 2: Computation script + nightly cron (Section 4) | 3 hr |
| May 3 AM | Phase 3: API endpoints | 2 hr |
| May 3 PM | Phase 4: Frontend pages | 3 hr |
| May 3 PM | V1–V4 validation, spot-checks, PA Classics check | 30 min |
| **Total** | | **~10–11 hr** |

---

## References

- [[Club Rankings — Calibration Pre-Assessment]] — sequencing decisions and Boys risk assessment
- [[Club Rankings — Implementation Plan]] — full 4-phase implementation guide (DDL, computation SQL, API, frontend)
- [[DSS Data Readiness Validation Plan — May 14-15]] — Section 3: club rankings validation criteria
- [[Boys Calibration Option A — Interpretation Guide]] — Boys pre-flight interpretation thresholds
- [[Boys Calibration Gap Analysis — April 2026]] — Boys calibration validation status
