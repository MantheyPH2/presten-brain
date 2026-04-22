---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-23
status: pending
priority: high
---

# Task: Design Club Ranking System

## Context

USA Rank publishes club rankings — computed as the average of a club's top team ratings, covering ages U11–U17, requiring at least 5 age groups to qualify. We have no club ranking system. Clubs are a primary customer segment for division placement services, and club rankings are a high-visibility product that drives organic traffic (club directors and parents search for their club's standing).

We have 144,676 teams in the database with substantial club affiliation data in team names. The data foundation exists. This task is to design the ranking methodology and output system.

Read before starting:
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — Club Rankings section
- `02 - Tiger Tournaments/Projects/Rankings/Ranking Engine.md` — Elo/Glicko v7, how team ratings are computed
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — teams table structure, club/org fields
- `02 - Tiger Tournaments/Projects/Pipeline/Dedup Strategy.md` — team_merges table, how teams are linked
- `02 - Tiger Tournaments/Projects/Knowledge/Age Groups and Birth Years.md` — U11–U17 scope definition

## What You Need to Design

### 1. Club Identity Resolution
- How are clubs currently identified in the database? (Club name in team name string? Separate org_id field? GotSport club ID?)
- Query the teams table: `SELECT DISTINCT club_name FROM teams LIMIT 100` — what does the club field look like?
- Are clubs consistently named, or do "FC Dallas" and "FC Dallas Academy" and "FCD" all refer to the same club?
- Does GotSport provide an org_id or club_id that we already store?
- Design: what is the canonical club identifier? (Existing DB field vs new `clubs` table)

### 2. Team-to-Club Mapping
- Define the mapping logic: how does a team record get assigned to a club?
  - Option A: Parse club name from team name (fragile, requires NLP/fuzzy matching)
  - Option B: Use GotSport org_id if available in the teams table
  - Option C: Build a `clubs` table and a `team_club_assignments` junction table (most robust)
- Which option is feasible with current data? What % of teams have a usable club identifier?
- Write the SQL query to estimate coverage: how many of our 144,676 teams have a club identifier?

### 3. Ranking Methodology
USA Rank uses "average of top team ratings, U11–U17, 5+ age groups required." Design our version:

- **Eligible age groups:** U11, U12, U13, U14, U15, U16, U17 (boys and girls — combined or separate club rankings?)
- **Top team selection:** Does "top team" mean highest-rated team in each age group? What if a club has multiple U14G teams — do we take the top one or average all?
- **Minimum threshold:** Require teams in at least 5 of 7 eligible age groups (match USA Rank) or set our own threshold?
- **Rating input:** Use the Elo rating from the most recent ranking run, or a rolling average?
- **Gender handling:** Produce one combined club rank, or separate boys/girls club rankings? (Recommend separate — clubs like LAFC only have boys, others are girls-only.)
- **Activity requirement:** Should a club's team be counted if it's been inactive for 6+ months?

Write the formula as a SQL aggregate query.

### 4. Club Rankings Table Design
Design the database schema:

```sql
CREATE TABLE clubs (
  id SERIAL PRIMARY KEY,
  name VARCHAR(200) NOT NULL,
  normalized_name VARCHAR(200),  -- lowercase, stripped for matching
  state VARCHAR(2),
  gotsport_org_id VARCHAR(50),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE club_rankings (
  club_id INT REFERENCES clubs(id),
  gender CHAR(1),  -- 'M' or 'F'
  computed_at TIMESTAMPTZ,
  club_rating NUMERIC(8,2),
  club_rank INT,
  age_groups_counted INT,
  top_teams JSONB,  -- [{age_group, team_id, team_name, rating}]
  PRIMARY KEY (club_id, gender, computed_at)
);
```

Review and revise this schema. Add missing fields. Ensure it integrates with the existing `teams` and `rankings` tables.

### 5. Computation Script Design
- Write pseudocode (or a SQL CTE chain) for the club ranking computation:
  1. For each club, find all ranked teams in U11–U17
  2. For each age group, select the top-rated team
  3. If fewer than 5 age groups have ranked teams, mark club as ineligible
  4. Compute club_rating = AVG(top team rating per age group)
  5. Rank clubs by club_rating descending
  6. Insert results into `club_rankings`

### 6. API and Frontend Design
- New endpoint: `GET /api/club-rankings?gender=F&state=TX` → returns ranked list of clubs
- New endpoint: `GET /api/clubs/{club_id}` → club profile with all team ratings by age group
- Frontend view: sortable table of clubs with rank, rating, state, age group coverage indicators

## What Done Looks Like

Produce a design document at:
`02 - Tiger Tournaments/Projects/Rankings/Club Rankings Design.md`

The document must include:
1. Club identity resolution approach — chosen option with justification
2. Coverage estimate — what % of teams can be club-assigned today
3. Ranking methodology — the formula, eligibility rules, gender handling decision
4. SQL schema — `clubs` and `club_rankings` tables (production-ready DDL)
5. Computation SQL — CTE chain or pseudocode for the full ranking computation
6. API endpoint specs — request/response schemas for both endpoints
7. Bootstrap plan — how to seed the `clubs` table initially (from existing team names or GotSport org data)
8. Implementation timeline estimate

## Success Criteria

- Design document written with all 8 sections
- SQL DDL is production-ready
- The computation logic is correct — Presten can implement it in one session
- Coverage estimate is based on actual DB query results, not assumptions
- Club rankings are achievable as a feature within 4 weeks of implementation start
