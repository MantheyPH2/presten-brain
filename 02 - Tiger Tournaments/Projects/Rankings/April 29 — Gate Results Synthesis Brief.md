---
title: April 29 — Gate Results Synthesis Brief
type: synthesis-brief
author: ELO
date: 2026-04-27
status: template — ELO fills on April 29 after all gates complete
related: "[[April 29 Gate Results — Structured Log]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]", "[[April 29 Post-Fix Decision Tree]]"
tags: [calibration, ga-aspire, april-29, synthesis, gates, evo-draw]
---

# April 29 — Gate Results Synthesis Brief

> **[Template — ELO fills on April 29. Section 1 prediction column pre-filled from Pre-Run Model. Do not pre-fill "Actual" or "Match" columns before running the gates.]**

---

## Header

```
Session date: April 29, 2026
Filled by: ELO (based on Presten query output)
Pre-run model reference: Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md
Gate log reference: Rankings/April 29 Gate Results — Structured Log.md
```

---

## Section 1: Pre-Run Model vs. Actual Results

After running all 4 gates, ELO fills the "Actual" and "Match" columns. The "Pre-Run Prediction" column is pre-filled from the Pre-Run Model (written April 25).

| Signal | Pre-Run Prediction | Actual (April 29) | Match? |
|--------|-------------------|-------------------|--------|
| Girls GA ASPIRE avg rating direction | **INCREASE** (positive shift) | | Yes / No / Partial |
| Girls GA ASPIRE shift magnitude (U13G) | **+10 to +40 pts** | | Within range / Outside |
| Girls GA ASPIRE shift magnitude (U14G) | **+10 to +40 pts** | | Within range / Outside |
| Girls GA ASPIRE shift magnitude (U15G) | **+10 to +40 pts** | | Within range / Outside |
| Girls GA ASPIRE shift magnitude (U16G) | **+10 to +35 pts** | | Within range / Outside |
| Girls GA ASPIRE shift magnitude (U17G) | **+10 to +30 pts** | | Within range / Outside |
| Boys avg rating change | **0 pts ± 5 pts** | | Yes / No |
| Girls ECNL avg rating change | **< 5 pts** | | Yes / No |
| GA ASPIRE event_tier remaining as 'ga' | **0 rows** | | Confirmed / Failed |
| Total teams with > 50-pt shift | **< 80 teams** | | Within / Exceeded |

**Interpretation guidance (ELO fills on April 29):**

If any signal shows "Outside" or "No": one sentence stating whether the deviation changes the interpretation of the fix, or indicates an execution failure vs. a modeling error.

---

## Section 2: Gate Verdicts Summary

| Gate | Description | Result | Verdict |
|------|-------------|--------|---------|
| G1 | Source Baseline — GA ASPIRE event_tier reclassified, DB state confirmed | | PASS / FAIL / HOLD |
| G2 | Girls rating distribution post-fix — shifts in predicted direction and range | | PASS / FAIL / HOLD |
| G3 | Calibration stability — ECNL/MLS NEXT minimal movement, outlier count < 80 | | PASS / FAIL / HOLD |
| G4 | Boys stability — Boys avg change ≤ ±5 pts, no unintended scope | | PASS / FAIL / HOLD |

**Overall session verdict (ELO selects one):**

- **ALL PASS** — April 28 fix worked as predicted. Calibration is stable. Proceed to Phase 1 authorization.
- **PARTIAL PASS** — ___ of 4 gates passed. Named exception(s): [state]. Phase 1 authorization pending named resolution.
- **FAIL** — Gate [X] failed with result [Y]. Root cause: [ELO's assessment]. Phase 1 authorization held. SENTINEL briefed for Presten escalation.

**ELO selection:** [ELO fills April 29]

---

## Section 3: ECNL CP1 Result

Run before G1–G4 (first action in session).

- ECNL RL staleness (days since last game): [ELO fills April 29]
- CP1 verdict: **PASS** (≤ 30 days) or **FAIL** (> 30 days)
- If FAIL: Option 1 is DISQUALIFIED. State current ECNL migration option recommendation: [ELO fills]
- If PASS: Option 1 remains viable. Remaining checkpoints: CP2 (score integrity), CP3 (unique team IDs).

---

## Section 4: Event Strength Phase 1 Authorization Recommendation

ELO's recommendation to SENTINEL based on gate results:

- If all G1–G4 pass: **ELO recommends authorizing Event Strength Phase 1.** GA ASPIRE calibration is confirmed stable. Girls ratings are in expected post-fix range. No open blockers.
- If any gate fails: **ELO recommends holding Phase 1 authorization until [specific condition].** State the specific gate failure and what must be resolved.
- If G1 fails: **Phase 1 cannot proceed.** Calibration baseline is not established. ELO files blocker with SENTINEL before running any downstream analysis.

**ELO recommendation:** [AUTHORIZE / HOLD — ELO fills April 29]

**Reason:** [one sentence — ELO fills April 29]

---

## Section 5: One-Paragraph Summary for Presten

*(4–6 sentences, plain English, no jargon. SENTINEL relays this to Presten verbatim or with minor adaptation. ELO writes on April 29 after all gates complete.)*

[ELO fills April 29 — covers: (1) whether the April 28 fix worked as expected, (2) whether calibration is stable post-fix, (3) whether Phase 1 is authorized or what is blocking it, (4) what comes next — Boys Option A verdict window April 29–May 5.]

---

## Filing Checklist (ELO completes April 29)

- [ ] All "Actual" and "Match" cells in Section 1 filled from Gate Results Structured Log
- [ ] Section 2 gate verdicts completed with Overall selection marked
- [ ] Section 3 ECNL CP1 result recorded
- [ ] Section 4 authorization recommendation selected with reason stated
- [ ] Section 5 one-paragraph summary written (plain English, complete sentences)
- [ ] ELO briefing includes: "April 29 Synthesis Brief filed. Overall verdict: [ALL PASS / PARTIAL / FAIL]. Phase 1 authorization: [ELO recommends AUTHORIZE / HOLD]. Presten summary: [Section 5 first two sentences]."

---

## References

- `Rankings/April 29 Gate Results — Structured Log.md` — raw gate results this brief synthesizes
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — predictions ELO compares against
- `Rankings/April 29 Post-Fix Decision Tree.md` — G1–G4 gate criteria
- `Rankings/April 29 — ELO Pre-Session Readiness Check.md` — pre-session gate this brief follows
- `Rankings/Event Strength Phase 1 — ELO Execution Package.md` — Phase 1 authorization Section 4 recommends or holds

*ELO — 2026-04-27 (template pre-filed)*
