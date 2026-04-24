---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength Post-Compute Validation — 2026-04-24.md"
priority: high
due: 2026-04-30
---

# Task: Event Strength Post-Compute Validation Spec

## Context

The Event Strength Ratings implementation plan (`Rankings/Event Strength Ratings — Implementation Plan.md`) defines how to build the feature — schema, computation, API, frontend. It has a "Definition of Done" checklist. What it does not have is the post-compute validation: after Presten runs the 7-hour implementation session and the `event_strength` table is populated, how does Presten know the output is correct?

This is the same gap that existed for Rank Bands before ELO wrote the Rank Bands threshold validation spec. For Rank Bands, the spec made a 5.5-hour session safe by providing a 15-minute pre-flight. For Event Strength, the post-compute validation makes the 7-hour session safe by providing a 30-minute quality check before the feature goes live.

Without this spec, Presten could deploy Event Strength labels to the frontend and show users event strength scores that are miscalibrated. The DSS demo specifically calls for showing a High-strength event — if the classification is wrong (e.g., a regional friendly classified as High), it undermines the credibility claim.

## What You Need to Do

### Step 1: Define What Correct Event Strength Looks Like

From your design and knowledge of the youth soccer event landscape:

**High-strength events** should include:
- ECNL national events (GAL, ECNL Finals, ECNL Regionals with national-level attendance)
- MLS NEXT Academy League events (top competitive tier)
- GA National League Showcase events
- Major named invitationals with strong field depth (Surf Cup, Jefferson Cup, etc.)

**Medium-strength events** should include:
- ECNL regional events (not national showcases)
- GA regional events
- Most league play (state-level)

**Low-strength events** should include:
- Local and recreational league games
- Invitationals with open entry (no field quality gate)
- State cup lower divisions

Write this as a lookup table of specific named events you expect to find in the DB:
| Event Name (approximate) | Expected Strength | Why |
|---|---|---|
| ECNL Girls National Event | High | Top 1% of national competition |
| GA National League | High | Top-tier college prep circuit |
| [State] State Cup Elite Division | Medium | Competitive but state-level |
| [Generic invitational] | Low | Open entry, mixed quality |

This table is the ground truth for validating that the algorithm produces sensible results.

### Step 2: Write the Sanity Check Queries

After Presten runs Phase 1 and 2 of the Event Strength implementation:

**Query A: Confirm table is populated**
```sql
SELECT COUNT(*), MIN(season), MAX(season),
  COUNT(CASE WHEN strength_class = 'High' THEN 1 END) as high_count,
  COUNT(CASE WHEN strength_class = 'Medium' THEN 1 END) as medium_count,
  COUNT(CASE WHEN strength_class = 'Low' THEN 1 END) as low_count
FROM event_strength;
```
Expected (roughly): High ≈ 5–15% of events, Medium ≈ 40–60%, Low ≈ 30–50%. If 90%+ are High or Low, the algorithm may have edge-case problems.

**Query B: Named event spot check**
```sql
-- Check specific known events
SELECT e.event_name, es.strength_class, es.avg_team_rating, es.strength_percentile,
       es.competitive_spread, es.upset_rate, es.team_count
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE e.event_name ILIKE '%ECNL%' 
   OR e.event_name ILIKE '%GA National%'
   OR e.event_name ILIKE '%Surf Cup%'
   OR e.event_name ILIKE '%Jefferson%'
ORDER BY es.strength_percentile DESC
LIMIT 20;
```
Manually compare the output against the lookup table from Step 1. Any major discrepancy (an ECNL National event rated Low, or a local friendly rated High) is a flag.

**Query C: Strength distribution by event tier**
```sql
-- Event strength vs event tier should correlate
SELECT e.event_tier, es.strength_class, COUNT(*) as event_count
FROM event_strength es
JOIN events e ON e.id = es.event_id
GROUP BY e.event_tier, es.strength_class
ORDER BY e.event_tier, es.strength_class;
```
Expected: `ga` and `mlsnext` tier events should skew High/Medium. `state` and `local` tier events should skew Low. A `ga` event with Low strength is suspicious; a `local` event with High strength is suspicious. Document the acceptable cross-tab vs red flags.

**Query D: Minimum viable data quality check**
```sql
-- Events with suspiciously high or low team count
SELECT e.event_name, es.team_count, es.strength_class, es.avg_team_rating
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE es.team_count < 4  -- less than 4 teams = probably not a real event
ORDER BY es.team_count ASC
LIMIT 20;
```
Events with fewer than 4 teams should not have a strength classification — they're too small to be meaningful. If many such events appear, the minimum team count filter may not be working.

### Step 3: Define Pass/Fail Criteria

Write numeric thresholds Presten can apply to decide if the output is ready for the demo:

| Check | Pass Criteria | Fail Criteria |
|---|---|---|
| High event count | 5–15% of all classified events | <3% or >25% |
| ECNL events classified | ≥80% as High or Medium | Any ECNL National event as Low |
| GA tier event classification | ≥90% as High or Medium | >10% of GA events as Low |
| State/local tier | ≥70% as Low or Medium | >20% of state events as High |
| Events with team_count < 4 | 0 classified events under 4 teams | Any events with team_count < 4 classified |
| avg_team_rating spread | High events avg >200 pts above Low events | High events avg ≤ Low events |

If all pass: event strength is ready for the demo.
If any fail: identify the specific miscalibration before going live.

### Step 4: Identify the Demo Event

The DSS Demo Spec requires one specific High-strength event to feature in the demo. This query identifies the best candidate:

```sql
SELECT e.event_name, es.avg_team_rating, es.strength_class, es.strength_percentile,
       es.competitive_spread, es.upset_rate, es.team_count
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE es.age_group = 'U13G'
  AND es.strength_class = 'High'
  AND es.team_count >= 8
  AND e.season >= '2025-2026'
  AND e.event_name NOT ILIKE '%friendly%'
ORDER BY es.strength_percentile DESC
LIMIT 10;
```

From the output: pick the event with the most recognizable name to the DSS audience (NJ/NE club directors). Ideally an ECNL or GA event with a well-known name.

Document the selection criteria:
- Must be recognizable to NJ/NE audience
- Must have clean metrics (all four fields non-null: avg_rating, spread, upset_rate, team_count)
- Prefer events the target audience's teams have likely competed in

Record the selected event name in this deliverable so it's confirmed before the DSS data validation plan runs (May 14).

### Step 5: Write the Red Flag Protocol

If validation fails, what does Presten investigate?

**Scenario A: Too many High events (>25%)**
Likely cause: the percentile threshold for "High" is set too low. Check the `strength_percentile` values of the High events — if many are below p85, the percentile cutoff needs to be raised to p90 or p95.

**Scenario B: ECNL National event classified as Medium or Low**
Likely cause: that event has unusual data — very few teams entered (e.g., due to DB coverage gaps) or avg_team_rating is suppressed by unranked teams in the field. Check: does the event have the expected 32–64 teams? Do the team ratings look realistic?

**Scenario C: avg_team_rating for High events is lower than expected**
Likely cause: games from teams without ratings (new teams) are pulling down the average. Check whether unranked teams are included in the avg_team_rating computation.

For each scenario: write the diagnostic query and the recommended fix.

## Deliverable

Write the validation spec to:
`02 - Tiger Tournaments/Projects/Rankings/Event Strength Post-Compute Validation — 2026-04-24.md`

The document must:
- Define the expected distribution (Step 1 lookup table + Step 3 pass/fail)
- Include all four sanity check queries (Steps 2A–2D) with expected outputs
- Include the demo event selection query and documented selection criteria
- Include the red flag protocol (Step 5)

## Definition of Done

- Expected event strength distribution is defined (High/Medium/Low % targets)
- Pass/fail criteria are numeric for each check
- Four sanity check queries are schema-correct
- Demo event selection query is present with explicit selection criteria
- Red flag protocol covers 3 failure scenarios
- Summary in briefing: event strength validation spec ready; demo event will be confirmed after Phase 2 runs

## Why This Matters

Event Strength is a 7-hour implementation session. The DSS demo relies on it. Running validation after implementation takes 30 minutes and either confirms the output is demo-ready or gives Presten a specific issue to fix. Without this spec, the decision to show Event Strength at DSS is made on gut feel. With it, it's made on data.

## References

- `02 - Tiger Tournaments/Projects/Rankings/Event Strength Ratings — Implementation Plan.md` — implementation plan (what's being built)
- `02 - Tiger Tournaments/Projects/Rankings/Event Strength Ratings Design.md` — design rationale (strength_class thresholds)
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Demo Spec — May 2026.md` — demo event requirements (High event needed for Step 3)
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Data Readiness Validation Plan — May 14-15.md` — query 2B in that document uses similar logic
