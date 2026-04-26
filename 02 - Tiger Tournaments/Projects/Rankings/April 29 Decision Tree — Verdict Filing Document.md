---
title: April 29 Decision Tree — Verdict Filing Document
type: verdict-filing
author: ELO
date: 2026-04-26
status: awaiting-execution
related: "[[April 29 Post-Fix Decision Tree]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]", "[[Post-April-28 Rating Shift Analysis Spec]]"
tags: [calibration, ga-aspire, april-29, verdict, authorization, evo-draw]
---

# April 29 Decision Tree — Verdict Filing Document

> **Purpose:** Structured authorization record for Event Strength Phase 1. ELO fills Part A after running the Decision Tree on April 29. SENTINEL fills Part B to record authorization. This document replaces the briefing-narrative authorization with a structured, auditable record.
>
> **How to use:** ELO completes this document during the April 29 analysis session. Part B is reserved for SENTINEL exclusively — ELO does not fill it. File this document in the same session as the Decision Tree run. SENTINEL will not authorize Event Strength Phase 1 without it.
>
> **If April 28 did not run by April 29:** File a one-line update in the next briefing: "Decision Tree not run — April 28 not confirmed executed. All gates blocked." Do not file a partial verdict.

---

## Header

```
Date of April 28 session: ___________
Date ELO ran Decision Tree: ___________
ELO signing analyst: ELO
Pre-run baseline source: Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md
```

---

## Part A — ELO Analysis (ELO fills)

### Gate G1: Execution Log Complete

**Source:** `Infrastructure/April 28 Execution Log.md`

| Field | ELO Confirms |
|-------|-------------|
| April 28 Execution Log exists and has all 4 steps marked complete | YES / NO |
| Recompute confirmed complete (pipeline confirmation on file) | YES / NO |
| **G1 Verdict** | **PASS / FAIL** |

> **If G1 FAIL:** Stop. Do not run G2–G4. File this document with "HELD — G1 fail: April 28 execution not confirmed" and flag to SENTINEL. Proposed next action: ___________

---

### Gate G2: Rating Distribution Check

**Source:** `Rankings/Post-April-28 Rating Shift Analysis Spec.md` — Query 1 (Girls GA ASPIRE average by age group, pre vs. post)

**Pre-run predictions from:** `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md`

Run the rating shift analysis spec queries and fill the table below. Pre-fix averages come from the baseline snapshot.

| Age Group | Pre-Fix Avg Rating | Post-Fix Avg Rating | Delta (pts) | Direction Matches Prediction? | Within Predicted Magnitude Range? |
|-----------|-------------------|--------------------|-----------|-----------------------------|----------------------------------|
| U13G | | | | YES / NO | YES / NO |
| U14G | | | | YES / NO | YES / NO |
| U15G | | | | YES / NO | YES / NO |
| U16G | | | | YES / NO | YES / NO |
| U17G | | | | YES / NO | YES / NO |

**G2 PASS criteria:** ≥ 4 of 5 age groups show delta in the predicted direction (positive for Girls GA ASPIRE cohort), AND at least 2 age groups are within the predicted magnitude range (see Pre-Run Model Section 3 for age-group ranges).

| G2 Verdict | PASS / FAIL / INCONCLUSIVE |
|-----------|--------------------------|
| Age groups passing direction check: | ___ / 5 |
| Age groups passing magnitude check: | ___ / 5 |
| Reason if not PASS (one line): | |

---

### Gate G3: Prediction vs. Actual Match

**Source:** `Rankings/April 29 — Prediction vs Actual Match Assessment.md` Part B (fill Part B in that document first, then summarize here)

| Assessment | ELO Verdict |
|-----------|------------|
| Pre-Run Model direction prediction: | MATCHED / OPPOSED / INCONCLUSIVE |
| Pre-Run Model magnitude prediction: | WITHIN RANGE / ABOVE RANGE / BELOW RANGE |
| G3 overall classification | MATCHED / CLOSE / DIVERGED |

**G3 PASS:** MATCHED or CLOSE.
**G3 FAIL:** DIVERGED (actual ratings moved more than 2× the predicted magnitude, or in the opposite direction for the majority of age groups).

| G3 Verdict | PASS / FAIL |
|-----------|-----------|
| Summary (one sentence if not PASS): | |

---

### Gate G4: Boys Rating Unchanged

**Source:** Any Boys age group average from `rankings` table, compared against pre-April-28 baseline snapshot.

**Rationale:** April 28 targets Girls GA ASPIRE events. Boys ratings should not change. A shift > 5 points in any Boys age group average indicates the fix applied to an incorrect population scope.

Run the following check for each Boys age group:

```sql
-- Boys average rating check — compare to pre-April-28 baseline
SELECT
  age_group,
  ROUND(AVG(rating)::numeric, 1) AS post_fix_avg
FROM rankings
WHERE gender = 'M'
  AND age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
GROUP BY age_group
ORDER BY age_group;
```

Cross-reference each result against the Pre-April-28 Baseline Snapshot.

| Metric | Result |
|--------|--------|
| Largest Boys rating change (any age group, any direction) | ___ pts |
| Age group with largest change: | |
| Threshold: PASS ≤ 5 pts; FAIL > 5 pts | |
| **G4 Verdict** | **PASS / FAIL** |

> **If G4 FAIL (Boys moved > 5 pts avg in any age group):** This is an Option A violation. The GA ASPIRE fix affected Boys data unexpectedly. Stop both Event Strength and Rank Bands. ELO investigates before any further authorization. File G4 incident note in Queue.

---

### ELO Overall Verdict

```
G1: ___    G2: ___    G3: ___    G4: ___

Decision logic:
  All 4 PASS                       → AUTHORIZED
  Any G1, G2, or G3 FAIL           → HELD
  G4 FAIL (Boys > 5 pts)           → HELD — investigation required
  G3 = CLOSE, all others PASS      → CONDITIONAL AUTHORIZATION (SENTINEL decides)
  G4 borderline (5–15 pts Boys)    → CONDITIONAL — ELO identifies cause before EOD

ELO Verdict: AUTHORIZED / HELD / CONDITIONAL

ELO summary (one sentence, state what happened and why the verdict follows):
___________

Event Strength Phase 1 status per this verdict: AUTHORIZED / HELD / CONDITIONAL
Girls calibration status per this verdict: CONFIRMED IMPROVED / INCONCLUSIVE / FLAG FOR REVIEW
```

**ELO signature:** ELO | Date: ___________

---

## Part B — SENTINEL Disposition (SENTINEL fills — DO NOT PRE-FILL)

```
Decision Tree reviewed: [ ]
Part A read in full: [ ]
ELO verdict accepted: [ ]

Event Strength Phase 1: AUTHORIZED / HELD / HELD-CONDITIONAL
Rank Bands: AUTHORIZED / HELD

Girls calibration status post-April-28: CONFIRMED / FLAGGED
Boys calibration: CONFIRMED UNAFFECTED / FLAG OPEN — awaiting ELO investigation

Any flags to Presten: ___________

SENTINEL sign-off: ___________  Date: ___________
```

---

## Authorization Statement (SENTINEL copies to briefing when all gates PASS)

> **Event Strength Phase 1 — AUTHORIZED**
>
> Date: 2026-04-29
> Authorization basis: All four April 29 verdict gates confirmed by ELO.
>
> - G1 (Execution log complete): PASS
> - G2 (Rating distribution check): PASS — Girls GA ASPIRE avg shift: [+X pts across N/5 age groups]
> - G3 (Prediction vs. actual match): PASS — classification: [MATCHED / CLOSE]
> - G4 (Boys rating unchanged): PASS — max Boys change: [N pts], within threshold
>
> Event Strength Phase 1 may proceed per `Rankings/Event Strength Phase 1 — Full Execution Package.md`.
> Rank Bands implementation may proceed per the Rank Bands spec.
>
> Authorized by: SENTINEL | April 29, 2026

---

## References

- `Rankings/April 29 Post-Fix Decision Tree.md` — gate logic context
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — G3 comparison source (Section 3)
- `Rankings/Post-April-28 Rating Shift Analysis Spec.md` — G2 query methodology
- `Rankings/April 29 — Prediction vs Actual Match Assessment.md` — G3 source (fill Part B there first)
- `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md` — pre-fix baseline for G2 and G4 delta
- `Infrastructure/April 28 Execution Log.md` — G1 source

*ELO — 2026-04-26*
