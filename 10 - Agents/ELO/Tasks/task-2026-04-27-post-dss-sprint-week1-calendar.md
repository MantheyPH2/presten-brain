---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-05-05
priority: medium
status: completed
completed: 2026-04-27
topic: post-dss-sprint-week1-calendar
---

# Task: Post-DSS Calibration Sprint — Week 1 Execution Calendar

## Context

ELO has produced:
- `Rankings/Post-DSS Calibration Priority Queue.md` — ordered list of calibration tasks for the post-DSS sprint
- `Rankings/Post-DSS Calibration Roadmap.md` — high-level roadmap
- `Rankings/Post-DSS Calibration Sprint Plan.md` — sprint plan

What does not exist: a day-by-day execution calendar for Week 1 (May 17–24). The sprint plan and priority queue say what to do in what order, but not which day each item starts, how long each item takes in hours, or what Presten needs to have approved before each item can begin.

SENTINEL cannot hand a coherent session plan to Presten for the May 17 sprint without this calendar. The current documents answer "what" but not "when" or "how long."

## Task

Produce `Rankings/Post-DSS Sprint — Week 1 Execution Calendar.md`.

### Section 1: Pre-Sprint Approval Requirements

List every item that requires Presten's approval before May 17. For each:
- Item description
- Which calibration work it blocks
- Where the approval document lives
- Status: APPROVED / PENDING / NOT YET SUBMITTED

If any approval is currently PENDING or NOT YET SUBMITTED, flag it for SENTINEL. These are blockers.

### Section 2: Day-by-Day Calendar (May 17–24)

For each day, specify:
- **Which ELO task(s) run this day**
- **Estimated hours** (be specific — e.g., "2 hours of query execution + 1 hour filing" vs. "3 hours")
- **Dependencies** (what must have happened on a prior day before this task starts)
- **Presten session required?** YES / NO — if YES, what Presten must do and approximate time needed

Use the priority queue ordering. Do not re-sequence without stating the reason.

Target format:

```
May 17 (Day 1)
- Task: Boys Option A calibration implementation (if Option A PASSED April 29)
  Hours: 2h ELO execution + 30m Presten runs compute-rankings.js
  Dependency: Boys Option A verdict PASS (filed April 29); compute-rankings.js accessible
  Presten session: YES — run compute-rankings.js after ELO files SQL

- Task: Tier 2 calibration SQL + monitoring queries (see tier2 monitoring plan)
  Hours: 1h SQL review + Presten executes
  Dependency: None (can run same session as Boys if time allows)
  Presten session: YES — runs SQL; ELO monitors output
```

### Section 3: Contingency Slots

Build in at least one contingency day (no planned tasks). State where it falls and what it covers: investigation of unexpected rating shifts, re-runs if monitoring queries show Yellow/Red, or Presten availability slip.

### Section 4: Week 1 Success Definition

What does a successful Week 1 look like? State 3–5 measurable outcomes (e.g., "Boys Option A calibration applied and confirmed via monitoring queries," "Tier 2 calibration live with all 6 monitoring checks GREEN").

## Deliverable

File to: `Rankings/Post-DSS Sprint — Week 1 Execution Calendar.md`

Update this task to `status: completed` when filed.

## Notes

Due May 5 — SENTINEL needs this 12 days before the sprint begins. This gives time for Presten to review, adjust the calendar, and ensure all approvals are in place by May 14.

If any approval item (Section 1) is blocking and currently NOT YET SUBMITTED, ELO should file the approval request in ELO's Queue/ immediately and reference it from this document. Do not file the calendar without addressing all blocking approval items.
