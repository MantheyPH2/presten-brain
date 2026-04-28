---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
priority: medium
status: completed
completed: 2026-04-27
due: 2026-05-05
---

# Task: ECNL Boys vs GA Boys Win-Rate Gap Investigation

## Context

The Boys ECNL Calibration Scope document (filed 2026-04-27) identified a specific weakness in the Boys calibration chain: the 20-point gap between ECNL Boys (cal=120) and GA Boys (cal=100) is based on hierarchy positioning logic, not a directly measured cross-league win rate. Every other Boys calibration relationship has at least indirect win-rate support. This one does not.

The `task-2026-04-23-ga-data-coverage-audit` was assigned to address this, but ELO noted it does not have visibility into whether that task's cross-league win-rate component was completed. This task closes the gap regardless of what the April 23 audit produced.

## Task

### Step 1: Check the April 23 GA Coverage Audit

Read `task-2026-04-23-ga-data-coverage-audit.md` and its associated deliverable (if filed). Determine whether the task measured a direct ECNL Boys vs GA Boys cross-league win rate. Document the finding in one sentence.

### Step 2: If the win-rate was measured — confirm or challenge the 20-point gap

If a cross-league win rate exists, apply the standard calibration formula and check whether the empirical result supports the 20-point gap between ECNL Boys (120) and GA Boys (100). Note any discrepancy.

### Step 3: If the win-rate was NOT measured — write the SQL to capture it

Write the SQL query needed to measure ECNL Boys vs GA Boys cross-league win rate from existing game data. The query should:
- Filter to games where one team is classified ECNL Boys (cal source = ecnl_boys) and the opponent is GA Boys (cal source = ga_boys)
- Compute ECNL Boys win rate
- Output: sample size, win rate, implied calibration delta, whether this supports or challenges the 20-point gap

### Step 4: Produce a verdict

State clearly: is the 20-point ECNL Boys → GA Boys gap (a) empirically confirmed, (b) empirically challenged, or (c) unmeasurable from current data? If (c), specify what data gap prevents measurement.

## Deliverable

File to: `Rankings/ECNL-Boys-GA-Boys-Win-Rate-Gap-Investigation-2026-04-27.md`

Update this task to `status: completed` when filed.

## Notes

This is not a DSS blocker. There is no active miscalibration signal and the gap is defensible on structural grounds. But the gap is the weakest link in Boys calibration and should be resolved or formally acknowledged before the post-DSS calibration sprint begins. If the win rate can't be measured from current data, the deliverable should specify what additional game data (source or volume) would make measurement possible.
