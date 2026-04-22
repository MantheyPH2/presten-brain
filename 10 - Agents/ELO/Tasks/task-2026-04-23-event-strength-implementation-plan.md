---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-23
status: pending
priority: high
---

# Task: Write Implementation Plan for Event Strength Ratings (DB → Compute → API → Frontend)

## Context

Yesterday ELO delivered the Event Strength Ratings Design (`02 - Tiger Tournaments/Projects/Rankings/Event Strength Ratings Design.md`). The design is approved. Four metrics are defined: Average Team Rating, Strength Classification (High ≥p80 / Average p40–p79 / Low <p40), Competitive Spread (Tight/Mixed/Wide), and Upset Rate (Low <20% / Average 20–40% / High >40%). The `event_strength` table DDL is specified. Implementation estimate was ~13 hours.

SENTINEL priority: Event Strength must be demo-ready for DSS (May 16, 2026). Per ELO's own timeline, Presten needs to start by April 28 to demo on May 9. This plan needs to be ready NOW so implementation can begin immediately.

USA Rank shows strength labels. We will show labels AND the underlying numbers (percentile, StdDev, upset %). This is a competitive advantage — document it in the plan as the differentiator.

Read before starting:
- `02 - Tiger Tournaments/Projects/Rankings/Event Strength Ratings Design.md` — your design from yesterday. Source of truth.
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — events table, games table — confirm `event_id` FK, `home_team_id`, `away_team_id`, score fields
- `02 - Tiger Tournaments/Projects/Infrastructure/Server and Frontend.md` — current endpoints, frontend event page structure
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — "Match Predictions" section, event page observations

## What You Need to Produce

### 1. Pre-Flight Checklist
Before implementation begins:
- Confirm `events` table exists with columns: `id`, `event_name`, `age_group`, `gender`
- Confirm `games.event_id` references `events.id` (or document fallback if no FK — use `games.event_name` match)
- Confirm `rankings` table has current ratings (query: `SELECT COUNT(*) FROM rankings WHERE rating IS NOT NULL`)
- Confirm no `ratings_history` table exists — current rating approximation is acceptable (per design doc decision)
- Run edge case count: `SELECT COUNT(*) FROM events e JOIN games g ON g.event_id = e.id GROUP BY e.id HAVING COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) < 4` — how many events are below the 4-team minimum?

### 2. Step-by-Step Implementation Sequence

Each step: what to do, exact SQL/code to run, how to verify, time estimate.

**Phase 1 — Database (Target: ~2 hours)**
1. Run the `CREATE TABLE event_strength` DDL from the design doc
2. Verify: `\d event_strength`
3. Run the average team rating query (from design doc) on a sample of 10 events — confirm it returns results
4. Run the upset rate query on the same sample — confirm it returns a rate between 0 and 1
5. Confirm percentile computation works: `PERCENT_RANK() OVER (PARTITION BY age_group, gender ORDER BY avg_team_rating)`

**Phase 2 — Computation Script (Target: ~5 hours)**
6. Write `compute-event-strength.js` (or SQL script) that:
   a. Clears stale `event_strength` rows for the current season
   b. Computes avg_team_rating, rating_spread, team_count per event
   c. Computes upset_rate per event
   d. Computes strength_percentile using window function across events in same age_group + gender
   e. Assigns strength_label (High/Average/Low) from percentile
   f. Assigns spread_label (Tight/Mixed/Wide) from StdDev thresholds
   g. Assigns upset_label (Low/Average/High) from upset rate thresholds
   h. Inserts into `event_strength`
   i. Skips events with fewer than 4 distinct teams or fewer than 5 games (edge case handling from design)
7. Test on current season events (2025–2026): run script, confirm rows inserted
8. Spot check 5 events manually: do the labels make sense for known tournaments?
9. Add to nightly cron — run after rankings recompute (Steps 6–9 of main pipeline)

**Phase 3 — API (Target: ~2 hours)**
10. Add `GET /api/events/:event_id/strength` endpoint — returns all metrics + labels for one event
11. Add `GET /api/events?strength=High&age_group=U15G` filter endpoint — returns events matching criteria
12. Test both endpoints with curl
13. Confirm response includes numeric values (avg_team_rating, rating_spread as StdDev, upset_rate as decimal) in addition to labels — this is the differentiator vs USA Rank

**Phase 4 — Frontend (Target: ~4 hours)**
14. Add "Event Strength Card" component to the event detail page
15. Card displays: Strength badge (High/Average/Low in color), Average Rating (numeric + percentile), Spread badge (Tight/Mixed/Wide), Upset Rate (% + label)
16. Add tooltip or expandable section showing: "This event ranked in the Xth percentile for U15G events" — transparent methodology note
17. Add event strength filter to any existing events listing page
18. Test: load 3 different event pages — confirm cards render with correct labels and numbers

### 3. Code Artifacts (Complete and Ready to Run)

**A. `CREATE TABLE event_strength` DDL** — from design doc, finalized, production-ready

**B. Full computation SQL** — complete CTE chain computing all metrics in one pass:
- avg_team_rating + rating_spread (STDDEV) per event
- upset_rate computation (games where lower-rated team won, 50-point gap minimum)
- percentile window function
- CASE statements for all three labels
- INSERT INTO event_strength

**C. `compute-event-strength.js` skeleton** — Node.js script that runs the computation SQL and logs results. Should follow same pattern as existing pipeline scripts.

**D. API handler stubs** — both endpoints: `GET /api/events/:event_id/strength` and `GET /api/events` with strength filter

**E. Event Strength Card component** — frontend component displaying all 4 metrics. Should visually distinguish our numeric transparency from USA Rank's label-only approach.

### 4. Edge Case Handling (from Design Doc — Confirm Implementation)

For each edge case, confirm the code handles it correctly:
- Event with fewer than 4 teams → skip, do not insert row
- Event with all draws → upset_rate = 0 (no decisive games), upset_label = 'Low'
- Event with no teams in rankings table → skip (cannot compute avg rating)
- Multi-age-group events → compute separately per age_group + gender sub-event
- Event re-runs (running the script twice) → UPSERT on (event_id, computed_at) or delete + reinsert

### 5. Rollback Plan
- Drop `event_strength` table if computation produces wrong results
- Remove API endpoints without affecting existing routes
- Nightly cron: document the exact crontab line and how to comment it out

### 6. Definition of Done
- [ ] `event_strength` table populated with current season events
- [ ] At least 100 events have computed strength ratings
- [ ] `GET /api/events/:event_id/strength` returns correct data for 5 test events
- [ ] Event strength filter endpoint works
- [ ] Event detail page shows strength card with labels AND numeric values
- [ ] Nightly recompute is scheduled in cron
- [ ] Edge cases all confirmed handled (small events, all-draw events, unranked teams)

## What Done Looks Like

Produce an implementation plan at:
`02 - Tiger Tournaments/Projects/Rankings/Event Strength Ratings — Implementation Plan.md`

The plan must be executable starting April 28 to meet the May 9 demo-ready deadline.

## Success Criteria

- Plan written with all 6 sections
- All 5 code artifacts complete (no placeholders)
- The competitive differentiator (showing numbers, not just labels) is explicitly called out in the plan and in the frontend component
- Presten can start on April 28 and demo event strength cards at DSS on May 16
