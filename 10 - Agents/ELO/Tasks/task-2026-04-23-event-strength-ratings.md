---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-23
status: complete
completed: 2026-04-23
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength Ratings Design.md"
priority: high
---

# Task: Design Event Strength Rating System

## Context

USA Rank displays event strength indicators on their event pages — including High/Average/Low quality ratings, spread (competitive balance), and upset rates. These signals tell coaches and parents whether a tournament was genuinely competitive or a low-stakes round-robin. We have no event-level quality metrics despite having ~10,500 events in our database with 1.65M games.

Event strength ratings serve multiple purposes:
1. They make our event pages valuable to browse (not just data dumps)
2. They feed directly into ranking accuracy — a win at a High-strength event should matter more
3. They differentiate us from USA Rank if we can display richer metrics (they show labels; we can show the math)

Read before starting:
- `02 - Tiger Tournaments/Projects/Knowledge/USA Rank Competitive Intel.md` — Match Predictions section, event page observations
- `02 - Tiger Tournaments/Projects/Rankings/Ranking Engine.md` — how opponent strength weighting works in Glicko v7
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — events table, games table structure
- `02 - Tiger Tournaments/Projects/Knowledge/Cross-League Analysis.md` — methodology for the 575K-game study (reference for stat computation approach)

## What You Need to Design

### 1. Define the Metrics

Design a suite of event strength signals. Required metrics:

**A. Average Team Rating (Event Rating)**
- Mean Elo rating of all participating teams at the event
- Normalize by age group (a U13G event's avg rating is compared against other U13G events, not U17B)
- Output: numeric score + percentile rank among all events in that age group

**B. Strength Classification (High / Average / Low)**
- Map the average team rating percentile to a 3-tier label
- Define the percentile thresholds: e.g., Top 20% = High, 20–60% = Average, Bottom 40% = Low
- Or use a different cutoff — justify with the data distribution

**C. Competitive Spread**
- Standard deviation of team ratings within the event
- Low spread = tightly competitive field (all teams similar quality)
- High spread = blowout risk (wide rating gaps)
- How to label spread: Tight / Mixed / Wide

**D. Upset Rate**
- An "upset" = lower-rated team wins (or draws) against a higher-rated opponent
- Formula: `upsets / total_decisive_games` (exclude draws if using strict upset definition)
- Rating gap threshold for an upset: define minimum gap (e.g., 50 Elo points) to avoid counting noise
- Output: percentage, labeled Low (<20%) / Average (20–40%) / High (>40%)

**E. (Optional) Schedule Strength**
- Average rating of opponents faced by each team
- Used in USA Rank as "schedule rating"
- This is per-team, not per-event — include if feasible without major scope increase

### 2. Data Availability Assessment
- How are events currently stored? Query: `SELECT id, event_name, age_group, game_count FROM events LIMIT 20`
- Do games link to events via a foreign key? (`games.event_id → events.id`)
- Are team Elo ratings available at the time of the event, or only the current (post-event) rating?
  - Important: event strength should use ratings AT THE TIME of the event, not current ratings
  - Is there a `ratings_history` table or equivalent? If not, is current rating an acceptable approximation?
- How many events have 5+ games and 4+ distinct teams? (Minimum for meaningful event strength)

### 3. Computation Design

Write the SQL for each metric:

```sql
-- Event Rating (average team rating at event)
SELECT 
  e.id as event_id,
  e.event_name,
  e.age_group,
  AVG(r.rating) as avg_team_rating,
  STDDEV(r.rating) as rating_spread,
  COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) as team_count
FROM events e
JOIN games g ON g.event_id = e.id
JOIN rankings r ON r.team_id IN (g.home_team_id, g.away_team_id)
WHERE g.is_hidden = false
GROUP BY e.id, e.event_name, e.age_group
HAVING COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) >= 4
```

Review and revise this query. Then write:
- Upset rate computation SQL
- Strength classification CASE statement (High/Avg/Low based on percentile)
- Spread classification CASE statement (Tight/Mixed/Wide)

### 4. Database Schema

Design the `event_strength` table:

```sql
CREATE TABLE event_strength (
  event_id INT REFERENCES events(id),
  computed_at TIMESTAMPTZ,
  age_group VARCHAR(10),
  gender CHAR(1),
  team_count INT,
  avg_team_rating NUMERIC(8,2),
  rating_spread NUMERIC(8,2),
  upset_rate NUMERIC(5,4),
  strength_label VARCHAR(10),   -- 'High', 'Average', 'Low'
  spread_label VARCHAR(10),     -- 'Tight', 'Mixed', 'Wide'
  upset_label VARCHAR(10),      -- 'Low', 'Average', 'High'
  strength_percentile NUMERIC(5,2),
  PRIMARY KEY (event_id, computed_at)
);
```

Review and revise. What's missing? What should change?

### 5. Integration with Ranking Engine
- Should event strength feed back into the Elo/Glicko v7 computation?
- Current system: "games against similar-strength opponents count more" — is this already implemented, or is it a gap?
- If event strength is computed post-hoc, how do we use it to adjust historical rating calculations?
- Recommendation: start with event strength as a display metric only (no ranking adjustment), then evaluate integration in Phase 2. Confirm or challenge this recommendation with reasoning.

### 6. API and Frontend Design
- New endpoint: `GET /api/events/{event_id}/strength` → returns all strength metrics + labels
- New endpoint: `GET /api/events?strength=High&age_group=U15G` → filter events by strength label
- Frontend: event detail page shows a strength card with label badges and metric numbers

## What Done Looks Like

Produce a design document at:
`02 - Tiger Tournaments/Projects/Rankings/Event Strength Ratings Design.md`

The document must include:
1. Metric definitions — all 4 required metrics (Rating, Classification, Spread, Upset Rate) fully specified
2. Data availability findings — query results confirming events/games schema supports computation
3. Computation SQL — production-ready queries for each metric
4. `event_strength` table DDL — revised and production-ready
5. Ranking engine integration decision — with reasoning (display-only vs feedback loop)
6. API specs — both endpoints with request/response schemas
7. Edge cases — what happens for small events (1–3 teams), events with all draws, events with no rated teams
8. Rollout plan — which events to compute first (most recent? largest? all?)

## Success Criteria

- Design document written with all 8 sections
- SQL is production-ready — Presten can run it against `youth_soccer` DB
- The 4 required metrics are fully defined with thresholds, not vague descriptions
- Edge cases are handled (not deferred)
- Feature produces visible output on event pages — Presten can demo this at DSS if implemented before May 16
