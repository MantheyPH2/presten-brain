---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-25
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Pre-May-1 Girls Calibration Baseline — Snapshot Package.md"
---

# Task: Pre-May-1 Girls Calibration Baseline — Snapshot Package

## Context

The Post-Fix Calibration Stability Monitoring Spec (filed this cycle) defines a monitoring schedule from April 29 through May 14. Its Section 3 monitoring schedule begins with "April 29: Run post-fix baseline (full Decision Tree)." This April 29 baseline becomes the comparison point for all subsequent stability checks.

However: the daily pipeline starts running on May 1. Each pipeline run changes ratings. The calibration stability monitoring spec runs Queries 1–3 "after each pipeline run" from April 30 – May 2 — but against what baseline? The April 29 post-fix state is Day 0 for the DB session; May 1 run 1 is Day 0 for the production pipeline period.

These are different snapshots. Without a pre-May-1 baseline specifically scoped to the post-fix GA ASPIRE state, ELO's May 1 stability comparison collapses April 29's DB-session changes with the first production pipeline run's changes. If ratings shift on May 1, ELO cannot distinguish "expected May 1 pipeline movement" from "unexpected calibration drift" without a clean May 1 baseline.

This task: ELO writes the SQL queries for the pre-May-1 snapshot. These are run immediately after April 29 analysis is confirmed GREEN — before May 1 pipeline launch. They capture the stable post-fix state as the true production baseline.

## What to Produce

### File: `Rankings/Pre-May-1 Girls Calibration Baseline — Snapshot Package.md`

---

### When to Run

**Timing:** Immediately after April 29 analysis is confirmed complete and the Decision Tree verdict is GREEN (or INVESTIGATE but not HOLD). Must be run **before** the May 1 pipeline executes. Do not run if April 29 verdict is RED (ratings are in an unstable state — baseline is meaningless until remediated).

**Why this timing:** The pre-May-1 baseline must capture the post-fix, pre-production state. Once May 1 pipeline runs, this window has closed.

---

### Snapshot Query 1 — Girls GA ASPIRE Average by Age Group

```sql
-- ELO writes the actual query
-- Returns: age_group, avg_rating, median_rating, stddev_rating, team_count
-- WHERE: girls teams, GA ASPIRE event tier exposure
-- This is the primary calibration health signal
```

**Save result to:** `Rankings/Pre-May-1 Baseline Results — [date].md`

---

### Snapshot Query 2 — Girls Cross-Tier Distribution

```sql
-- ELO writes the actual query
-- Returns: tier_group (ECNL / MLS NEXT / GA / NPL / DPL / other), age_group, avg_rating, team_count
-- Gender: Girls only
-- Purpose: captures the full Girls rating distribution across tiers
```

**Save result to:** Same results file as Query 1.

---

### Snapshot Query 3 — Girls Rating Distribution Percentiles

```sql
-- ELO writes the actual query
-- Returns: age_group, p10, p25, p50, p75, p90 rating percentiles
-- Gender: Girls only
-- Purpose: anchors the distribution shape for drift detection
```

---

### How to Use This Baseline

After each pipeline run (May 1, 2, 3…), Stability Monitoring Query 1 (from the Monitoring Spec) tracks Girls GA ASPIRE average rating. It compares against **this pre-May-1 baseline**, not the April 29 Decision Tree baseline.

The comparison logic:
- If Girls GA ASPIRE May 1 avg ≈ pre-May-1 baseline avg (within the stable-range threshold from the Monitoring Spec): STABLE
- If Girls GA ASPIRE May 1 avg diverges from pre-May-1 baseline by more than the Monitoring Spec's Level 2 threshold: flag in briefing

---

### Baseline Reference Table

ELO fills this table when the queries run (April 29 session or immediately after):

| Age Group | GA ASPIRE Avg Rating | ECNL Avg Rating | MLS NEXT Avg Rating | Date Captured |
|-----------|---------------------|-----------------|---------------------|---------------|
| U13G | | | | |
| U14G | | | | |
| U15G | | | | |
| U16G | | | | |
| U17G | | | | |

This table becomes the lookup reference for all stability monitoring through May 14.

---

### Integration with Monitoring Spec

Add a note to `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` (or reference this document from it): "Pre-May-1 baseline captured at `Rankings/Pre-May-1 Girls Calibration Baseline — Snapshot Package.md`. Stability Monitoring Queries 1–3 compare against this baseline, not against the April 29 Decision Tree baseline."

---

## Definition of Done

- 3 snapshot queries written and copy-paste ready
- Timing instructions are unambiguous (when GREEN on April 29, before May 1)
- Baseline Reference Table is included with blank rows ready to fill
- Integration note added to or filed separately for reference by the Monitoring Spec
- Summary in briefing: "Pre-May-1 baseline snapshot package ready. Timing: run April 29 post-GREEN, before May 1 launch. Queries: GA ASPIRE avg by age group, cross-tier distribution, percentile distribution."

## Why This Matters

The calibration stability monitoring spec is only as good as its baseline. Writing the spec without the baseline is like defining acceptable heart rate without measuring resting heart rate. This snapshot is the resting heart rate — measured at the exact right moment (post-fix, pre-production), used as the reference for all monitoring through DSS. At 30 minutes to write, it is the cheapest way to make the calibration monitoring spec actually function as designed.

## References

- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — spec this baseline supports
- `Rankings/April 29 Post-Fix Decision Tree.md` — GREEN verdict required before running this baseline
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — predicted values to compare against when filling the Baseline Reference Table
