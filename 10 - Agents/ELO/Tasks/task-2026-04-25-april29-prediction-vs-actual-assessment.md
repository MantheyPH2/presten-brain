---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: urgent
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Rankings/April 29 — Prediction vs Actual Match Assessment.md"
---

# Task: April 29 — Prediction vs Actual Match Assessment

## Context

ELO filed two April 29 documents this cycle:
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — ELO's predictions before April 28 runs
- `Rankings/April 29 Analysis — Results Filing Template.md` — the form ELO fills with actual results after April 29 analysis

What does not exist: a structured comparison document that explicitly maps predictions to actuals and produces a verdict. The Results Filing Template captures raw data. The Pre-Run Model captures predictions. Without a comparison document, SENTINEL must mentally synthesize the two — reading the prediction ("expected -25 pts for U15G GA teams") against the actual ("U15G GA average moved -18 pts") and forming a judgment. That synthesis is not auditable and produces different conclusions depending on how liberally "matched" is defined.

ELO should define the comparison framework now, before April 29, so the framework itself is not influenced by the results it will be applied to.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/April 29 — Prediction vs Actual Match Assessment.md`

This document has two parts: a framework section (written before April 29) and a results section (filled in after April 29 analysis runs). SENTINEL reads the completed document to authorize "fix confirmed" or "investigate further."

---

### Part A: Comparison Framework (ELO writes before April 29)

#### A1. Match Criteria Definitions

ELO defines what "matched," "close," "exceeded," and "refuted" mean in numeric terms. Example structure:

| Verdict | Definition |
|---------|-----------|
| MATCHED | Actual is within [±X pts or ±Y%] of prediction |
| CLOSE | Actual is within [±X to ±Z pts] of prediction — acceptable but investigate cause of gap |
| EXCEEDED | Actual moved further than predicted — note whether this is concerning or expected |
| REFUTED | Actual moved in the opposite direction from prediction, or magnitude was less than [threshold] |

ELO sets the specific thresholds. Reference: the Pre-Run Model uses "40+ points of distortion" as the overcalibration magnitude. A 5-point miss on a 25-point predicted shift is MATCHED. A 25-point miss on a 25-point predicted shift is REFUTED.

**For Boys:** The only valid verdict is MATCHED (≈ 0 pts) or REFUTED (any movement > 5 pts). Boys have only one prediction and it is binary.

#### A2. Comparison Table Template

| Prediction | Expected | Actual (fill April 29) | Verdict | Notes |
|-----------|----------|----------------------|---------|-------|
| U13G GA avg rating direction | [up/down] | | | |
| U13G GA avg rating magnitude | [N pts] | | | |
| U14G GA avg rating direction | | | | |
| U14G GA avg rating magnitude | | | | |
| U15G GA avg rating direction | | | | |
| U15G GA avg rating magnitude | | | | |
| U16G GA avg rating direction | | | | |
| U16G GA avg rating magnitude | | | | |
| U17G GA avg rating direction | | | | |
| U17G GA avg rating magnitude | | | | |
| Boys avg rating change | 0 pts | | | |
| Girls ECNL avg rating change | < 5 pts | | | |
| Girls GA ASPIRE avg rating change | [±N pts] | | | |
| Distribution convergence (GA → ECNL gap) | narrowing | | | |

Pre-fill the "Expected" column from the Pre-Run Model before April 29. Do not pre-fill the "Actual" column.

#### A3. Overall Verdict Rules

ELO states the rule for computing the overall assessment from the row-level verdicts:

- **FIX CONFIRMED:** All direction predictions MATCHED or CLOSE. Boys = MATCHED. No row is REFUTED.
- **INVESTIGATE:** Any direction prediction is REFUTED, OR Boys row shows movement > [threshold]. Full investigation before SENTINEL authorization.
- **PARTIAL:** All directions correct but ≥ 2 rows show magnitude outside expected range. Fix applied correctly; magnitude gap explained or investigated.

---

### Part B: Results (ELO fills after April 29 analysis)

#### B1. Actual Results Summary

Fill in the "Actual" column of the A2 table from the April 29 Results Filing Template. Do not fill B1 until the April 29 analysis session is complete and the Results Filing Template is filed.

#### B2. Row-Level Verdict Assignment

For each row in the comparison table: assign a verdict using the A1 criteria. Write one sentence explaining any CLOSE, EXCEEDED, or REFUTED verdict.

#### B3. Overall Assessment

```
Overall verdict: FIX CONFIRMED / INVESTIGATE / PARTIAL
Reasoning (2–3 sentences): [ELO's plain-language explanation]
Boys check: MATCHED / REFUTED — [note if any movement observed]
Largest gap from prediction: [row], expected [N], actual [M], delta [D pts]
```

#### B4. SENTINEL Authorization Block

```
[ELO fills B1–B3]
[SENTINEL reads and fills this block]

SENTINEL authorization:
Fix confirmed for Event Strength Phase 1: YES / NO
If NO, hold reason: ___________
```

---

## Definition of Done

- Framework (Part A) filed before April 28 execution. The "Expected" column of the comparison table is pre-filled from the Pre-Run Model.
- After April 29 analysis: Part B filled in, all rows assessed, Overall Assessment written.
- SENTINEL Authorization Block present and blank (SENTINEL fills it).
- ELO summary in next briefing: "Prediction vs Actual Assessment filed. Overall verdict: [X]. [N]/[total] predictions MATCHED. Boys: MATCHED/REFUTED. SENTINEL authorization pending."

## Why This Matters

SENTINEL's current authorization path for Event Strength Phase 1 is: "read ELO's April 29 analysis briefing and decide." That is a narrative review. This comparison document converts it into a structured check: if all direction verdicts are MATCHED and Boys = MATCHED, ELO has confirmed the fix worked as designed. SENTINEL's authorization becomes a checkbox review, not a subjective interpretation. The framework must be written before April 29 — a framework written after seeing the results is not a test, it is a post-hoc rationalization.

## References

- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — predictions that populate Part A's Expected column
- `Rankings/April 29 Analysis — Results Filing Template.md` — actuals that populate Part B
- `Rankings/April 29 Post-Fix Decision Tree.md` — G2 gate that this assessment informs
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — downstream monitoring that begins once this assessment clears
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Girls calibration cell that this assessment authorizes
