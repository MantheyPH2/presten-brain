---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength — League Tier Coverage Audit.md"
---

# Task: Event Strength — League Tier Coverage Audit

## Context

ELO completed `Rankings/Event Strength Ratings — Implementation Plan.md`. Phase 1 of the implementation (7 hours total) assigns `event_tier` to all events in the database. The implementation plan describes the assignment logic, but it does not answer a prerequisite question: **are all leagues in the League Hierarchy already assigned an `event_tier`, and if not, what happens to events from unassigned leagues during Phase 1?**

This is ELO's responsibility because event_tier assignment drives calibration — an event with null or incorrect event_tier will produce a wrong strength rating, which will then propagate into rankings.

The Event Strength implementation session cannot run clean until this audit is done.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Rankings/Event Strength — League Tier Coverage Audit.md`

---

### Section 1: League Hierarchy Tier Assignment Coverage

Read `02 - Tiger Tournaments/Projects/Rankings/League Hierarchy.md`.

For every league in the hierarchy, verify it has an `event_tier` assignment. Produce a coverage table:

| League | Tier in Hierarchy | event_tier Assigned? | Notes |
|--------|-------------------|----------------------|-------|
| ECNL | 1 | Yes — `ecnl_regional` / `ecnl_national` | — |
| MLS NEXT | 1 | Yes — `mls_next_league` / `mls_next_showcase` | — |
| GA | 2 | ? | — |
| GA ASPIRE | 2 | ? (being re-assigned by April 28 fix) | — |
| USL Academy | 2 | ? | — |
| EDP | 2 | ? | — |
| Elite 64 | 2 | ? | — |
| NAL | 2 | ? | — |
| USYS State Cup | 3 | ? | — |
| [all other leagues in hierarchy] | ? | ? | — |

Adapt the table to include all leagues actually in the hierarchy. Do not add leagues that aren't in the hierarchy.

### Section 2: Null Tier Behavior — Phase 1 Gap Analysis

The Event Strength implementation plan assigns event_tier in Phase 1. For any league with a null or unassigned event_tier:

**Q1: What does Phase 1 do with events from leagues that have no tier assignment?**
- Are they skipped? Assigned a default? Assigned the wrong tier because the logic falls through?
- Read the Phase 1 implementation design from `Rankings/Event Strength Ratings — Implementation Plan.md` Section [relevant section] and state explicitly what happens.

**Q2: How many events in the DB come from leagues with unassigned tiers?**
Provide the SQL to count them:
```sql
SELECT l.league_name, COUNT(e.id) AS event_count
FROM events e
JOIN leagues l ON l.id = e.league_id
WHERE l.event_tier IS NULL
GROUP BY l.league_name
ORDER BY event_count DESC;
```
(Adapt to actual schema. If leagues aren't a separate table, adjust the join accordingly.)

If you cannot determine the exact schema from vault documents, describe what the query should accomplish and flag that Presten needs to verify the column names before Phase 1 begins.

**Q3: For each league with a null tier — what SHOULD the tier be?**
Using the League Hierarchy as the reference, assign a proposed event_tier value for each unassigned league. Justify each assignment briefly.

### Section 3: Remediation SQL

For each league that needs a tier assignment before Phase 1 can run correctly, write the UPDATE SQL:

```sql
-- Example:
UPDATE leagues SET event_tier = 'ga_regional' WHERE name ILIKE 'GA Soccer%';
UPDATE leagues SET event_tier = 'usys_state' WHERE name ILIKE 'USYS%State Cup%';
-- etc.
```

Or, if event_tier is on the events table rather than a leagues table, write the assignment logic at the event level.

This SQL must be run by Presten BEFORE starting Phase 1 of Event Strength implementation. Label it as a pre-flight requirement.

### Section 4: Phase 1 Pre-Flight Gate

Based on the audit, write the Phase 1 pre-flight gate — a one-time check Presten runs before starting the Event Strength session to confirm all events have a valid tier assignment:

```sql
-- Phase 1 gate: must return 0 rows before proceeding
SELECT COUNT(*) AS unassigned_events
FROM events e
JOIN leagues l ON l.id = e.league_id
WHERE l.event_tier IS NULL;
```

If this query returns > 0: apply the remediation SQL from Section 3 first. Rerun the gate until it returns 0.

State this gate as a hard prerequisite in the implementation plan by filing a queue item or flagging it in the Event Strength implementation plan directly.

### Section 5: GA ASPIRE Interaction Note

The GA ASPIRE fix (April 28) changes `event_tier` for GA ASPIRE events. If Phase 1 of Event Strength runs BEFORE the GA ASPIRE fix, it will assign event strength ratings to GA ASPIRE events using the wrong tier value. After the April 28 fix, those strength ratings would need to be recomputed.

State explicitly: **Phase 1 must run AFTER the April 28 GA ASPIRE fix, not before.** If the implementation plan does not already say this, write a note to SENTINEL to ensure FORGE includes this ordering constraint in the reference card.

## Definition of Done

- Audit written at `Rankings/Event Strength — League Tier Coverage Audit.md`
- Coverage table lists all leagues from the hierarchy with their event_tier assignment status
- Null tier behavior is stated explicitly (skip / default / wrong assignment)
- Remediation SQL written for all unassigned leagues
- Phase 1 pre-flight gate query is present
- GA ASPIRE ordering constraint is stated
- Summary in briefing: "Event Strength tier audit complete. [N] leagues lacked event_tier assignment. Remediation SQL ready. Phase 1 pre-flight gate written. GA ASPIRE must run first."

## Why This Matters

Phase 1 of Event Strength iterates over events in the DB and assigns tier values. If it silently skips or mislabels events from unassigned leagues, the resulting Event Strength ratings will be systematically wrong for those leagues — and the implementation plan gives no indication of this. The audit catches the gap before 7 hours of Presten's implementation time is spent on a partially-correct output.

## References

- `02 - Tiger Tournaments/Projects/Rankings/Event Strength Ratings — Implementation Plan.md` — Phase 1 design
- `02 - Tiger Tournaments/Projects/Rankings/League Hierarchy.md` — authoritative league list with tier and calibration values
- `10 - Agents/ELO/Tasks/task-2026-04-24-event-strength-post-compute-validation.md` — post-Phase 1 checks (downstream of this audit)
- `10 - Agents/ELO/Tasks/task-2026-04-24-june2-ecnl-migration-verification-spec.md` — ECNL event tier continuity context
