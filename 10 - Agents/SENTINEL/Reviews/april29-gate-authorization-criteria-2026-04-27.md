---
title: April 29 — SENTINEL Gate Authorization Criteria
type: sentinel-reference
author: ELO
date: 2026-04-27
status: ready
purpose: SENTINEL decision reference for April 29 gate authorization
tags: [sentinel, april-29, gate-authorization, calibration, evo-draw]
---

# April 29 SENTINEL Gate Authorization Reference

> **For SENTINEL use.** ELO fills the Gate Results Structured Log on April 29. SENTINEL reads this document alongside the log to make the authorization decision. Total review time: ~5 minutes.

---

## G0 — Pre-Session Readiness

**What ELO checks:** Confirms April 28 execution is complete before running any gate queries. Checks for: April 28 Execution Log filed, all 3 execution steps marked YES, FORGE schema confirmation filed.

**PASS condition:** April 28 Execution Log and FORGE Schema Confirmation both exist and show completion.

**SENTINEL's role:** None. ELO self-authorizes G0. If G0 fails, ELO escalates to SENTINEL immediately and stops — no gates run.

---

## G1 — Source Data Gate

**What ELO checks:** Two SQL queries confirm the GA ASPIRE `event_tier` fix executed correctly.
- Query 1: `SELECT COUNT(*) FROM events WHERE event_name ILIKE '%ASPIRE%' AND event_tier = 'ga'` → **Expected: 0**
- Query 2: `SELECT COUNT(*) FROM events WHERE event_name ILIKE '%ASPIRE%' AND event_tier = 'ga_aspire'` → **Expected: > 0**
- Also checks `ecnl_verified` column exists on the games table.

**PASS condition:** Query 1 returns 0 AND Query 2 returns > 0.

**SENTINEL's role:** Confirm G1 PASS/FAIL before reviewing other gates. **If G1 fails, do not authorize anything.** ELO must re-run the GA ASPIRE UPDATE SQL and confirm execution before proceeding to G2–G4.

---

## G2 — Rating Distribution Gate

**What ELO checks:** Post-fix cohort average ratings by age group, compared against ELO's pre-committed predictions (written 2026-04-25 in the Pre-Run Model — values cannot be changed on April 29).

**Pre-committed expectations:**
| Age Group | Expected Direction | Expected Cohort Avg Shift |
|-----------|-------------------|--------------------------|
| U13G | INCREASE | +5 to +15 pts |
| U14G | INCREASE | +8 to +18 pts |
| U15G | INCREASE | +6 to +15 pts |
| U16G | INCREASE | +4 to +12 pts |
| U17G | INCREASE | +2 to +8 pts |
| Boys (all ages) | NO CHANGE | 0 ± 5 pts |

**PASS condition:** All Girls age groups show positive direction AND magnitude within predicted range. Boys delta ≤ 5 pts.

**SENTINEL's role:** If any Girls group shows NEGATIVE direction → G2 hard fail; do not authorize. If a Girls group is slightly outside the upper bound (e.g., U14G shows +22 pts instead of +18 pts) but direction is correct → SENTINEL may authorize with a flag if ELO provides an explanation. If Boys shows > 5 pts → investigate before authorizing.

---

## G3 — Calibration Stability Gate

**What ELO checks:** Anomaly detection against pre-committed thresholds from the Pre-Run Model.

**Pre-committed thresholds:**
| Signal | Pass Threshold | Hard Fail Condition |
|--------|---------------|---------------------|
| Girls GA ASPIRE avg change direction | Must be POSITIVE | Negative direction = hard fail |
| Girls GA ASPIRE shift (U13G/U14G) | +10 to +40 pts | — |
| Girls ECNL avg change | < 5 pts | > 10 pts = investigate |
| Boys avg change (any age group) | 0 ± 5 pts | > 15 pts = hard fail |
| Teams with > 50-pt shift total | < 80 teams | > 80 teams = investigate |

**PASS condition:** All signals within stated thresholds.

**SENTINEL's role:** If Boys > 15 pts → hold all authorizations; escalate to Presten. If > 80 large-movers → require ELO to identify specific teams before SENTINEL signs off.

---

## G4 — Baseline Snapshot Delta Gate

**What ELO checks:** Comparison of post-fix values against the pre-April-28 snapshot taken by Presten (from `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md`).

**Key rows SENTINEL checks:**
- Total ranked teams: delta should be 0 (no teams should have been dropped from rankings)
- Girls GA ASPIRE avg rating by age group: should show positive delta consistent with G2
- Girls ECNL avg rating: should be near-zero delta (< 5 pts)
- Boys top-10 avg rating: should be near-zero delta (0 ± 5 pts)

**PASS condition:** No row shows a delta outside the Pre-Run Model's predicted range. Specifically: Girls GA ASPIRE must show positive delta; ECNL and Boys must show near-zero delta.

**SENTINEL's role:** G4 also establishes the baseline ELO uses for Event Strength Phase 1 validation. SENTINEL notes the G4 baseline values — these become the Phase 1 comparison reference. If G4 fails (e.g., total ranked teams decreases), flag to Presten before authorizing any downstream work.

---

## SENTINEL Authorization Criteria

After reviewing ELO's filled Gate Results Structured Log:

- **All 4 gates GREEN → SENTINEL signs immediately.** Event Strength Phase 1 is authorized. ELO notified in SENTINEL's next briefing.

- **G1 FAIL (ASPIRE fix not confirmed) → Do not sign.** SENTINEL contacts Presten: "April 28 execution incomplete — GA ASPIRE UPDATE did not run. G2–G4 results are against a pre-fix database and are invalid. Session must be re-run after April 28 fix executes."

- **G2 hard fail (Girls age group shows NEGATIVE direction) → Do not sign.** Escalate to Presten. The fix may have applied backward. ELO investigates scope of GA ASPIRE UPDATE before any further sessions.

- **G3 hard fail (Boys > 15 pts movement) → Do not sign.** Hold Phase 1. ELO identifies the source of Boys rating change before any Phase 1 or Boys Option A work proceeds.

- **G2/G3 FLAGGED (values outside range but no hard fail) → SENTINEL reviews ELO's explanation.** If ELO provides a credible explanation (e.g., "U14G shift was +22 pts because 3 high-exposure teams had 30+ ASPIRE games — within the individual team expected range"), SENTINEL may authorize with a note. If explanation is absent or implausible → hold.

- **G4 fail only (distribution anomaly but G1–G3 pass) → SENTINEL may conditionally authorize Phase 1** while requiring ELO to investigate the G4 anomaly. Log the conditional in the authorization field.

---

## What SENTINEL Is Authorizing

When SENTINEL signs the authorization field in the Gate Results Structured Log, SENTINEL is authorizing:

1. **Event Strength Phase 1** — FORGE proceeds with Phase 1 event strength assignments. ELO runs Phase 1 ELO Execution Package validation queries after FORGE execution.
2. **Boys Option A window opens** — ELO begins monitoring Boy Option A query results (window: April 29 – May 5).
3. **Calibration stability monitoring begins** — ELO's post-fix monitoring spec activates; ELO reports stability checks in each briefing.

SENTINEL is NOT authorizing Event Strength Phase 2 (separate authorization process, criteria filed in Phase 2 Authorization Criteria document) or Boys calibration changes (dependent on Option A verdict).

---

*ELO — 2026-04-27 | Task: task-2026-04-27-april29-sentinel-gate-authorization-prep*
