---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
due: 2026-04-30
priority: high
status: completed
completed: 2026-04-28
tags: [elo, task, boys, option-a, calibration, fallback, dss]
---

# Task: Boys Option A — Fail Recovery Brief

## What to Build

A pre-written recovery brief ELO files within 30 minutes of a Boys Option A FAIL verdict on April 29. The brief covers all downstream consequences of a FAIL and the specific actions ELO takes in each area. Pre-writing this now means a FAIL on April 29 produces an immediate, structured response — not a gap in the record.

## Why This Exists

Boys Option A has a binary outcome. The PASS path is well-documented: `task-2026-04-27-boys-option-a-post-pass-implementation-timeline.md` and `task-2026-04-27-boys-option-a-change-execution-protocol.md` cover it. The FAIL path has no equivalent document. If Option A fails, ELO must simultaneously update 5+ documents and notify SENTINEL — under session pressure, without a plan. This brief is written in advance so execution is instant.

## Trigger

File this brief on April 29 within 30 minutes of reading Boys Option A query results that meet the FAIL criteria from `task-2026-04-27-boys-option-a-pre-verdict-evidence-checklist.md`. If April 29 results are inconclusive (YELLOW, not a hard FAIL), this brief is NOT filed — the verdict doc covers that case.

## Required Sections

### 1. Verdict Summary
One paragraph. State the specific query result(s) that triggered FAIL. Reference the FAIL threshold from the pre-verdict evidence checklist. Example: "Boys Option A: FAIL. [Metric] = [value], below [threshold] required for PASS. Verdict filed: April 29."

### 2. DSS Feature Readiness Tracker — Update Instructions
Specific update ELO makes to `Rankings/DSS Feature Readiness Tracker.md`:
- Boys calibration row: change status to DEFERRED
- Boys Option A row: change status to FAIL (date: April 29)
- DSS Club Rankings — Boys row: change to NOT INCLUDED (OPTION A FAIL)
- Estimated re-attempt window: fill in `[POST-DSS, TENTATIVELY MAY 20+]`

### 3. Club Rankings — Boys Scope Impact
State explicitly: Boys Club Rankings is excluded from the DSS demo. Girls-only fallback activates per `Rankings/Club Rankings — Girls-Only Fallback Spec.md`. No new decisions required — the fallback spec is self-executing. ELO updates the Club Rankings May Launch Conditions document (filed 2026-04-28) to reflect Boys deferred status.

### 4. Calibration Sprint — Scope Revision
The post-DSS calibration sprint (April 27 task: `task-2026-04-27-post-dss-calibration-sprint-plan.md`) included a Boys branch. On FAIL, that branch is removed from Week 1:
- Week 1 scope: Girls full calibration run only (already planned)
- Boys removed from Week 1 and Week 2 sprint
- Re-attempt criteria: Boys Option A query pack re-run when ≥ 20 additional boys games have been ingested (estimated: post-DSS, May 15–20 window)

ELO does NOT rewrite the sprint plan — it appends a "Boys Option A FAIL — Scope Revision" section to the existing plan document.

### 5. Event Strength Phase 1 — Boys Impact
State whether Boys teams are still included in Phase 1 event strength computation despite Option A FAIL. They are — event strength applies to all teams regardless of calibration status. No change to Phase 1 scope required.

### 6. Next Boys Option A Attempt
State conditions for re-attempt:
- Minimum additional boys games ingested: 20+ (post-Stage 1 crawl and May 1 pipeline run)
- Earliest re-attempt: May 15 (if games accumulate)
- Decision authority: ELO proposes, SENTINEL authorizes session
- Presten notification: ELO files a queue item to SENTINEL with re-attempt readiness criteria

### 7. SENTINEL Notification
Pre-written queue entry ELO files immediately:
- Category: alert
- Priority: high
- Body: "Boys Option A FAIL on April 29. DSS Club Rankings scope reduced to Girls-only (fallback spec activated). Calibration sprint Boys branch deferred to May 15+ re-attempt window. No immediate action required from Presten. Girls Club Rankings on track for May 9 go/no-go."

## File When Done

`02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Fail Recovery Brief.md`

Mark this task `status: completed` in frontmatter.

If Option A PASSES on April 29: mark this task `status: completed` with a note "Option A PASS — brief not filed, not needed." Do not leave it pending.
