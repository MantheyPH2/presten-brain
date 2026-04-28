---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
priority: medium
status: completed
completed: 2026-04-27
due: 2026-05-09
---

# Task: Post-DSS Calibration Priority Queue

## Context

DSS is May 16. After May 16, ELO's calibration work will enter a new phase — no longer constrained by DSS readiness, focused instead on increasing ranking accuracy and closing the gaps that were acknowledged but deferred. Several calibration items are already queued or referenced across task files but have never been consolidated into a single priority-ordered work queue with explicit success criteria.

The post-DSS sprint should not begin without a clear queue. This document is that queue.

## Task

Produce the ELO post-DSS calibration priority queue. This is a planning document, not implementation. Draw entirely from existing task files, briefings, and deliverables — do not conduct new analysis.

## Document Must Include

### Section 1: Open Calibration Issues (as of April 27)

List all known calibration issues or gaps that are deferred until post-DSS. For each:
- Description of the issue
- Which age groups / leagues are affected
- Why it was deferred (DSS timing, data gap, Boys Option A dependency, etc.)
- Current evidence strength (is there an active miscalibration signal, or is this theoretical?)

Known items to check (not exhaustive):
- ECNL Boys vs GA Boys win-rate gap (Boys calibration weakest link — see Boys ECNL Scope doc)
- Boys Option A outcome (if FAILS on April 29, GA Boys calibration is under revision)
- Tier 2 undefined leagues calibration (task-2026-04-27-tier2-undefined-leagues-calibration-recommendation.md)
- Boys club rankings (deferred pending girls validation)
- Event strength Phase 2 (scope doc exists, authorization not yet granted)
- Any items surfaced in the post-fix distribution analysis (April 29)

### Section 2: Prioritized Work Queue

Rank the issues from Section 1 in priority order using these criteria:
1. **Impact** — how many teams/age groups affected?
2. **Evidence** — active signal vs theoretical concern?
3. **Sequencing dependency** — does this block other work?
4. **Effort** — estimated hours to resolve?

Format: ranked table (Priority | Issue | Impact | Evidence | Dependency | Est. Hours)

### Section 3: Sprint Plan Outline

For the top 3 issues: a 2–3 sentence description of what ELO would do to resolve it, what data is needed, and what the success criterion looks like.

### Section 4: What Presten Needs to Decide

List any items that require a Presten decision before ELO can execute (calibration value changes, Boys Option A verdict, authorization for Phase 2, etc.).

## Deliverable

File to: `Rankings/Post-DSS-Calibration-Priority-Queue-2026-04-27.md`

Update this task to `status: completed` when filed.

## Notes

This document should be written to be handed directly to Presten after DSS. It should give Presten confidence that ELO knows exactly what to work on next and in what order, without requiring Presten to re-derive the context from prior briefings.

Due date is May 9 (before DSS) so Presten can review and redirect the queue before the sprint begins.
