---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: urgent
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Reports/GA ASPIRE Calibration Stability — Baseline 2026-04-29.md"
---

# Task: Calibration Stability Monitoring — April 29 Baseline Filing

## Context

ELO filed `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` (completed April 27) which defines a monitoring schedule starting April 29. That spec requires ELO to file a "post-fix baseline" on April 29 — the first data point for all subsequent stability monitoring.

ELO's briefing (2026-04-26 06:39) listed "File calibration stability baseline (first data point for monitoring spec)" as an April 29 action. No task was created for it. Without a formal task, this is at risk of being skipped if the April 29 session is busy with G1–G4 Decision Tree work.

This baseline is load-bearing: every stability monitoring run from April 30 through May 14 compares against it. If the baseline is not filed on April 29, ELO has no anchor point and stability monitoring degrades to "does it feel stable?" — which is not acceptable for a DSS-critical feature.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Reports/GA ASPIRE Calibration Stability — Baseline 2026-04-29.md`

This is the first execution of the three monitoring queries defined in the Stability Monitoring Spec. Run all three queries immediately after the April 29 session recompute confirms GREEN on G1.

---

### Header

```
Baseline Type: Post-April-28 Fix — Initial Stability Baseline
Date: 2026-04-29
Session context: First read after GA ASPIRE event_tier fix and full recompute
ELO notes: [any anomalies in session context that affect baseline interpretation]
```

---

### Query 1 Results: Girls GA ASPIRE Average Rating by Age Group

Run Query 1 from the Stability Monitoring Spec (`Rankings/Post-Fix Calibration Stability Monitoring Spec.md`).

| Age Group | Avg Rating (Post-Fix) | Game Count | Team Count | Notes |
|-----------|----------------------|------------|------------|-------|
| U13G | | | | |
| U14G | | | | |
| U15G | | | | |
| U16G | | | | |
| U17G | | | | |

**Baseline annotation:** Compare against predicted values in `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` Section 3. Note whether each age group's actual average is within, above, or below the predicted range.

---

### Query 2 Results: Daily Rating Volatility Check

Run Query 2 from the Stability Monitoring Spec.

```
Count of teams with rating change > [threshold] since previous run: [N]
Maximum single-team rating change: [N] points
Run date comparison: April 28 recompute → April 29 first read
Volatility status: PASS (< threshold) / FAIL (> threshold)
```

Note: This first baseline run measures the April 28 → April 29 delta, which is expected to be large (the fix itself). This count is the reference ceiling, not a stability metric. Document it explicitly: "This run captures the fix effect itself — use April 30 run as the first true stability datapoint."

---

### Query 3 Results: Cross-Gender Convergence Check

Run Query 3 from the Stability Monitoring Spec.

| Segment | Avg Rating | Delta from Pre-Fix Baseline |
|---------|------------|----------------------------|
| Girls GA ASPIRE | | |
| Girls ECNL | | |
| Girls MLS NEXT | | |
| Delta: GA ASPIRE → ECNL | | |

**Expected post-fix:** Girls GA ASPIRE average should have moved toward (smaller delta vs.) Girls ECNL/MLS NEXT compared to pre-fix. Confirm this direction is present or flag if not.

---

### Baseline Status Assessment

At the bottom of the file:

```
Baseline Filed: YES
All 3 queries executed: YES / NO (if NO: reason)
Predicted direction confirmed: YES / NO
Anomalies: [none / description]
Next monitoring run: April 30 (Query 1 + Query 2 only)
ELO stability assessment at baseline: [one sentence]
```

---

## Quality Standard

- Run all three queries in the same psql session as the April 29 G1–G4 Decision Tree work. File results before ending the April 29 session.
- The baseline must use post-recompute data. Do not file it if G1 (recompute confirmed complete) has not passed.
- Explicitly label Query 2's first run as the "fix delta run" — stability comparisons start April 30.
- File the baseline before ELO's April 29 briefing so SENTINEL can see it in the same session.

## Sequence Dependency

1. April 28 execution completes (GA ASPIRE UPDATE + recompute) ← prerequisite
2. April 29 G1 gate passes (recompute confirmed) ← prerequisite
3. **This task: run 3 queries, file baseline** ← this task
4. April 29 G2 gate: ELO uses this baseline's Query 3 output for the distribution check

## Why This Matters

The Calibration Stability Monitoring Spec is only as good as its first data point. If the baseline is not filed on April 29, the entire monitoring schedule in the spec shifts by one day — and the May 14 pre-demo spot check is the last one with any response time. A one-day slip in the baseline means ELO loses a monitoring data point at the most critical time. This is a 30-minute task that unlocks 15 days of systematic stability monitoring.

## References

- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — monitoring queries (copy from there)
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — predicted values to compare against
- `Rankings/April 29 Post-Fix Decision Tree.md` — G2 gate uses this baseline's Query 3 output
- `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` — parallel baseline filing context
