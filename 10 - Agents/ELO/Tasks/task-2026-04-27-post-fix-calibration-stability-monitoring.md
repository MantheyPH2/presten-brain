---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-25
priority: medium
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Post-Fix Calibration Stability Monitoring Spec.md"
---

# Task: Post-Fix Calibration Stability Monitoring Spec

## Context

April 28 makes the largest calibration change since the system was built: GA ASPIRE event_tier fix, potentially affecting thousands of Girls games and ratings across all age groups. The April 29 analysis (covered by the Decision Tree and Rating Shift Analysis Spec) confirms the immediate post-fix state. What is not covered: how ELO monitors calibration stability from April 29 through DSS (May 16).

Calibration changes can have delayed effects. New games ingested after April 28 will be computed with the corrected `ga_aspire` cal value — which may produce further rating drift over the following 2–3 weeks. If Girls GA ASPIRE teams are playing active games in late April and early May, their ratings are still moving. ELO needs to know:
1. Whether the fix introduced a new instability (unexpected rating oscillation after the initial shift settles)
2. Whether the corrected ratings converge to the expected distribution before DSS
3. When the system is "stable enough" to trust the DSS demo numbers

This document makes post-April-28 monitoring systematic.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Post-Fix Calibration Stability Monitoring Spec.md`

---

### Section 1: Stability Definition

What does "calibration stable" mean for ELO's purposes?

Define specific criteria. Example structure:
- **Rate of change:** Average Girls GA rating change per daily pipeline run is < [X] points for [N] consecutive runs
- **Distribution convergence:** Girls GA ASPIRE average rating is within [Y] points of the April 29 post-fix baseline for 5 consecutive days
- **Outlier count:** Number of teams with day-over-day rating change > [Z] points is < [threshold] per run

ELO sets the specific values. Reference the Girls GA overcalibration magnitude (40+ pts distortion) to anchor the thresholds — a system with 40-pt instability is not stable; a system with 2-pt day-over-day variation likely is.

State explicitly: does "stable" mean ratings have stopped moving, or that they are moving predictably? ELO's call.

---

### Section 2: Monitoring Query Pack

Three queries Presten runs during psql sessions between April 29 and May 9 to check stability:

**Query 1 — Girls GA ASPIRE Average Rating Trend**
Track average rating for Girls GA ASPIRE teams by age group, run after each pipeline execution. Detects ongoing drift.

```sql
-- ELO writes the actual query
-- Should return: age_group, avg_rating, run_date
-- Presten saves output to Reports/ga-aspire-stability-[date].md
```

**Query 2 — Daily Rating Volatility Check**
Count of teams whose rating changed by > [threshold] since the previous run. High volatility count is an early signal of instability.

```sql
-- ELO writes the actual query
-- Should return: count_large_movers, max_single_move, run_date
-- PASS: count < [N]; FAIL: count > [N] — flag to SENTINEL
```

**Query 3 — Cross-Gender Convergence Check**
After the fix, Girls GA average ratings should move toward (not away from) the Girls ECNL/MLS NEXT distribution. Check the gap is narrowing or stable, not widening.

```sql
-- ELO writes the actual query
-- Should return: girls_ga_avg, girls_ecnl_avg, girls_mlsnext_avg, delta_ga_to_ecnl
-- Expected: delta_ga_to_ecnl decreases or holds constant post-April-28
```

---

### Section 3: Monitoring Schedule

| Date | Action | Decision |
|------|--------|----------|
| April 29 | Run post-fix baseline (full Decision Tree). File in briefing. | GREEN continues to May 1. Any RED holds Event Strength Phase 1. |
| April 30 – May 2 | Run Query 1 + 2 after each pipeline run. | If volatility count > threshold → flag in briefing, ELO investigates |
| May 5 | Run full 3-query pack. File stability status in briefing. | Decision: "stable" → Event Strength Phase 1 cleared. "Not stable" → ELO investigation before Phase 1. |
| May 9 | Final stability check before DSS readiness assessment. | Must be stable to proceed. |
| May 14 | Pre-demo spot check. | Any instability flagged to SENTINEL by 11 AM. |

---

### Section 4: Instability Response Protocol

If any monitoring run shows instability (volatility count above threshold, convergence reversing, or distribution diverging):

**Level 1 — Mild (single-day spike, resolves next run):**
- ELO logs in briefing as "spike observed, resolved." No SENTINEL action.

**Level 2 — Persistent (2+ consecutive runs above threshold):**
- ELO flags in briefing: "Calibration instability detected — [specific signal]."
- ELO identifies whether the instability is sourced from new GA ASPIRE games being ingested (expected, will self-correct as game volume grows) or from a computation error.
- SENTINEL holds Event Strength Phase 1 authorization until ELO clears the flag.

**Level 3 — Structural (distribution diverging, average moving in wrong direction):**
- ELO escalates directly to SENTINEL and Presten.
- April 28 fix may need revision. ELO files an incident note in Queue.
- DSS demo risk elevated — ELO flags Girls GA ASPIRE features as uncertain in the Feature Readiness Tracker.

---

### Section 5: Stability Clearance Protocol

When ELO determines calibration is stable (all monitoring criteria met for [N] consecutive runs):

- ELO files a "Calibration Stability Clearance" note in the next briefing
- Format: "GA ASPIRE calibration stability confirmed as of [date]. Criteria met: [list]. Event Strength Phase 1 cleared from calibration stability dependency."
- SENTINEL records clearance in the next briefing and removes the stability monitoring flag

---

## Definition of Done

- Spec written at `Rankings/Post-Fix Calibration Stability Monitoring Spec.md`
- All 5 sections present
- Stability criteria are explicit numbers — not "low volatility" or "approximately stable"
- Three monitoring queries are written and copy-paste ready
- Monitoring schedule covers April 29 through May 14
- Instability response protocol distinguishes 3 levels with specific ELO actions at each

## Why This Matters

SENTINEL currently has no systematic view of calibration health between sessions. When Presten runs the pipeline on May 1, 2, 3 — ratings are changing with every run. Without a monitoring spec, ELO does not look at those changes unless something obviously breaks. With a monitoring spec, ELO runs 3 queries per session and files a one-line stability update in every briefing. The cost is low; the catch is high — a slow calibration regression between April 28 and May 9 is exactly the kind of failure that would not surface until the DSS readiness check on May 9, when it is too late to fix.

## References

- `Rankings/Post-April-28 Rating Shift Analysis Spec.md` — April 29 analysis that establishes the post-fix baseline this spec monitors against
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — predicted values this spec compares against
- `Rankings/April 29 Post-Fix Decision Tree.md` — G2 gate that feeds into this spec's baseline
- `Rankings/Boys Calibration Status Summary — April 2026.md` — confirms Boys should not be affected
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — calibration flags this spec maintains
