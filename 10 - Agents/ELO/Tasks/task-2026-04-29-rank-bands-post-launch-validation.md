---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-29
priority: medium
due: 2026-04-30 EOD
status: pending
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Rank Bands — Post-Launch Validation Spec.md"
topic: rank-bands-post-launch-validation
tags: [elo, task, rank-bands, validation, post-launch, may1]
---

# Task: Rank Bands — Post-Launch Validation Spec

## Context

Rank bands are implemented and ready. The pipeline launches May 1. After the first week of new game data ingests (May 1–8), ELO must verify that rank bands are behaving correctly: bands are non-empty, distribution is reasonable, no band is absorbing a disproportionate share of teams.

There is currently no spec for what "correct" rank band behavior looks like post-launch. Without pre-defined thresholds and queries, ELO will have to improvise on May 8 while also completing the calibration dashboard and responding to pipeline results. Pre-staging this spec eliminates that crunch.

## What to Build

Produce: `02 - Tiger Tournaments/Projects/Rankings/Rank Bands — Post-Launch Validation Spec.md`

---

## Required Sections

### Section 1 — What Rank Bands Are and What Could Go Wrong

2–3 sentences for context. What are rank bands? What are the specific failure modes ELO is checking for after a pipeline launch?

Failure modes to cover:
- Band collapse: all teams fall into 1–2 bands (miscalibrated thresholds)
- Band overfill: one band is disproportionately large (>50% of teams)
- Empty bands: a band has zero teams (threshold too aggressive)
- Rating drift: pipeline's new games shift Elo enough to push large numbers of teams across band thresholds

### Section 2 — Validation Queries

Write 3–4 SQL queries Presten can run May 2–8. Each query:
- Has a clear label (RB-V-1, RB-V-2, etc.)
- States what it measures
- States the expected/acceptable output
- States the FAIL condition

**RB-V-1: Band Distribution Count**
- Measure: Number of teams per rank band, by age group and gender
- Expected: Distribution roughly follows a bell curve or expected shape per band design
- Fail: Any band with 0 teams OR any band with >50% of teams

**RB-V-2: Band Threshold Coverage Check**
- Measure: Min and max Elo rating within each band
- Expected: No gaps between bands; each band's max < next band's min (or = if bands are contiguous)
- Fail: Gaps between bands; overlaps between bands

**RB-V-3: Post-Pipeline Rating Shift per Band**
- Measure: Average Elo change (pre vs. post May 1 pipeline run) by band
- Expected: Small average shifts (<10 pts avg across all teams)
- Fail: Any age/gender group with average shift >25 pts (indicates calibration instability)

**RB-V-4: Cross-Gender Band Check**
- Measure: Confirm boys and girls rank band populations are computed independently (no cross-gender contamination)
- Expected: Zero teams appear in both boys and girls band tables for the same team_id
- Fail: Any team_id appearing in both

### Section 3 — Pass/Fail Decision Table

| Query | Pass | Fail | Action on Fail |
|-------|------|------|----------------|
| RB-V-1 | All bands non-empty, no band >50% | Any band empty or >50% | Adjust band thresholds; escalate to Presten |
| RB-V-2 | No gaps, no overlaps | Gap or overlap found | Review band definition SQL; file bug with FORGE |
| RB-V-3 | All age groups avg shift <25 pts | Any group >25 pts | Flag in calibration dashboard; flag to SENTINEL |
| RB-V-4 | Zero cross-gender team_ids | Any cross-gender ID found | Pipeline FORGE data quality bug; pause band display |

### Section 4 — Execution Timeline

| Step | When | Who | Action |
|------|------|-----|--------|
| Pre-stage queries | Now (April 29–30) | ELO | File this spec |
| Pipeline runs | May 1 | Presten | First pipeline execution |
| Run RB-V-1 through RB-V-4 | May 2–8 | Presten | Share results with ELO |
| ELO verdict | May 8 | ELO | File pass/fail in calibration dashboard |
| SENTINEL review | May 8–9 | SENTINEL | Incorporate into May 9 DSS gate assessment |

### Section 5 — Expected Band Shape (Pre-Filled)

ELO states what the expected distribution shape is, based on the rank bands design and known Elo rating distribution. This becomes the comparison baseline:
- [ELO fills: what percentage of teams should be in each band tier based on the band design doc and known rating distribution]
- Reference: `task-2026-04-23-rank-bands-design.md`, `task-2026-04-25-rank-bands-demo-examples.md`

---

## Definition of Done

- All 4 queries are written in copy-paste-ready SQL
- Pass/fail thresholds are stated for each query
- Execution timeline is clear
- Section 5 expected shape is pre-stated (no blanks)
- Document is self-contained — Presten can execute without ELO present

## References

- `task-2026-04-23-rank-bands-design.md` — band design and thresholds
- `task-2026-04-24-rank-bands-threshold-validation.md` — prior validation spec
- `task-2026-04-25-rank-bands-post-april28-validation-spec.md` — post-April-28 spec
- `task-2026-04-25-rank-bands-demo-examples.md` — expected band distribution examples
