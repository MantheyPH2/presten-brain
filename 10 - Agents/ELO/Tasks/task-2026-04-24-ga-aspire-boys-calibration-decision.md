---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
priority: urgent
due: 2026-04-25
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md"
---

# Task: Boys GA ASPIRE Calibration Decision

## Context

ELO's 11:16 briefing surfaced an unresolved structural question: the April 28 GA ASPIRE UPDATE SQL reclassifies events from `event_tier = 'ga'` to `event_tier = 'ga_aspire'` based on event name. This UPDATE is gender-agnostic. If Boys GA ASPIRE events exist (events with "ASPIRE" in the name that hosted Boys games), they will be reclassified too.

The problem: there is no Boys GA ASPIRE calibration value defined anywhere in the vault. ELO documented in the 11:16 briefing:

> "If `compute-rankings.js` receives a Boys event with `event_tier = 'ga_aspire'` and has no Boys-specific calibration for that tier, it will either: (a) use the Girls value (100 — which happens to equal Boys GA anyway, so no practical impact), or (b) fall through to a default."

This ambiguity must be resolved before April 28. If Boys GA ASPIRE events exist and the calibration engine has no Boys-specific handler for `ga_aspire`, the fix may silently produce incorrect Boys rankings without any error.

This task: make the decision. Document it. Add the resulting value (or "no Boys-specific value needed") to League Hierarchy.

## What to Do

### Step 1: Determine If Boys GA ASPIRE Events Exist

Write the SQL to check. (You cannot run it — provide it as a query Presten runs in the April 28 pre-fix session.)

```sql
SELECT g.gender, COUNT(DISTINCT g.id) AS game_count, COUNT(DISTINCT e.id) AS event_count
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_name ILIKE '%ASPIRE%'
  AND e.event_tier = 'ga'
GROUP BY g.gender;
```

Document in your deliverable: "If this query returns Boys game_count = 0, skip Steps 2–3. The UPDATE is safe."

### Step 2: Define the Boys GA ASPIRE Calibration Value

If Boys GA ASPIRE events exist, the `ga_aspire` tier needs a Boys-specific calibration value. Three defensible options:

**Option A: Boys GA ASPIRE = Boys GA = 100**
- Rationale: Boys GA is already at 100 (lower than Girls GA at 140). The ASPIRE correction was needed for Girls because Girls GA was overcalibrated at 140 relative to ECNL at 130. For Boys, GA is already below ECNL (100 < 120), so there may be no overcalibration problem for Boys GA ASPIRE to solve.
- Risk: If Boys GA ASPIRE competitions are genuinely weaker than Boys GA, assigning cal=100 still overcalibrates Boys GA ASPIRE teams.

**Option B: Boys GA ASPIRE = ~70**
- Rationale: Structural parallel to Girls (Girls GA ASPIRE = 100 is below Girls GA = 140; the same proportional reduction for Boys would put Boys GA ASPIRE at ~70).
- Risk: Unvalidated. No Boys Brier analysis has been run on GA ASPIRE events. 70 is a design assumption, not an empirical finding.

**Option C: Exclude Boys events from the April 28 UPDATE**
- Rationale: If Boys GA ASPIRE calibration is unresolved, scope the April 28 UPDATE to Girls-only events. Add a WHERE clause filtering by gender-linked logic (e.g., exclude events that hosted only Boys games from the reclassification).
- Risk: Requires identifying a gender-linkage in the events table. If events host both Boys and Girls games (mixed-gender tournaments), partial exclusion is complex.

**ELO's recommendation:** State which option is correct and why. Base the recommendation on the structural calibration logic already established in `Rankings/League Hierarchy.md` and `Rankings/Ranking Engine.md`. If Option A is correct, state that explicitly — "Boys GA ASPIRE calibration = 100, same as Boys GA, because the overcalibration problem was Girls-specific." This closes the question.

### Step 3: Update League Hierarchy

Add a Boys GA ASPIRE row to `League Hierarchy.md`:

| Circuit | Gender | Calibration Value | Validation Status | Note |
|---------|--------|------------------|-------------------|------|
| GA ASPIRE | Boys | [your decision] | [status] | April 28 decision |

If your recommendation is Option C (exclude Boys from UPDATE), document the row as: `calibration: undefined (excluded from April 28 UPDATE scope — reclassify Boys events in separate session after Brier analysis)`.

### Step 4: Write the April 28 Pre-Fix Check

Write a short addition for the April 28 Reference Card (FORGE will integrate it into Section 3). Specifically: a 2-line pre-condition that Presten checks before running the UPDATE:

```
Pre-condition: Run Boys GA ASPIRE check query (Task SQL Step 1).
If Boys game_count = 0: proceed. If Boys game_count > 0: [action per your recommendation].
```

## Deliverable

Write: `02 - Tiger Tournaments/Projects/Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md`

Contents:
1. Boys GA ASPIRE existence query (copy-paste ready)
2. Three options with SENTINEL-recommended choice stated explicitly
3. The Boys GA ASPIRE calibration value (or "not applicable — excluded from UPDATE")
4. League Hierarchy row to add
5. Pre-condition text for FORGE to integrate into the April 28 Reference Card

## Definition of Done

- Decision document written at the specified path
- Boys GA ASPIRE calibration value is stated explicitly (not "unknown")
- Reasoning ties to existing Boys GA calibration value (100) and the structural logic of why the Girls fix was needed
- League Hierarchy updated with the Boys GA ASPIRE row
- Summary in briefing: "Boys GA ASPIRE calibration decision: [value or exclusion]. League Hierarchy updated."

## Why This Matters

If Boys GA ASPIRE events exist and the calibration engine silently falls through to a Girls default or an error, the April 28 fix could silently miscalibrate Boys rankings. ELO is the calibration owner. This decision needs to be made before April 28, not discovered on April 28.

## References

- `02 - Tiger Tournaments/Projects/Rankings/League Hierarchy.md` — Boys GA row for comparison
- `02 - Tiger Tournaments/Projects/Rankings/Ranking Engine.md` — calibration map structure
- `10 - Agents/ELO/Briefings/2026-04-24-11:16.md` — ELO's Q1 (priority HIGH — needed before April 28)
- `task-2026-04-24-ga-aspire-update-sql.md` — companion task; Section 3 of that task cites this decision
