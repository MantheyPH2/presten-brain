---
title: Boys Brier Analysis — Pre-May-17 Readiness Check
type: readiness-checklist
author: ELO
date: 2026-04-27
status: complete
related: "[[Boys Brier Analysis Scope Spec — May 2026]]", "[[Boys Calibration Status Summary — April 2026]]"
tags: [boys, brier, calibration, readiness, post-dss, evo-draw]
---

# Boys Brier Analysis — Pre-May-17 Readiness Check

> **Purpose:** Confirm everything ELO needs is ready before the May 17 Boys Brier analysis window opens. Written April 27 — when there is time to identify and close gaps. Read Section 4 for the go/no-go check on May 17 morning.

---

## Section 1: What the Boys Brier Analysis Requires

Status assessed as of 2026-04-27.

| Requirement | Status | Evidence / Source |
|-------------|--------|-------------------|
| Scope spec completed (age groups, calibration questions, success criteria) | **MET** | `Rankings/Boys Brier Analysis Scope Spec — May 2026.md` — covers Boys GA ASPIRE only (cal=100 vs ~70), 2024–2026 seasons, ≥ 300 cross-tier games minimum, Brier delta threshold 0.008 |
| SQL queries written for Brier calculation (Step 1–4) | **MET** | Scope spec Section 3 — four-step analysis method with Brier formula |
| SQL query written for cross-tier game count check | **MET** | Scope spec Section 2 — two queries: Boys GA ASPIRE game count and cross-tier game count |
| ECNL Boys vs GA Boys win-rate gap measurement SQL | **MET** | `Rankings/ECNL-Boys-GA-Boys-Win-Rate-Gap-Investigation-2026-04-27.md` Step 3 — full query written, interpretation thresholds defined |
| Pre-analysis Boys baseline snapshot plan | **MET** | April 29 post-fix analysis (Decision Tree G4 gate) establishes Boys avg rating change baseline. April 29 gate results confirm Boys are unaffected by GA ASPIRE fix. Boys Option A queries (if passed) provide a secondary pre-Brier snapshot. |
| Boys Option A outcome accounted for in Brier scope | **PENDING** | Boys Option A verdict not yet filed. Query window opens April 29. Two Brier interpretations documented: If PASS → Brier tests whether cal=70 improves over the already-validated cal=100 baseline. If FAIL → calibration revision may precede Brier analysis. |
| Threshold for "Boys calibration confirmed" defined | **MET** | Scope spec Section 5: delta < 0.003 → retain cal=100; 0.003–0.008 → borderline (review age groups); > 0.008 → revise to cal=70; < 300 games → data-limited, retain cal=100 |
| Decision tree for each outcome documented | **MET** | Scope spec Section 5 — all four delta ranges have explicit verdicts. If data-limited: retain 100, defer to post-June 1 season data. |
| Girls analysis framework to replicate (methodology) | **MET** | `02 - Tiger Tournaments/Projects/Reports/calibration-held-out-validation-2026-04-23.md` — Boys analysis is adaptation of this framework |
| Brier computation code adapted for Boys | **PARTIAL** | The Girls framework exists and the adaptation is documented in scope spec. However, the Boys-specific held-out game set has not been extracted — that requires Presten DB access (~30 min). |

**Items MET:** 8 of 10
**Items PARTIAL:** 1 (Brier code Boys adaptation — execution step only)
**Items PENDING:** 1 (Boys Option A verdict — resolves April 29)

---

## Section 2: What May Change Before May 17

### Boys Option A Verdict (April 29)

**How it changes Brier scope:**

| Verdict | Brier Scope Impact |
|---------|--------------------|
| **PASS** (Boys vs Girls distribution within 75-pt divergence; top-10 U13B credible) | Brier proceeds as scoped: test cal=100 vs cal=70. Boys K_base is validated. |
| **FAIL** (divergence > 100 pts or implausible top-10) | Boys calibration has a detected problem. Brier analysis must shift: first determine *what* is wrong (age group? merge artifact? cal value?). The May 17–30 window may start with diagnosis rather than the GA ASPIRE Brier comparison. Scope needs revision before May 17. |
| **INCONCLUSIVE** (results borderline, Presten judgment call) | Treat as PASS for Brier planning purposes. SENTINEL or Presten must explicitly confirm scope before May 17. |

**ELO action if FAIL verdict:** File a scope revision note in the May 5 briefing identifying what the Brier analysis needs to investigate instead of (or before) the GA ASPIRE comparison.

### May 1 Pipeline Deployment

New game data ingested after April 28 will be computed with the corrected `ga_aspire` cal value. By May 17, the pipeline will have run ~17 more times. For the Boys Brier analysis:
- More Boys GA ASPIRE games in the dataset → better cross-tier game count (approaches 300 threshold more closely)
- Boys ratings continue to shift as new games are processed → the pre-Brier Boys baseline should be captured as close to May 17 as possible, not from April 29

**ELO action:** On May 16 (after DSS), run Boys baseline snapshot queries to capture the pre-Brier state. File snapshot results in that day's briefing.

### April 29 Gate Outcomes

If the April 29 G4 gate (Boys rating change check) shows Boys avg change > 15 pts, the GA ASPIRE fix had unintended Boys scope. This would require investigation before the Brier analysis can proceed.

Expected April 29 G4 result: Boys avg change ≈ 0 pts (cal=100→100, no Boys net impact). If this expectation holds, no scope impact on Brier.

---

## Section 3: Gap Inventory

| Gap | Closing Action | Who | When |
|-----|---------------|-----|------|
| Boys Option A verdict not filed | Run 3 Option A queries in April 29 psql session; ELO files verdict | Presten (DB), ELO (verdict) | April 29 |
| Cross-tier Boys game count not verified | Run game count query from Scope Spec Section 2 | Presten (DB) | May 17 morning (first Brier action) |
| Boys-specific held-out game set not extracted | Pull Boys cross-tier game set in first psql session of Brier window | Presten (DB) | May 17 |
| Pre-Brier Boys baseline snapshot | Run Boys avg rating snapshot queries on May 16 after DSS | Presten (DB), ELO (log results) | May 16 (post-DSS) |
| Brier scope revision (if Boys Option A FAIL) | ELO files scope revision note in May 5 briefing | ELO | May 5 (if FAIL) |

**What ELO cannot close before May 17:** The data-dependent steps (game count verification, held-out set extraction) require live DB access. These are the first 30 minutes of the May 17 session, not a gap.

**What ELO CAN close before May 17 without Presten:** All documentation, SQL, and planning gaps are closed as of this document.

---

## Section 4: May 17 Entry Conditions

**ELO confirms the following on May 17 morning before beginning Brier computation:**

1. **April 28 fix confirmed complete** — Boys GA ASPIRE events are tagged `event_tier = 'ga_aspire'` in events table. (Confirmed via April 29 gate results briefing.)

2. **Boys Option A verdict filed** — `Rankings/Boys Option A — Verdict Filing Document.md` has a final verdict (PASS or FAIL). If FAIL: Brier scope revision from May 5 briefing is in hand.

3. **April 29 G4 gate confirmed GREEN** — Boys avg rating change post-April-28 < 15 pts. (Filed in April 29 gate results briefing.)

4. **May 16 Boys baseline snapshot run** — Boys avg rating by age group captured before Brier begins, so Brier results can be expressed as a delta from the pre-analysis state.

5. **Cross-tier game count ≥ 300** — Run the scope spec Section 2 count query in the first psql session of May 17. If < 300: the analysis is data-limited. Retain cal=100, defer to June 2026 after more Boys GA ASPIRE games accumulate. Do not proceed with underpowered Brier analysis.

6. **DSS concluded** — Presten confirms DSS session complete (May 14–15 event). No active demo obligations. Ratings can move without demo risk.

**If any entry condition is NOT met:** Stop. ELO determines what is unresolved and files an updated scope note before beginning. Do not begin Brier computation with an unresolved Boys calibration question — the results will be uninterpretable.

---

## References

- `Rankings/Boys Brier Analysis Scope Spec — May 2026.md` — primary scope document
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys calibration state going into this analysis
- `Rankings/ECNL-Boys-GA-Boys-Win-Rate-Gap-Investigation-2026-04-27.md` — win-rate gap SQL and verdict
- `Rankings/Boys Option A — Verdict Filing Document.md` — Boys prerequisite gate
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — April 29 query execution plan
- `02 - Tiger Tournaments/Projects/Reports/calibration-held-out-validation-2026-04-23.md` — Girls Brier methodology to replicate

*ELO — 2026-04-27*
