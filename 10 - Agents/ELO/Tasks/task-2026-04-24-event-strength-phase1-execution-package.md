---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — Full Execution Package.md"
ordering_constraint: "Must run AFTER April 28 GA ASPIRE fix"
---

# Task: Event Strength Phase 1 — Full Execution Package

## Context

ELO has completed:
- Event Strength implementation plan (`Rankings/Event Strength — Implementation Plan.md`)
- Event Strength League Tier Coverage Audit (`Rankings/Event Strength — League Tier Coverage Audit.md`)
  - 8 core leagues confirmed with `event_tier` assignments
  - 6 unconfirmed leagues: USL Academy, EDP, Elite 64, NAL, State Leagues/USYS State Cup, ODP
  - Remediation SQL written for all 6
  - Critical ordering constraint confirmed: **Phase 1 MUST run after April 28 GA ASPIRE fix**

The DSS demo (May 16) may reference Event Strength. What does NOT exist yet: a complete, ready-to-run Phase 1 SQL package — the document Presten opens after April 28 to compute event strength ratings.

This task writes that package.

## What to Write

Write `02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — Full Execution Package.md`.

---

### Section 0: Pre-Flight Gate

Before any Phase 1 SQL runs, Presten checks two conditions:

**Gate A — April 28 GA ASPIRE fix has run**
```sql
-- Confirm GA ASPIRE event_tier is set to 'GA_ASPIRE' (not NULL or wrong value)
SELECT COUNT(*) FROM events WHERE event_name ILIKE '%GA Aspire%' AND event_tier IS NULL;
```
Expected: 0. If >0, the GA ASPIRE fix has NOT run. Stop — do not proceed with Phase 1.

**Gate B — Null event_tier count**
```sql
SELECT COUNT(*) FROM events WHERE event_tier IS NULL AND event_name IS NOT NULL;
```
If count > 0: run Section 1 remediation SQL before proceeding. If 0: skip to Section 2.

### Section 1: Remediation SQL (Run Only If Gate B > 0)

Pull the remediation SQL from `Rankings/Event Strength — League Tier Coverage Audit.md` (Section 3) and include it here verbatim. This assigns `event_tier` to the 6 unconfirmed leagues based on provisional League Hierarchy values.

After running, re-run Gate B. Must return 0 before continuing.

### Section 2: Schema — event_strength Table

Write the `CREATE TABLE IF NOT EXISTS event_strength` DDL:

```sql
CREATE TABLE IF NOT EXISTS event_strength (
  id                SERIAL PRIMARY KEY,
  event_id          INTEGER NOT NULL REFERENCES events(id),
  age_group         VARCHAR(10) NOT NULL,
  season            VARCHAR(10) NOT NULL,           -- e.g. '2025-2026'
  avg_team_rating   NUMERIC(8,2),                  -- avg rating of participating teams
  strength_class    VARCHAR(20),                   -- 'High' | 'Medium' | 'Low'
  strength_percentile NUMERIC(5,2),                -- 0.00–100.00
  competitive_spread  NUMERIC(8,2),                -- std dev of team ratings
  upset_rate          NUMERIC(5,4),                -- fraction of games where lower-rated team won
  team_count          INTEGER,
  computed_at         TIMESTAMP DEFAULT NOW(),
  UNIQUE(event_id, age_group)
);
```

If the table already exists: `TRUNCATE event_strength;` before inserting to ensure clean results.

### Section 3: Computation Queries

Write the Phase 1 computation — one INSERT per metric group, or a single CTE that computes all metrics and inserts in one pass. Choose whichever is cleaner and explain why.

The computation must calculate, per (event_id, age_group):

**avg_team_rating** — average rating of all teams that participated in the event for that age group. Pull team ratings from `rankings` at the time of the most recent game in the event (or current snapshot if full historical not available — note the assumption).

**competitive_spread** — standard deviation of team ratings for that event/age group combination.

**upset_rate** — fraction of games where the lower-rated team (at time of game) won. If ratings are snapshot-only (not time-series), use current ratings as a proxy and note the limitation.

**strength_percentile** — percentile rank of this event's avg_team_rating within all events of the same age_group and season. Use `PERCENT_RANK()` window function.

**strength_class** — derived from strength_percentile:
```
WHEN strength_percentile >= 75 THEN 'High'
WHEN strength_percentile >= 33 THEN 'Medium'
ELSE 'Low'
```

**team_count** — distinct teams in the event for that age_group.

### Section 4: Post-Computation Validation Queries

Write 4 validation queries to run immediately after the INSERT completes:

**V1 — Row count by season**
```sql
SELECT season, COUNT(*) FROM event_strength GROUP BY season ORDER BY season;
```
Expected: >50 events for '2025-2026'. If 0, computation failed silently.

**V2 — Strength class distribution (sanity check)**
```sql
SELECT strength_class, COUNT(*), ROUND(AVG(avg_team_rating), 1)
FROM event_strength
WHERE season = '2025-2026'
GROUP BY strength_class;
```
Expected: roughly 25% High, 42% Medium, 33% Low (by design of percentile cutoffs). Large skew suggests the percentile computation is off.

**V3 — Null metric check**
```sql
SELECT COUNT(*) FROM event_strength
WHERE avg_team_rating IS NULL OR strength_class IS NULL OR strength_percentile IS NULL;
```
Expected: 0.

**V4 — DSS demo query (pre-check)**
This is the exact query from the DSS Data Readiness Validation Plan Section 2B. Run it now to confirm the High-strength U13G event exists before May 14:
```sql
SELECT e.event_name, es.avg_team_rating, es.strength_class, es.strength_percentile,
       es.competitive_spread, es.upset_rate, es.team_count
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE es.age_group = 'U13G'
  AND es.strength_class = 'High'
  AND es.team_count >= 8
  AND e.event_name NOT ILIKE '%friendly%'
ORDER BY es.strength_percentile DESC
LIMIT 10;
```
If this returns 0 rows: flag for SENTINEL — the DSS demo's Event Strength card is at risk.

### Section 5: Rollback

```sql
TRUNCATE event_strength;
-- OR if you want to drop and recreate:
DROP TABLE IF EXISTS event_strength;
```
No downstream tables depend on event_strength in Phase 1. Rollback is clean.

## Execution Notes

Include an estimated runtime for the INSERT computation. If the events and games tables are large (>500K games), note whether a batch insert or index hint is needed.

## Definition of Done

- All 5 sections present (pre-flight, remediation, schema, computation, validation)
- Computation SQL is complete and runnable (no pseudocode)
- All 5 metrics are computed (avg_team_rating, competitive_spread, upset_rate, strength_percentile, strength_class, team_count)
- V4 pre-checks the DSS query
- Rollback SQL is present
- Ordering constraint (post-April 28) is stated in Section 0
- Summary in briefing: "Event Strength Phase 1 execution package written. Ready to run after April 28 GA ASPIRE fix. High-strength U13G event availability: [confirmed/unknown until Phase 1 runs]."

## Why This Matters

Event Strength is a new feature that Presten must implement before May 14 (DSS validation day). The design and ordering constraints are done. The only remaining gap is the SQL Presten runs to actually compute it. With April 28 as the ordering gate, FORGE could theoretically run Phase 1 immediately post-fix — but only if the execution package exists. Write it before April 28 so execution can happen the same session.

## References

- `02 - Tiger Tournaments/Projects/Rankings/Event Strength — Implementation Plan.md`
- `02 - Tiger Tournaments/Projects/Rankings/Event Strength — League Tier Coverage Audit.md` — Section 3: remediation SQL
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Data Readiness Validation Plan — May 14-15.md` — Section 2B: High-strength event query
- `10 - Agents/ELO/Tasks/task-2026-04-23-event-strength-implementation-plan.md`
