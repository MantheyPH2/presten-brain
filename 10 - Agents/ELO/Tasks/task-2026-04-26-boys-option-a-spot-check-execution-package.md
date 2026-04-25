---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-25
priority: high
due: 2026-05-02
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Option A Spot Check — Presten Execution Package.md"
---

# Task: Boys Option A Spot Check — Presten Execution Package

## Context

The Boys Calibration Status Summary (filed April 25) confirms: 0 Boys age groups are Brier-validated. All Boys age groups are unvalidated. The DSS demo risk is Medium, contingent on passing the "Option A spot check" — 3 queries, ~20 min, before May 9.

Currently the spot check is described in multiple places (Boys Calibration Status Summary, Boys Calibration Option A Interpretation, Boys Brier Analysis Scope Spec) but there is no single document Presten can open and execute. This task is that document.

Format it identically to `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md` — same structure, same copy-paste standard, same pre-run gate, same output file template.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Boys Option A Spot Check — Presten Execution Package.md`

### Required Sections

#### Pre-Run Gate
One condition Presten must confirm before running: April 28 GA ASPIRE fix applied and recompute completed. If not, Boys ratings may still reflect Girls rating contamination from GA ASPIRE events. Run after April 29 verification.

#### Query 1 — Boys vs. Girls Average Rating by Age Group
Compares Boys and Girls average ratings for U13, U14, U15, U16, U17. Rationale: Boys and Girls ratings should differ by ≤100 points for same age group, given similar game volumes and calibration logic. A divergence > 100 points signals either a coverage artifact or a miscalibration.

PASS: all age groups within 100 points.
FAIL: any age group diverges > 100 points → flag to SENTINEL, Boys club rankings cannot be shown at DSS.

Save output as: `02 - Tiger Tournaments/Projects/Reports/boys-option-a-query1-[date].md`

#### Query 2 — Boys GA Game Volume Validation
Counts games per Boys age group where `event_tier = 'ga'`. Rationale: Boys GA calibration (cal=100) is set conservatively. If Boys GA game volume is < 500 for any key age group (U13B–U17B), the calibration is underpowered and could drift unpredictably.

PASS: all U13B–U17B age groups have ≥ 500 Boys GA games.
FAIL: any age group < 500 games → flag to SENTINEL. Boys GA calibration for that age group should not be cited as validated.

Save output as: `02 - Tiger Tournaments/Projects/Reports/boys-option-a-query2-[date].md`

#### Query 3 — MLS NEXT Boys Rating Distribution
Checks that Boys MLS NEXT teams (cal=160/135, the best-validated Boys calibration) have a rating distribution consistent with expectations: median ≈ 1400–1600, top 10% ≥ 1700, bottom 10% ≤ 1300.

PASS: distribution within expected range for current season.
FAIL: median or top/bottom decile outside range by > 100 pts → flag to ELO. Indicates MLS NEXT cal values may need adjustment.

Save output as: `02 - Tiger Tournaments/Projects/Reports/boys-option-a-query3-[date].md`

#### Decision Table

| Result | Action |
|--------|--------|
| All 3 PASS | Boys club rankings are demo-safe. Include Boys in Club Rankings implementation plan. |
| Query 1 FAIL only | Boys club rankings excluded from DSS. Girls-only. Notify SENTINEL. |
| Query 2 FAIL on any age group | Note in demo script: Boys GA coverage is thin for [age group]. Do not claim Boys GA is validated. |
| Query 3 FAIL | Contact ELO immediately. ELO will assess whether Boys MLS NEXT calibration needs re-calibration before DSS. |

#### ELO Action After Results
Once Presten sends results, ELO will: (a) classify each result per thresholds, (b) update DSS Feature Readiness Tracker (Club Rankings row — Boys cell), (c) file updated assessment in next briefing to SENTINEL.

## Quality Standard

Match `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md` exactly in structure and tone. Every query copy-paste ready. No interpretation required from Presten — just run queries, save output files, send to ELO.

Run window: April 29 (after GA ASPIRE recompute confirmed) through May 5.
