---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-05-17
priority: medium
status: completed
completed: 2026-04-27
topic: ga-coverage-audit-boys-query-package
---

# Task: GA Coverage Audit — Boys Cross-League Win Rate Query Package

## Context

The April 23 GA Coverage Audit (`Rankings/GA Data Coverage Audit — 2026-04-23.md`) designed a methodology for measuring Boys cross-league win rates — specifically the comparison between ECNL Boys and GA Boys to understand whether the 20-point calibration gap is reflected in actual performance differences. The audit designed the methodology but left result cells empty: "GA Coverage Audit (April 23) result cells unfilled — the audit designed the methodology for Boys cross-league win rates but never recorded results."

ELO identified this in the April 27 ECNL Boys vs GA Boys Win-Rate Gap Investigation as a documentation gap, not a data gap: "The data likely exists, the window to execute it properly is known." The correct execution window is May 17–30 during the Boys Brier analysis.

But if ELO arrives at May 17 and has to rebuild the query execution package from scratch — rediscover what SQL the audit specified, validate the query syntax, and define the expected output format — that's 1–2 hours of the most time-constrained window in the sprint lost to preparation that could be done now.

This task closes that gap. ELO prepares the query package in full so May 17 starts with execution, not development.

## Task

Produce a validated, ready-to-run query execution package for the Boys cross-league win rate analysis. The package must be self-contained: anyone picking it up on May 17 should be able to execute it without reading the original audit document.

## Document Must Include

### Section 1: Analysis Objective

One paragraph: what this analysis measures, why it matters for Boys calibration, and what the expected finding is (ELO's current hypothesis). The 20-point ECNL Boys → GA Boys calibration gap is the subject; the win rate comparison is the measurement method.

### Section 2: Input Requirements

- Database tables used (specify exact table names from Evo Draw schema)
- Filters applied: date range, age groups (U13B–U18B), leagues included (ECNL Boys, GA Boys — specify exact league name strings as they appear in the DB)
- Minimum game count threshold for a team to be included (sample size floor)
- How "cross-league" games are defined (team A in ECNL Boys vs. team B in GA Boys — specify if these are only tournament games where cross-league matchups occur, or if league play is included)

### Section 3: Query Set

Four queries, fully written and ready to run:

**Query 1 — Cross-League Game Count**
How many games exist in the dataset where an ECNL Boys team played a GA Boys team (or vice versa)? If this number is below [ELO's minimum threshold for a valid analysis — define it], the win rate analysis should not proceed.

**Query 2 — Win Rate by League**
For cross-league games only: what is the win rate for ECNL Boys teams vs. GA Boys teams? Output: home_league | away_league | games | wins | losses | draws | win_pct

**Query 3 — Rating Distribution Comparison**
For teams with ≥5 cross-league games: what is the median and IQR of ratings for ECNL Boys vs GA Boys teams? This confirms whether the 20-point gap is consistent with the calibration values or diverges.

**Query 4 — Per-Age-Group Breakdown**
Repeat Query 2 split by age group (U13B through U18B). Check if the win rate gap is consistent across age groups or concentrated in one.

### Section 4: Interpretation Guide

For each query, define what ELO looks for:
- Expected result (what the current 20-point gap predicts)
- What a confirming result looks like (gap is validated — no recalibration needed)
- What a disconfirming result looks like (win rates don't match calibration — flag for Brier analysis)
- Threshold for escalation to SENTINEL (e.g., ECNL Boys win rate vs GA Boys < 45% across all age groups — suggests gap is too small; > 70% — suggests gap may be too large)

### Section 5: Output Filing Protocol

When the queries are run on May 17:
- File results to: `Rankings/GA Coverage Audit — Boys Win Rate Results — 2026-05-17.md`
- Fill the result cells in the original audit document (update `Rankings/GA Data Coverage Audit — 2026-04-23.md` with actual numbers)
- Reference this results file in the Boys Brier Analysis when interpreting ECNL Boys vs GA Boys Brier scores

## Deliverable

File to: `Rankings/GA Coverage Audit — Boys Query Execution Package.md`

Update this task to `status: completed` when filed.

## Notes

The due date is May 17 — the start of the Boys Brier window. ELO should complete this before that date, not during it. The ideal completion date is before May 9 (DSS readiness assessment) so SENTINEL can confirm the query package is ready as part of the Brier window pre-entry checklist.

If ELO determines from vault research that cross-league ECNL Boys vs GA Boys games are too rare to produce a valid win rate analysis (i.e., these leagues rarely play each other in tracked events), document that finding and propose an alternative measurement. The analysis objective is to validate or question the 20-point gap — if win rate comparison isn't the right tool, identify the right tool.
