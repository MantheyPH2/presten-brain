---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
priority: high
due: 2026-05-01
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Club Rankings — Implementation Specification.md"
ordering_constraint: "Must not run before April 28 GA ASPIRE fix + 24h stabilization. Target session: May 2-3."
---

# Task: Club Rankings — Implementation Specification

## Context

ELO completed the Club Rankings Calibration Pre-Assessment (`Rankings/Club Rankings — Calibration Pre-Assessment.md`), which established:
- GA ASPIRE dependency is material — run AFTER April 28 fix + 24h stabilization
- Boys club rankings: run Option A validation as pre-flight
- Target session: May 2–3
- U12/U13/U14/U19 calibration gap post-DSS is acceptable (precision improvement, not systematic error)

The existing Club Rankings implementation plan (`task-2026-04-25-club-rankings-implementation-plan.md`) contains design intent but NOT the step-by-step SQL Presten runs on May 2–3. The DSS Data Readiness Validation Plan (Section 3) references a `club_rankings` table that Presten needs to have created.

This task writes the full implementation spec: what to run, in what order, on May 2–3.

## What to Write

Write `02 - Tiger Tournaments/Projects/Rankings/Club Rankings — Implementation Specification.md`.

---

### Section 0: Pre-Flight Checks

**Gate A — GA ASPIRE fix has stabilized (≥24h since April 28 run)**
```sql
-- Confirm GA ASPIRE ratings have been recomputed post-fix
SELECT MAX(computed_at) FROM rankings WHERE age_group IN (
  SELECT DISTINCT age_group FROM rankings WHERE team_id IN (
    SELECT DISTINCT home_team_id FROM games g
    JOIN events e ON e.id = g.event_id
    WHERE e.event_tier = 'GA_ASPIRE'
  )
);
```
Expected: timestamp is ≥24 hours after April 28. If the most recent compute is from before April 28, rankings have not been updated — stop.

**Gate B — Girls vs Boys Option A validation (Boys only)**

If computing Boys club rankings, first run this pre-flight check from the pre-assessment:
```sql
-- Option A validation: sufficient cross-tier games for Boys GA ASPIRE?
SELECT COUNT(*) FROM games g
JOIN events e ON e.id = g.event_id
JOIN teams ht ON ht.id = g.home_team_id
JOIN teams at ON at.id = g.away_team_id
WHERE e.event_tier = 'GA_ASPIRE'
  AND ht.gender = 'M' AND at.gender = 'M';
```
If result < 300: flag for SENTINEL before computing Boys club rankings. Default to Girls-only if Boys pre-flight fails.

### Section 1: Schema — club_rankings Table

Write the `CREATE TABLE IF NOT EXISTS club_rankings` DDL. The table must support:
- `club_id` (FK to clubs)
- `gender` ('M' / 'F')
- `club_rank` (integer, rank within gender)
- `club_rating` (numeric — the computed aggregate)
- `age_groups_counted` (integer — how many age groups contributed)
- `top_team_rating` (numeric — highest-rated team contributing)
- `computed_at` (timestamp)
- `UNIQUE(club_id, gender)`

Include `TRUNCATE club_rankings;` before INSERT to ensure clean recompute.

### Section 2: Club Aggregation Logic

Specify the methodology for computing `club_rating` from team ratings. Pull from the implementation plan and state explicitly:

- **Which teams are included:** Only teams with `rank IS NOT NULL` and `last_game_date >= NOW() - INTERVAL '7 months'` (active teams). Stale teams do not contribute to club ratings.
- **How ratings aggregate:** Specify the weighting method — e.g., weighted average by number of recent games, straight average, or average of top-N teams per club. State which method is chosen and why (reference pre-assessment if it specifies).
- **Minimum team threshold:** How many teams must a club have to appear in club rankings? (e.g., ≥2 age groups). State the threshold.
- **Age group handling:** All age groups contribute equally, or are older/more competitive age groups weighted higher? State the decision.
- **Gender split:** Girls and Boys computed separately, producing two ranked lists. Confirmed.

### Section 3: Computation SQL

Write the INSERT statement for `club_rankings`. Use a CTE that:
1. Joins `teams` → `rankings` → `clubs` (if a clubs table exists) or groups by `team_name` pattern
2. Filters to active teams (≥7 months, not stale)
3. Computes `club_rating`, `age_groups_counted`, `top_team_rating` per (club, gender)
4. Filters out clubs below the minimum team threshold
5. Assigns `club_rank` using `RANK() OVER (PARTITION BY gender ORDER BY club_rating DESC)`

Note: If a canonical `clubs` table does not exist and teams are linked to clubs by name pattern or `club_id` on the `teams` table, state which column to use and include a comment in the SQL.

### Section 4: Validation Queries

**V1 — Counts by gender**
```sql
SELECT gender, COUNT(*) as club_count, ROUND(AVG(club_rating), 1) as avg_rating
FROM club_rankings GROUP BY gender;
```
Expected: >50 clubs per gender. If <20, the minimum team threshold or active filter is too strict — adjust threshold.

**V2 — Top 10 Girls clubs**
```sql
SELECT cr.club_rank, c.name, cr.club_rating, cr.age_groups_counted
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE cr.gender = 'F' ORDER BY cr.club_rank LIMIT 10;
```
Sanity check: top clubs should be recognizable programs (PDA, NYCFC, PA Classics, Eclipse, etc.). If unknown clubs appear in top 5, the aggregation logic has an error.

**V3 — PA Classics presence (DSS demo club)**
```sql
SELECT c.name, cr.club_rank, cr.club_rating, cr.age_groups_counted
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE (c.name ILIKE '%PA Classics%' OR c.name ILIKE '%Pennsylvania Classics%')
  AND cr.gender = 'F';
```
Expected: present, rank in top 30, age_groups_counted ≥ 5. If not present or rank > 30, flag for SENTINEL — may need to pivot to NJ Wildcats or PDA as the demo club.

**V4 — Boys Option A validation result**
After Boys club rankings are computed, re-run the Gate B query to confirm the pre-flight was accurate (not just that it passed, but that the result was as expected).

### Section 5: Boys-Only Pre-Flight Failure Protocol

If Gate B (Boys Option A validation) fails at runtime:
1. Compute Girls club rankings only.
2. Flag to SENTINEL: "Boys club rankings not computed — Option A pre-flight returned <300 cross-tier GA ASPIRE games. Boys ranks should not be presented at DSS."
3. Do NOT compute Boys rankings with a lower threshold — this is a data quality gate, not a configurable parameter.

### Section 6: Rollback

```sql
TRUNCATE club_rankings;
-- OR
DROP TABLE IF EXISTS club_rankings;
```

No downstream tables depend on `club_rankings` at time of Phase 1 computation. Rollback is clean.

## Execution Timeline

State the expected runtime for the INSERT on the current DB size. If >5 minutes, include a progress-check query Presten can run mid-session.

## Definition of Done

- All 6 sections present (pre-flight, schema, aggregation logic, SQL, validation, rollback)
- Computation SQL is complete and runnable
- Aggregation methodology is stated explicitly (not "TBD")
- PA Classics validation query is pre-written
- Boys-only failure protocol is specified
- Ordering constraint (post-April 28 + 24h) is stated in Section 0
- Summary in briefing: "Club Rankings implementation spec written at `Rankings/Club Rankings — Implementation Specification.md`. Ready for May 2–3 session. Pre-flight gates documented: [Gate A, Gate B]."

## Why This Matters

Club rankings is ELO's most ambitious new feature for DSS and the 8–12 hour session estimate makes it Presten's largest upcoming commitment. If Presten opens the May 2–3 session and has to figure out the methodology on the fly, the session runs long and the output may be miscalibrated. Write the spec now — before April 28 — so Presten can read it in advance and the session is pure execution, not design.

## References

- `02 - Tiger Tournaments/Projects/Rankings/Club Rankings — Calibration Pre-Assessment.md` — sequencing and Boys risk
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Data Readiness Validation Plan — May 14-15.md` — Section 3: validation queries (already aligned)
- `10 - Agents/ELO/Tasks/task-2026-04-25-club-rankings-implementation-plan.md` — design context
- `10 - Agents/ELO/Tasks/task-2026-04-24-club-rankings-calibration-pre-assessment.md` — ordering decision
