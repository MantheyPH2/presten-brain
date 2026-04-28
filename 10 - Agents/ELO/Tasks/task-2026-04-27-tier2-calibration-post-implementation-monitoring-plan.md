---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-05-05
priority: medium
status: completed
completed: 2026-04-27
topic: tier2-calibration-post-implementation-monitoring-plan
---

# Task: Tier 2 Calibration — Post-Implementation Monitoring Plan

## Context

ELO filed two related deliverables today (April 27):
- `task-2026-04-27-tier2-undefined-leagues-calibration-recommendation.md` — recommendation for what calibration values to assign to currently-undefined Tier 2 leagues
- `task-2026-04-27-tier2-calibration-implementation-sql.md` — the SQL to implement those values

What does not exist is a monitoring plan: after the Tier 2 calibration values are implemented, what queries confirm the implementation is correct, and what signals indicate a problem requiring rollback or recalibration?

Tier 2 leagues affect a meaningful share of mid-tier teams. A miscalibrated implementation — wrong tier assigned, wrong league name match, or unexpected interaction with existing calibration values — could silently degrade ranking accuracy without triggering an alert. ELO needs to know what to check and when.

This gap matters before DSS. If the Tier 2 fix is implemented in the May 17–30 sprint window (per the Post-DSS Calibration Priority Queue), and monitoring queries are not written, ELO arrives at the May 17 session with implementation SQL but no validation plan.

## Task

Produce a Tier 2 Calibration Post-Implementation Monitoring Plan. This document is used immediately after the Tier 2 SQL runs to confirm the implementation is correct.

## Document Must Include

### Section 1: What Was Implemented (Reference Summary)

One paragraph: what leagues are being recalibrated, what the calibration change is (old value → new value per tier, or range of values if multiple), and the expected direction of rating impact (teams in these leagues should shift in which direction by approximately how much).

Reference the implementation SQL and recommendation documents. Do not duplicate them — one-line summary is sufficient.

### Section 2: Immediate Post-Implementation Queries

Four to six queries to run within 24 hours of the SQL executing:

**Query 1 — Affected League Count**
How many leagues had their calibration value changed by this implementation? Expected: matches the count documented in the recommendation.

**Query 2 — Affected Team Count**
How many teams have games in the recalibrated leagues? This is the scope of the rating impact.

**Query 3 — Rating Shift Distribution**
After the next compute-rankings.js run: for teams with ≥5 games in Tier 2 leagues, what is the distribution of rating changes (delta from pre-implementation snapshot)? Expected: centered near the predicted shift range, no bimodal outliers.

**Query 4 — Tier 2 vs. Adjacent Tier Boundary Check**
For teams near the Tier 1/Tier 2 boundary and Tier 2/Tier 3 boundary: do their ratings now fall on the expected side of the boundary given their recent game results? Flag any team where the tier placement is inconsistent with win/loss record.

**Query 5 — No Unintended Scope**
Confirm that teams with zero games in the recalibrated leagues have ratings unchanged (delta = 0 after implementation).

**Query 6 — Top-100 Stability Check**
How many teams in the top 100 (per age/gender group most affected) changed rank position by ≥5 places? Positions in the expected range indicate calibration is working; major reshuffling at the top warrants review.

### Section 3: Thresholds and Flags

For each query, define:
- **Green** (implementation confirmed correct): specific value range or condition
- **Yellow** (investigate before proceeding): condition that warrants a second look but not rollback
- **Red** (escalate to SENTINEL): condition that suggests the implementation has an error

For Query 3, define explicitly: what magnitude of rating shift is too large or too small, given the recommendation's predicted range? A shift 2× the predicted maximum is a Red flag regardless of direction.

### Section 4: Rollback Criteria

Define the precise condition under which ELO would recommend rolling back the Tier 2 implementation. A rollback is a significant action — ELO should be specific about what would trigger it. Example conditions:
- Query 5 shows >0 teams with unexpected rating changes (scope error in the SQL)
- Query 3 shows a bimodal distribution (some teams shifted +40, others shifted –20) suggesting two different bugs
- Top-100 reshuffling is >20 positions for ≥3 teams in a single age/gender group

Note: rollback recommendation goes to SENTINEL, not direct execution. ELO flags; SENTINEL decides.

### Section 5: 1-Week Follow-Up Check

After the first pipeline run post-implementation: one query to confirm the Tier 2 calibration values are being correctly applied to new games ingested after the implementation date (not just retroactively). A simple count: games ingested in the first week with Tier 2 league assignments — confirm the count is non-zero and the tier assignments match expectations.

## Deliverable

File to: `Rankings/Tier 2 Calibration — Post-Implementation Monitoring Plan.md`

Update this task to `status: completed` when filed.

## Notes

Due date is May 5. The Tier 2 implementation is scheduled for the May 17–30 sprint window (per the Post-DSS Calibration Priority Queue). The monitoring plan should be ready before May 9 so SENTINEL can review it before the sprint begins.

If ELO determines from the recommendation document that the predicted rating shift is small enough (< 5 points for most teams) that the risk of silent miscalibration is low, the monitoring plan can be simplified — but it should still exist. A silent error is worse than a large one, precisely because it won't announce itself.

The monitoring plan is one deliverable in a pair. The implementation SQL (`task-2026-04-27-tier2-calibration-implementation-sql.md`) is the other. They should be filed together or cross-referenced — any developer running the SQL should also know to run the monitoring queries.
