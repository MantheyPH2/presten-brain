---
title: April 29 — Prediction vs Actual Match Assessment
type: comparison-framework
author: ELO
date: 2026-04-25
status: part-a-complete
related: "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]", "[[April 29 Analysis — Results Filing Template]]", "[[April 29 Post-Fix Decision Tree]]"
tags: [calibration, ga-aspire, april-28, girls, validation, evo-draw]
---

# April 29 — Prediction vs Actual Match Assessment

> **[Part A written 2026-04-25 — before April 28 execution. Criteria locked. Do not revise Part A after results are received.]**
>
> Part A is ELO's pre-run framework. Part B is filled after the April 29 analysis session using actuals from `Rankings/April 29 Analysis — Results Filing Template.md`. SENTINEL reads the completed document to authorize "FIX CONFIRMED" or "INVESTIGATE."

---

## Part A: Comparison Framework

_Written before April 29. This section is locked._

### A1. Match Criteria Definitions

| Verdict | Definition |
|---------|-----------|
| **MATCHED** | Actual direction is correct AND actual magnitude is within the predicted cohort-average range (± 5 pts tolerance on each bound) |
| **CLOSE** | Actual direction is correct AND actual magnitude is 5–15 pts outside the predicted range boundary — fix applied correctly, gap is within noise |
| **EXCEEDED** | Actual direction is correct AND actual magnitude exceeds predicted range maximum by > 15 pts — investigate whether overcorrection or ASPIRE game share was underestimated |
| **REFUTED** | Actual direction is OPPOSITE to prediction, OR actual magnitude is < 50% of the predicted minimum (for Girls age groups), OR Boys movement > ±5 pts |

**Boys-specific rule:** The only valid verdict for the Boys row is MATCHED (≤ ±5 pts change) or REFUTED (any change > ±5 pts). There is no CLOSE or EXCEEDED for Boys — the expected Boys change is 0 pts by mechanism.

**Direction-only rows:** For the "ECNL avg direction" and "GA ASPIRE vs ECNL convergence" rows, direction is the operative comparison, not magnitude. MATCHED = correct direction. REFUTED = wrong direction (divergence when convergence expected, or ECNL moves more than GA ASPIRE).

---

### A2. Comparison Table — Expected Column Pre-Filled from Pre-Run Model

_Expected values sourced from `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md`, Sections 3 and 5. "Actual" column filled after April 29 analysis._

| Prediction Row | Expected (Pre-Run) | Actual (fill April 29) | Verdict | Notes |
|---------------|-------------------|----------------------|---------|-------|
| U13G avg direction | **INCREASE** | | | Positive = MATCHED; any decrease = REFUTED |
| U13G avg cohort magnitude | **+5 to +15 pts** | | | Highly exposed teams: +20–40 pts |
| U14G avg direction | **INCREASE** | | | Highest ASPIRE exposure age group |
| U14G avg cohort magnitude | **+8 to +18 pts** | | | Highly exposed teams: +25–40 pts |
| U15G avg direction | **INCREASE** | | | |
| U15G avg cohort magnitude | **+6 to +15 pts** | | | Highly exposed teams: +20–40 pts |
| U16G avg direction | **INCREASE** | | | Tapering exposure |
| U16G avg cohort magnitude | **+4 to +12 pts** | | | Highly exposed teams: +15–35 pts |
| U17G avg direction | **INCREASE** | | | Lowest ASPIRE exposure |
| U17G avg cohort magnitude | **+2 to +8 pts** | | | Highly exposed teams: +10–30 pts |
| Boys avg rating change (all age groups) | **0 pts ± 5 pts** | | | > ±5 pts = immediate REFUTED; ≥ ±15 pts = G4 fail, hold all downstream |
| Girls ECNL avg rating change | **< 5 pts** | | | > 10 pts = unexpected propagation; investigate |
| Girls GA ASPIRE vs ECNL gap direction | **Convergence (gap narrows)** | | | Divergence = REFUTED; GA ASPIRE ↑ and ECNL minimal is the expected pattern |
| Total teams with > 50-pt shift (any direction) | **< 80 teams** | | | > 100 teams = overcorrection; investigate outliers before Rank Bands |

---

### A3. Overall Verdict Rules

| Scenario | Overall Verdict |
|----------|----------------|
| All direction rows MATCHED or CLOSE; Boys = MATCHED; zero REFUTED rows | **FIX CONFIRMED** |
| Any direction row is REFUTED, OR Boys row shows movement > ±5 pts | **INVESTIGATE** — full investigation before SENTINEL authorization |
| All direction rows correct; ≥ 2 magnitude rows outside predicted range (CLOSE only, no REFUTED) | **PARTIAL** — fix applied correctly; magnitude gap explained or accepted |
| Boys movement ≥ ±15 pts (any age group) | **G4 FAIL** — hold all downstream regardless of Girls-side results |

**Default path:** If the analysis session cannot be completed by April 30, treat as INVESTIGATE until results are in hand.

---

## Part B: Results

_ELO fills this section after the April 29 analysis session. Do not fill until `April 29 Analysis — Results Filing Template.md` is complete._

### B1. Actual Results Summary

_Fill "Actual" column in A2 table from the Results Filing Template. Record date and source._

```
Analysis date: ___________
Results source: Rankings/April 29 Analysis — Results Filing Template.md
Results confirmed received: YES / NO
Analyst: Presten (DB session)
```

---

### B2. Row-Level Verdict Assignment

_For each row in A2: assign MATCHED / CLOSE / EXCEEDED / REFUTED using A1 criteria. Write one sentence for any non-MATCHED verdict._

_(Fill after B1.)_

---

### B3. Overall Assessment

```
Overall verdict: FIX CONFIRMED / INVESTIGATE / PARTIAL / G4 FAIL
Reasoning (2–3 sentences): ___________
Boys check: MATCHED / REFUTED — [note if any movement observed]
Largest gap from prediction: [row], expected [range], actual [value], delta [D pts]
Direction verdicts: [N] MATCHED, [N] CLOSE, [N] EXCEEDED, [N] REFUTED
```

---

### B4. SENTINEL Authorization Block

```
[ELO fills B1–B3 before this block]
[SENTINEL reads B3 Overall Assessment and fills below]

SENTINEL authorization:
Fix confirmed for Event Strength Phase 1 (G2 gate): YES / NO
Fix confirmed for Rank Bands activation (G3 gate): YES / NO
Boys investigation required (G4 gate): YES / NO
If any NO: hold reason: ___________
Authorization date: ___________
```

---

## References

- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — source of all Expected column values in A2
- `Rankings/April 29 Analysis — Results Filing Template.md` — actuals source for B1
- `Rankings/April 29 Post-Fix Decision Tree.md` — G1/G2/G3/G4 gates this assessment informs
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — downstream monitoring that begins once FIX CONFIRMED
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Girls calibration cell that this assessment authorizes

*ELO — 2026-04-25*
