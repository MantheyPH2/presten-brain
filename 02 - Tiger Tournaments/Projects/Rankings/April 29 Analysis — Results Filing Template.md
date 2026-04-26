---
title: April 29 Analysis — Results Filing Template
type: results-template
author: ELO
date: 2026-04-25
status: ready — fill in during April 29 session
related: "[[April 29 Analysis Session — ELO Execution Package]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]", "[[Post-Fix Calibration Stability Monitoring Spec]]", "[[April 29 Post-Fix Decision Tree]]"
tags: [april-29, ga-aspire, results, calibration, dss, evo-draw]
---

# April 29 Analysis — Results Filing Template

> **Instructions:** Open this file alongside the `April 29 Analysis Session — ELO Execution Package.md` during the April 29 psql session. Fill in each blank as you go. Do not reformat — the structure is designed for SENTINEL to read directly. All data cells should contain numbers or YES/NO/PASS/FAIL — no prose summaries in cells.

---

## Session Header

```
Session Date: ___________
Session Start Time: ___________
Pre-condition confirmed: April 28 Execution Log all steps ✅ (YES / NO): ___________
Recompute confirmed complete (YES / NO): ___________
April 28 Execution Log file path: ___________
```

---

## Gate G1: April 28 Execution Confirmation

Reference: `Infrastructure/April 28 Execution Log.md` — SENTINEL Disposition section.

```
G1 Status: PASS / FAIL
SENTINEL Disposition confirmed: AUTHORIZED / HELD
If HELD — reason: ___________
G1 note: ___________
```

**G1 FAIL action:** Do not proceed. File this template with G1 = FAIL and contact SENTINEL. All downstream gates are void.

---

## Phase 2: Rating Shift vs. Pre-Run Model

Reference: `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — Section 3 prediction table.

**Predicted values (from Pre-Run Model — do not modify):**

| Age Group | Predicted Direction | Predicted Range (Highly Exposed) | Predicted Range (Cohort Avg) |
|-----------|--------------------|---------------------------------|-----------------------------|
| U13G | INCREASE | +20 to +40 pts | +5 to +15 pts |
| U14G | INCREASE | +25 to +40 pts | +8 to +18 pts |
| U15G | INCREASE | +20 to +40 pts | +6 to +15 pts |
| U16G | INCREASE | +15 to +35 pts | +4 to +12 pts |
| U17G | INCREASE | +10 to +30 pts | +2 to +8 pts |
| Boys (all) | NO CHANGE | 0 pts | 0 ± 5 pts |

**Actual results (fill in April 29):**

| Age Group | Pre-Fix Avg Rating | Post-Fix Avg Rating | Actual Shift (pts) | Predicted Direction Match? | Within Predicted Range? |
|-----------|--------------------|---------------------|--------------------|-----------------------------|------------------------|
| U13G | | | | YES / NO | YES / NO |
| U14G | | | | YES / NO | YES / NO |
| U15G | | | | YES / NO | YES / NO |
| U16G | | | | YES / NO | YES / NO |
| U17G | | | | YES / NO | YES / NO |
| U13B | | | | (expected: 0 ± 5) | YES / NO |
| U14B | | | | (expected: 0 ± 5) | YES / NO |
| U15B | | | | (expected: 0 ± 5) | YES / NO |
| U16B | | | | (expected: 0 ± 5) | YES / NO |
| U17B | | | | (expected: 0 ± 5) | YES / NO |

```
Boys max shift observed (any age group): _____ pts
Boys shift ≤ 5 pts confirmed: YES / NO
If Boys shifted > 5 pts — value and age group: ___________
Total teams with > 50-pt shift (any direction): _____
Expected threshold: < 80 teams. Threshold exceeded: YES / NO
Overall Phase 2 assessment: CONSISTENT WITH MODEL / INCONSISTENT — [note if inconsistent]: ___________
```

---

## Gate G2: Distribution Check

Reference: Pre-Run Model Section 4 — Distribution Shift Predictions.

```
Girls GA ASPIRE avg rating moved in expected direction (UP): YES / NO
Girls GA ASPIRE avg change (U13G): _____ pts
Girls GA ASPIRE avg change (U14G): _____ pts
Girls non-GA avg rating change < 5 pts: YES / NO
Girls non-GA avg change observed: _____ pts
Girls GA vs. Girls ECNL gap direction: NARROWED / HELD / WIDENED
G2 Status: PASS / FAIL
G2 note: ___________
```

**G2 FAIL conditions (from Pre-Run Model Section 5):**
- Girls GA ASPIRE avg change is negative (any amount) → hard FAIL
- Girls GA ASPIRE avg change < +10 pts (U13G or U14G) → partial FAIL
- Girls ECNL avg change > 10 pts → scope exceeded, investigate
- GA ASPIRE vs. ECNL gap WIDENED → fix produced wrong outcome

---

## Phase 4: Rank Bands Distribution

Reference: `Rankings/Rank Bands — Post-April-28 Validation Spec.md`

| Tier | Girls Before (count) | Girls After (count) | Net Change | Expected Direction |
|------|---------------------|---------------------|------------|--------------------|
| Elite / Gold | | | | INCREASE (GA ASPIRE teams correct upward) |
| Advanced / Silver | | | | MINOR CHANGE |
| Competitive / Bronze | | | | DECREASE (some teams move up) |
| Developmental | | | | MINOR CHANGE |
| Boys Elite | | | | ~0 (no change expected) |
| Boys Advanced | | | | ~0 |
| Boys Competitive | | | | ~0 |
| Boys Developmental | | | | ~0 |

```
Distribution shift abnormal — any tier moves > 15% of teams: YES / NO
If YES, affected tier and count: ___________
G3 (rank bands) Status: PASS / FAIL
G3 note: ___________
```

---

## Phase 5: Calibration Stability Baseline

Reference: `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — stability criteria and Query 1/3 format.

**This baseline is the reference point for every monitoring run through May 14. Save it.**

| Age Group | Girls GA ASPIRE Avg Rating (post-fix) | Girls ECNL Avg Rating | Delta ECNL − GA ASPIRE | Girls MLS NEXT Avg Rating |
|-----------|--------------------------------------|-----------------------|------------------------|--------------------------|
| U13G | | | | |
| U14G | | | | |
| U15G | | | | |
| U16G | | | | |
| U17G | | | | |

```
Baseline saved: YES / NO
Baseline save location (file path): ___________
Expected: delta_ecnl_minus_ga_aspire is POSITIVE for all age groups (ECNL is Tier 1A above GA ASPIRE)
All deltas positive: YES / NO
If any delta is negative, age group and value: ___________
```

---

## Gate G4: Boys Option A Window

Reference: `Rankings/Boys Option A Spot Check — Presten Execution Package.md`

```
Boys Option A spot-check queries reviewed: YES / NO
Boys Option A pre-run gate cleared (April 28 GA ASPIRE fix confirmed): YES / NO
Target run window: April 29 – May 5
Any Boys anomaly in Phase 2 that blocks Boys Option A window: YES / NO
If YES, describe blocker: ___________
G4 (Boys window open): CONFIRMED / BLOCKED
```

---

## Phase 6: DSS Feature Readiness Tracker Update

Reference: `Product & Planning/DSS Feature Readiness Tracker — May 2026.md`

```
DSS Feature Readiness Tracker updated: YES / NO
Girls calibration cell: ✅ / ⚠️ / ❌ — note: ___________
Boys calibration cell: ✅ / ⚠️ / ❌ — note: ___________
Event Strength Phase 1 cell: ✅ / ⚠️ / ❌ — note: ___________
Rank Bands cell (post-G3): ✅ / ⚠️ / ❌ — note: ___________
```

---

## Session Summary

*Fill at session end.*

```
Session End Time: ___________
All 6 phases completed: YES / NO
Phases not completed (if any): ___________

Gate Results:
  G1 (April 28 execution confirmed): PASS / FAIL
  G2 (distribution check): PASS / FAIL
  G3 (rank bands): PASS / FAIL
  G4 (Boys window open): CONFIRMED / BLOCKED

Overall verdict: GREEN / YELLOW / RED
Verdict reasoning: ___________

SENTINEL authorization request:
  [ ] Event Strength Phase 1: request AUTHORIZED / do not yet request
  [ ] Calibration stability monitoring: ACTIVE (begin daily queries)
  [ ] Boys Option A run window: OPEN / BLOCKED

ELO notes for SENTINEL: ___________
```

---

## SENTINEL Authorization

*SENTINEL fills this section after reviewing the completed template. ELO leaves blank.*

```
Results reviewed: [ ]
G1–G4 confirmed: [ ]
Event Strength Phase 1: AUTHORIZED / HELD — reason: ___________
Calibration stability monitoring: ACTIVE / HOLD
Boys Option A window: OPEN / BLOCKED — reason: ___________
Next ELO action: ___________
SENTINEL sign-off: SENTINEL — [date]
```

---

## Quick-Reference: Pass/Fail Thresholds

| Signal | PASS Threshold | FAIL Condition |
|--------|---------------|----------------|
| Girls GA ASPIRE avg shift (U13G/U14G) | > +10 pts, positive direction | < +10 pts OR negative |
| Boys avg shift (any age group) | ≤ ±5 pts | > +5 pts or < −5 pts |
| Girls ECNL avg shift | < 5 pts | > 10 pts |
| Girls non-GA avg shift | < 5 pts | > 10 pts |
| Total teams > 50-pt shift | < 80 teams | > 100 teams → investigate |
| ECNL–GA ASPIRE delta direction | NARROWING or stable | WIDENING by > 10 pts from pre-fix |

---

## References

- `Rankings/April 29 Analysis Session — ELO Execution Package.md` — execution checklist this template captures results from
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — prediction values for Phase 2 comparison
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — stability criteria for Phase 5 baseline
- `Rankings/April 29 Post-Fix Decision Tree.md` — G1–G4 gate definitions
- `Infrastructure/April 28 Execution Log.md` — G1 gate source
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Phase 6 update target

*ELO — 2026-04-25*
