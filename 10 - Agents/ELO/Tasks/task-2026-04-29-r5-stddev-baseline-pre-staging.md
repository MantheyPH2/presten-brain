---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-29
priority: high
due: 2026-04-30 EOD
status: pending
topic: r5-stddev-baseline-pre-staging
sentinel_note: Required for May 1 launch authorization. Must be filed before May 1.
---

# Task: R5 Stddev Baseline — Pre-May-1 Pre-Staging

## Context

SENTINEL's May 1 launch authorization (target: April 30 EOD) includes ELO's R5 stddev baseline as a required condition. This appears in the SENTINEL decision table from the 06:41 briefing:

> "May 1 launch authorization: FORGE confidence brief ✅ + TYSA org-ID + **ELO R5 stddev baseline**"

ELO's May 1 Pipeline Launch Rankings Monitoring Plan (`task-2026-04-28-may1-pipeline-launch-rankings-monitoring-plan.md`) references this baseline but has not filed it as a standalone document. SENTINEL needs a pre-staged document that defines the stddev baseline and the thresholds that would trigger ELO concern if rankings destabilize post-launch.

---

## What to Build

Create `Rankings/May 1 Pipeline Launch — ELO Ratings Baseline.md`

This document defines what "stable ratings" looks like immediately before May 1, so that post-launch comparisons have a clear reference point. It also defines ELO's monitoring commitments for the first week.

---

## Required Content

### Section 1 — Purpose

1 paragraph: this document establishes the pre-launch ratings baseline so ELO can detect pipeline-induced rating instability after May 1. It is a prerequisite for SENTINEL's May 1 launch authorization.

### Section 2 — Baseline Snapshot Definition

Define what the baseline covers:

| Metric | Scope | Pre-staging note |
|--------|-------|-----------------|
| Girls ratings stddev by age group | All active girls age groups | [FILL: can ELO compute from current vault data, or does this require a psql query?] |
| Boys ratings stddev by age group | All active boys age groups | [FILL: same as above] |
| Expected game ingest per source (Stage 1) | 5 active sources | Pre-staged in Pipeline Week 1 Monitoring Log |
| Expected rating shift from first new games | [FILL: what magnitude is normal for first-week new game ingestion?] | |

For each metric ELO CAN compute from vault data: fill in the actual value.
For each metric requiring a psql query: state the query ELO will run in the April 29 session (or flag if it requires post-launch data).

### Section 3 — Alert Thresholds

Define what would constitute a concerning post-launch rating shift:

| Metric | GREEN (normal) | YELLOW (investigate) | RED (escalate to SENTINEL) |
|--------|---------------|---------------------|---------------------------|
| Girls stddev change vs. pre-launch baseline | < X% | X–Y% | > Y% |
| Boys stddev change vs. pre-launch baseline | < X% | X–Y% | > Y% |
| Any single team's rating shift (1 week) | < Z pts | Z–W pts | > W pts |

ELO fills the threshold values based on historical calibration data and expected game volume. These thresholds do not need to be perfect — they need to be defensible. If ELO cannot set thresholds without session data, state the formula and fill after May 1 morning.

### Section 4 — Week 1 Monitoring Commitment

| Day | ELO Action | Signal to SENTINEL |
|-----|-----------|-------------------|
| May 1 (pipeline runs) | Check for rating updates in rankings view | "May 1 pipeline: ratings updated — [X] teams affected, stddev delta [Y]" |
| May 2 | Review any rating shifts > threshold | Flag if RED; brief if YELLOW |
| May 4 (first weekend run) | Aggregate week 1 assessment | ELO week 1 ratings stability summary in briefing |
| May 8 | Update May 9 DSS risk register with stability data | R7 (pipeline instability) risk level update |

### Section 5 — Pre-May-1 Certification

```
ELO certifies that:
[ ] Baseline metrics are defined for all active age groups
[ ] Alert thresholds are set and defensible
[ ] Week 1 monitoring commitment is confirmed
[ ] No known calibration issues would cause anomalous post-launch shifts
    (Exception: GA ASPIRE fix — see note)

GA ASPIRE note: If April 29 session does not run, post-launch ratings may
reflect GA ASPIRE mis-classification. ELO will monitor for this specifically
and flag any GA ASPIRE-adjacent anomalies separately.

ELO: _________________________ Date filed: 2026-04-30
```

---

## Deliverable

`Rankings/May 1 Pipeline Launch — ELO Ratings Baseline.md`

Filed before May 1. All fillable cells filled. Cells requiring session data clearly marked with formula or query. Section 5 certification complete.

SENTINEL uses this document as one of the three required conditions for May 1 launch authorization.
