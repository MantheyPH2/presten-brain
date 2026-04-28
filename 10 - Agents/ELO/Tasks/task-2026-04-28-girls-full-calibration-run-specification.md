---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
due: 2026-05-07
priority: high
status: completed
completed: 2026-04-28
tags: [elo, task, calibration, girls, brier, post-dss, sprint]
---

# Task: Girls Full Calibration Run — Post-DSS Specification

## What to Build

A complete run specification for the Girls full calibration run planned for Week 1 of the post-DSS calibration sprint. This document defines what changes, what stays fixed, the expected Brier score improvement, and the pass/fail criteria for the run. Filing it now, before the DSS session, ensures ELO can execute the run immediately post-DSS without design time.

## Why This Exists

The post-DSS calibration sprint plan (`task-2026-04-27-post-dss-calibration-sprint-plan.md`) designates Girls full calibration as the Week 1 primary deliverable. The sprint plan says it will be done — this spec says how. Without the spec:
- ELO enters the post-DSS sprint with a goal but no defined parameters
- SENTINEL cannot authorize the calibration run without knowing what changes are being made
- The May 9 calibration dashboard (`task-2026-04-28-pre-may9-calibration-state-dashboard.md`) cannot confidently list Girls calibration as MET

This spec is the pre-authorization document. SENTINEL pre-authorizes the Girls calibration run based on this spec. ELO does not need a separate authorization request during the sprint.

## Required Sections

### 1. Calibration Run Scope

**What is changing in this run:**
List each parameter ELO intends to modify. For each:
- Parameter name (e.g., K-factor for Girls U15G)
- Current value
- Proposed value
- Source of proposed value (Brier score analysis, manual calibration, league hierarchy calibration)
- Expected direction of rating impact: upward / downward / redistribution

**What is NOT changing:**
List any Girls calibration parameters ELO is explicitly holding fixed and why (e.g., parameters that depend on Boys Option A outcome, parameters requiring more data).

### 2. Prerequisites

Required conditions before the run executes:
- [ ] Event Strength Phase 1 completed (event tier values locked)
- [ ] ECNL migration implemented (correct league classification for ECNL teams)
- [ ] Girls data freshness: minimum [N] games ingested post-April-28
- [ ] Tier 2 undefined leagues calibration applied (reference: `task-2026-04-27-tier2-calibration-implementation-sql.md`)

State clearly: if any prerequisite is not met when the sprint starts, which parameters must be deferred.

### 3. Expected Brier Score Improvement

- Current Girls Brier score baseline: [value or PENDING from post-April-28 Brier run]
- Target Girls Brier score after calibration run: [range]
- Minimum acceptable improvement to call the run successful: [delta]
- Source of expectation: [reference doc or analysis used to estimate]

If Brier score baseline is not yet available (depends on post-April-28 data): state that ELO will update this section after the April 29 session.

### 4. Run Execution Steps

Numbered steps ELO executes in sequence:
1. Snapshot current ratings before run (backup point)
2. Apply parameter changes from Section 1 (in what order)
3. Trigger rankings recompute
4. Extract Brier score post-recompute
5. Compare against Section 3 targets
6. Spot-check: pull top-20 Girls U14G and U16G and confirm ranking face-validity (are top teams recognizable programs?)
7. File run results document (format defined in Section 5)

### 5. Pass/Fail Criteria

**RUN PASSES if:**
- Brier score improvement meets or exceeds minimum delta from Section 3
- Spot-check top-20 lists contain no obviously wrong teams (< 3 anomalous entries)
- No ratings moved by more than [N] points unexpectedly (define threshold from continuity spec)

**RUN FAILS if:**
- Brier score worsens vs. baseline
- Spot-check reveals systematic miscalibration (> 3 anomalous entries in top 20)
- Ratings shift exceeds deviation threshold without explanation

**If FAIL:** ELO files a failure report to SENTINEL within 30 minutes, reverts to pre-run snapshot, and proposes revised parameters. No re-run without SENTINEL authorization.

### 6. Post-Run Filing

After a successful run, ELO files:
- Girls Full Calibration Run — Results Report (`Rankings/Girls Full Calibration Run — Results Report.md`)
- Updates May 9 Calibration State Dashboard: Girls calibration row → MET
- Notifies SENTINEL that Girls calibration is closed for May 9 gate

## File When Done

This specification document: `02 - Tiger Tournaments/Projects/Rankings/Girls Full Calibration Run — Post-DSS Specification.md`

Mark this task `status: completed` in frontmatter after the spec is filed (before the run executes — filing the spec is the task, not executing the run).
File spec to SENTINEL for pre-authorization. SENTINEL's approval in the next briefing cycle is sufficient — no separate authorization request needed.
