---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-05-09
priority: medium
status: completed
completed: 2026-04-27
topic: boys-brier-pre-may17-readiness-check
---

# Task: Boys Brier Analysis — Pre-May-17 Readiness Check

## Context

The Boys Brier analysis is scheduled for the May 17–30 window. The scope spec (`task-2026-04-25-boys-brier-analysis-scope-spec.md`) and the post-DSS calibration priority queue (filed 2026-04-27) both reference this window as the primary mechanism for resolving Boys calibration questions, including the ECNL Boys vs GA Boys win-rate gap. The window is approximately 3 weeks away.

A Brier analysis that starts May 17 without its prerequisites confirmed wastes session time. This task ensures that ELO enters May 17 ready to execute, not ready to plan.

## Task

Produce a readiness checklist for the Boys Brier analysis window. Draw entirely from existing task files and deliverables — no new analysis.

## Document Must Include

### Section 1: What the Boys Brier Analysis Requires
For each requirement, state whether it is currently MET or NOT MET as of April 27:

- [ ] Scope spec completed (which age groups, which calibration questions, what success criteria)
- [ ] SQL queries written for Boys Brier calculation
- [ ] Pre-analysis Boys baseline snapshot plan confirmed (when does the snapshot get taken?)
- [ ] Boys Option A outcome accounted for (if PASS: GA Boys cal=100 confirmed; if FAIL: GA Boys cal is under revision — two different Brier interpretations)
- [ ] ECNL Boys vs GA Boys win-rate gap measurement SQL ready (filed in win-rate gap investigation, 2026-04-27)
- [ ] Threshold for "Boys calibration confirmed" defined (what Brier score or delta counts as a PASS?)
- [ ] Decision tree for each outcome documented (if Brier shows anomaly — what calibration change?)

### Section 2: What May Change Before May 17
- Boys Option A verdict (April 29) — how does each outcome change the Brier analysis scope?
- May 1 pipeline deployment — does new game volume affect the Brier baseline?
- Any other April 29 gate outcomes that affect Boys data quality?

### Section 3: Gap Inventory
- What is NOT yet ready as of April 27?
- For each gap: what task or action closes it, and when should that happen (before April 29, before May 9, before May 17)?

### Section 4: May 17 Entry Conditions
State the exact conditions ELO needs to confirm on May 17 before beginning Brier computation:
- All items in Section 1 confirmed MET
- Boys Option A verdict filed and calibration plan updated accordingly
- Any other explicit go/no-go condition

## Deliverable

File to: `Rankings/Boys-Brier-Pre-May17-Readiness-Check-2026-04-27.md`

Update this task to `status: completed` when filed.

## Notes

This is a planning document due before DSS. Presten should be able to read Section 4 (May 17 Entry Conditions) and know exactly what to confirm on May 17 morning before ELO begins. Keep it under 2 pages.
