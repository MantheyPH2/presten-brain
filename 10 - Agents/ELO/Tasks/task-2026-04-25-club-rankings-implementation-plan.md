---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-23
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Club Rankings — Implementation Plan.md"
priority: high
due: 2026-04-25
---

# Task: Write Implementation Plan for Club Rankings (DB → Compute → API → Frontend)

## Context

You delivered the Club Rankings Design (`02 - Tiger Tournaments/Projects/Rankings/Club Rankings Design.md`) on April 23. The design is complete. You followed the same workflow for Rank Bands and Event Strength — design first, then an implementation plan that converts the design into a sequenced, zero-ambiguity execution guide. Club Rankings needs the same treatment now.

Rank Bands estimated 5.5 hours. Event Strength estimated 13 hours. Based on your design, Club Rankings likely lands at 8–12 hours — the `clubs` table bootstrapping step adds complexity vs Rank Bands but the computation logic is simpler than Event Strength.

DSS is May 16. Club Rankings is on the demo-worthy feature list. With the Rank Bands and Event Strength plans already written, this is the last major design-to-plan conversion before implementation season begins.

Read before starting:
- `02 - Tiger Tournaments/Projects/Rankings/Club Rankings Design.md` — your design. Source of truth.
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — teams table (confirm `gotsport_org_id` or equivalent club identifier field), rankings table
- `02 - Tiger Tournaments/Projects/Infrastructure/Server and Frontend.md` — existing API endpoints, frontend structure
- `02 - Tiger Tournaments/Projects/Rankings/Rank Bands — Implementation Plan.md` — reference format and level of detail

## What to Produce

### 1. Pre-Flight Checklist

Before Presten writes a single line:

- Confirm `teams` table has a usable club identifier (GotSport org_id or equivalent) — query: `SELECT column_name FROM information_schema.columns WHERE table_name = 'teams'`
- Run coverage estimate from your design: what % of ranked teams have a non-null club identifier? The 70–80% figure from your design must be verified, not assumed
- Confirm `rankings` table has age_group and gender columns
- Check whether a `clubs` table already exists: `SELECT COUNT(*) FROM information_schema.tables WHERE table_name = 'clubs'`
- Confirm no club rankings table exists yet: `SELECT COUNT(*) FROM information_schema.tables WHERE table_name = 'club_rankings'`

If the coverage estimate is below 60%, flag immediately — bootstrapping strategy changes.

### 2. Step-by-Step Implementation Sequence

Each step: what to do, exact SQL/code, how to verify, time estimate.

**Phase 1 — Bootstrap the `clubs` Table (~2 hours)**

Step 1: Create the `clubs` and `club_rankings` tables from your design DDL — copy-paste ready  
Step 2: Seed the `clubs` table from existing team data  
- If GotSport `org_id` is available on teams: `INSERT INTO clubs SELECT DISTINCT org_id, name FROM teams WHERE org_id IS NOT NULL` — adapt to actual column names  
- If club name must be parsed from team name: define the parsing rule (e.g., strip age group suffix: "FC Dallas 2011B" → "FC Dallas") and write the INSERT  
Step 3: Verify: `SELECT COUNT(*) FROM clubs` — confirm seeded count is plausible  
Step 4: Create `team_club_assignments` from the seeded clubs — joining back to teams by org_id or parsed name  
Step 5: Spot check 10 club assignments: do the teams assigned to "FC Dallas" all belong to FC Dallas?

**Phase 2 — Computation Script (~3 hours)**

Step 6: Write `compute-club-rankings.js` (or SQL script):
- For each club + gender combination, find all ranked teams in U11–U17
- Select the top-rated team per age group (per your methodology)
- Discard clubs with fewer than 5 age groups represented
- Compute `club_rating = AVG(top team rating per age group)`
- Rank by club_rating descending, separately for boys and girls
- Insert into `club_rankings` (UPSERT on club_id + gender + computed_at)

Step 7: Test on a sample — run the script, confirm output in `club_rankings`  
Step 8: Spot check: find 3 well-known clubs (e.g., Eclipse Select, FC Dallas, PA Classics). Are their rankings plausible?  
Step 9: Add to nightly cron after rankings recompute

**Phase 3 — API (~2 hours)**

Step 10: Implement `GET /api/club-rankings?gender=F&state=TX` — returns ranked list of clubs with club_rating, club_rank, age_groups_counted, top_teams JSONB  
Step 11: Implement `GET /api/clubs/:club_id` — club profile with all team ratings by age group  
Step 12: Test both endpoints with curl  
Step 13: Confirm response includes top_teams JSONB (needed for the frontend club profile view)

**Phase 4 — Frontend (~3 hours)**

Step 14: Add club rankings page (`/club-rankings`) — sortable table: rank, club name, state, rating, age groups covered  
Step 15: Add age group coverage indicator (e.g., U11 ✓ U12 ✓ U13 ✓ U14 ✗ U15 ✓ U16 ✓ U17 ✗ — shows which age groups qualify)  
Step 16: Add gender toggle (Boys / Girls)  
Step 17: Add optional state filter dropdown  
Step 18: Add club detail page (`/clubs/:id`) — shows the club's individual team rankings by age group  
Step 19: Test: load the club rankings page, confirm data renders, filter works

### 3. Code Artifacts (Complete and Ready to Run)

**A. DDL** — `CREATE TABLE clubs` and `CREATE TABLE club_rankings` — finalized from your design, production-ready

**B. Bootstrap INSERT** — the SQL to seed `clubs` from existing team data; adapt based on what the pre-flight check reveals about the actual club identifier field

**C. Computation SQL** — full CTE chain implementing the 6-step ranking logic from your design:
1. Join teams to their clubs via team_club_assignments
2. Filter to U11–U17, active teams only (last game < 7 months ago)
3. Rank teams within each club by rating, take the top per age group
4. Count distinct age groups per club
5. Filter to clubs with 5+ age groups
6. AVG the top team ratings, rank by result

**D. `compute-club-rankings.js` skeleton** — Node.js script that runs the computation SQL and logs results. Follow pattern from existing pipeline scripts.

**E. API handler stubs** — both endpoints with request/response schemas. Include the `top_teams` JSONB structure so frontend knows what to expect.

### 4. Bootstrap Contingency

The `clubs` table has never been seeded. Two cases:

**Case A: GotSport org_id is available on teams (the design's intended path)**  
- Bootstrap is clean: one INSERT from distinct org_ids
- Document exact SQL and expected output

**Case B: org_id is not on teams, club name must be parsed**  
- Provide a regex/string-split approach for the most common pattern (club name is the first N words before a year-group suffix)
- Estimate false positive rate and flag clubs where parsing is ambiguous
- Define the manual review process for the top 100 ambiguous clubs

The plan must handle both cases so Presten doesn't hit a wall in pre-flight.

### 5. Rollback Plan

- How to drop `clubs`, `club_rankings`, `team_club_assignments` if the bootstrap produces bad data
- How to remove the API endpoints without affecting existing routes
- Nightly cron: how to disable without affecting other cron jobs

### 6. Definition of Done

- [ ] `clubs` table seeded with real clubs (coverage ≥ 60% of ranked teams)
- [ ] `club_rankings` populated for both boys and girls
- [ ] At least 50 clubs qualify (5+ age groups) per gender
- [ ] `GET /api/club-rankings` returns correct data
- [ ] `GET /api/clubs/:id` returns correct team breakdown
- [ ] Club rankings page renders with correct sort and filter
- [ ] Known-club spot check passes (FC Dallas, Eclipse, etc. appear at plausible positions)
- [ ] Nightly recompute scheduled

## What Done Looks Like

Produce an implementation plan at:
`02 - Tiger Tournaments/Projects/Rankings/Club Rankings — Implementation Plan.md`

Format: same structure and level of detail as `Rank Bands — Implementation Plan.md` and `Event Strength Ratings — Implementation Plan.md`. No placeholders. All 5 code artifacts complete.

## Success Criteria

- Plan is executable in a single session (8–12 hours estimated)
- Bootstrap contingency covers both org_id and name-parse paths
- The spot-check step is specified so Presten can validate output quality before shipping
- Club rankings are demo-ready for DSS (May 16) if implementation begins by May 2
