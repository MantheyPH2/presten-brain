---
title: April 29 ELO Session Master Script
type: session-master-script
author: ELO
date: 2026-04-26
status: ready — execute April 29 after April 28 recompute confirmed
due: 2026-04-29
related: "[[April 29 Analysis Session — ELO Execution Package]]", "[[ECNL Migration — Option Selection Decision Gate]]", "[[Post-Fix Calibration Stability Monitoring Spec]]", "[[April 29 Post-Fix Decision Tree]]", "[[Girls Calibration Status — Post-April-28 Narrative]]", "[[April 29 — Prediction vs Actual Match Assessment]]"
tags: [april-29, session-script, ecnl, calibration, ga-aspire, decision-tree, evo-draw]
---

# April 29 ELO Session Master Script

> **One document. Work top to bottom.** This script sequences all 8 April 29 items with explicit dependencies, branches, and time budgets. Do not open other spec documents during the session — references are cited inline.
>
> **Session estimate:** 130 min (steps 1–6 + briefing) to 170 min (with conditional steps 7–8).

---

## Pre-Session Gate (Before Starting — 5 min)

```
[ ] April 28 execution confirmed: GA ASPIRE events re-tagged (event_tier = 'ga_aspire')
[ ] Full ranking recompute completed after April 28 UPDATE
[ ] Pre-April-28 baseline snapshot file confirmed at: reports/pre-april28-*.txt
```

**If any pre-condition unchecked:** STOP. File briefing: "April 29 session blocked — April 28 execution not confirmed. All dependent steps held. Awaiting SENTINEL confirmation before proceeding." End session.

**If all three checked:** begin Step 1.

---

## Step 1: CP1 — ECNL RL Staleness Check (10 min)

**Why first:** CP1 determines whether ECNL Migration Option 1 is viable. If disqualified, ELO must escalate to SENTINEL before spending the session on calibration work. This is a 10-minute check with a high-stakes branch.

**Reference:** `Rankings/ECNL RL Staleness Verification — Presten Execution Package.md`

Run the ECNL RL staleness query from that package. Record result: **[ECNL RL staleness in days]**.

**Branch:**

- **CP1 PASS (staleness < 30 days):**
  - Note "ECNL Option 1 viable" in session log.
  - File CP1 result in `Rankings/ECNL Migration — Option Selection Decision Gate.md` → Checkpoint 1 row.
  - Continue to Step 2.

- **CP1 FAIL (staleness ≥ 30 days):**
  - Update Decision Gate: "Option 1: DISQUALIFIED — ECNL RL staleness > 30 days"
  - File Queue item immediately: `Queue/pending-2026-04-29-ecnl-option1-disqualified.md` (category: alert, priority: urgent). Content: "CP1 FAIL — ECNL RL staleness > 30 days. Option 1 disqualified. Option 2 is now the only viable path. Requires Presten sign-off. ELO requests SENTINEL escalate to Presten before May 10."
  - File CP1 result in Decision Gate.
  - **Continue to Step 2** — CP1 failure does not block calibration analysis.

---

## Step 2: Calibration Stability Baseline (30 min)

**Why second:** Step 3's G2 gate requires the Girls GA ASPIRE average ratings from Query 1 in this step. Do not run Step 3 before Step 2 is complete.

**Reference:** `Rankings/Post-Fix Calibration Stability Monitoring Spec.md`, `Reports/GA ASPIRE Calibration Stability — Baseline 2026-04-29.md`

Open the pre-filled template at `Reports/GA ASPIRE Calibration Stability — Baseline 2026-04-29.md`. Run all three queries from the Monitoring Spec (Section 2, Queries 1–3). Fill in the results tables in the baseline file.

**Important:** Label Query 2 results explicitly as "Fix Delta Run — not used for stability comparison. Stability tracking starts April 30." The large_movers count today reflects the recompute, not instability.

**This step's output required for Step 3:** Query 1 results (Girls GA ASPIRE average rating by age group) are the G2 observed rating distribution. Copy these values before moving to Step 3.

Complete the Baseline Status Assessment block at the bottom of the baseline file.

---

## Step 3: April 29 Decision Tree G1–G4 (20 min)

**Prerequisite:** Step 2 complete (Query 1 results in hand).

**Reference:** `Rankings/April 29 Post-Fix Decision Tree.md`, `Rankings/April 29 Decision Tree — Verdict Filing Document.md`

Work through G1 → G2 → G3 → G4. Use Step 2 Query 1 results as G2 observed values.

For each gate: fill the verdict line in the Verdict Filing Document. Decision Tree is complete when G4 is filled and verdict is filed.

**Branches after G4:**

- **All gates PASS (G1 ✅, G2 ✅, G3 ✅, G4 ✅):**
  - Event Strength Phase 1 authorized pending SENTINEL review.
  - Note in briefing: "G1–G4 all pass. Event Strength Phase 1 cleared from calibration gate."
  - Continue to Step 4.

- **G2 FAIL (Girls GA ASPIRE distribution wrong direction or > threshold):**
  - File in briefing immediately: "G2 FAIL — Girls rating distribution did not shift as expected. Investigating. Event Strength Phase 1 HELD."
  - Do not proceed to Step 4 until SENTINEL review. File briefing noting hold, end session.

- **G3 FAIL (Boys ratings shifted unexpectedly):**
  - Flag in briefing: "G3 FAIL — Boys rating shift > 5 pts post-recompute. Flagged to SENTINEL. Investigating Boys scope."
  - G3 fail does NOT block Girls calibration narrative — continue to Step 4.
  - SENTINEL holds Event Strength Phase 1 until Boys scope is cleared.

- **G4 FAIL (downstream features show unexpected distribution):**
  - Flag in briefing per Decision Tree specification.
  - Continue to Step 4 (G4 fail is a signal, not a session stopper for calibration analysis).

---

## Step 4: Girls Calibration Narrative — Sections 2–6 (30 min)

**Prerequisite:** Step 3 G2 documented. G2 actual Girls GA ASPIRE rating distribution must be in hand.

**Reference:** `Rankings/Girls Calibration Status — Post-April-28 Narrative.md`

Open the narrative template. Fill Sections 2–6 using Step 3 G2 data as the evidentiary foundation:
- Section 2: Observed rating shift (use Step 2 Query 1 actuals vs. pre-fix baseline)
- Section 3: Distribution comparison — Girls GA ASPIRE vs. ECNL/MLS NEXT (use Step 2 Query 3)
- Sections 4–6: Narrative interpretation, conclusions, monitoring outlook

**Fill Section 1 (Executive Summary) LAST** — after Sections 2–6 are complete.

If G2 failed (Step 3): do not fill Sections 2–6. Note "Deferred — pending G2 investigation." End session.

---

## Step 5: Prediction vs. Actual Assessment Part B (15 min)

**Prerequisite:** Step 2 complete (Query 1 actual results available).

**Reference:** `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md`, `Rankings/April 29 — Prediction vs Actual Match Assessment.md`

Open the Prediction vs. Actual Assessment document. Compare Step 2 Query 1 actual results against the Pre-Run Model's Section 3 Predicted Rating Shift Ranges.

For each Girls age group: fill Predicted Direction, Predicted Magnitude, Actual Direction, Actual Magnitude, Assessment (Consistent / Inconsistent / Within margin).

**Branch:**
- Any age group "Inconsistent": flag in briefing with one-paragraph analysis. Do not hold further steps — this is documentation, not a gate.
- All "Consistent" or "Within margin": note in briefing: "Prediction vs. actual: all age groups consistent with pre-run model."

---

## Step 6: CP2 — ecnl_verified Column Confirmation (5 min)

**Prerequisite:** FORGE has filed Source Ingestion Baseline Snapshot, Section 4 (expected: filed morning of April 29).

**Reference:** `Infrastructure/Source Ingestion Baseline Snapshot.md` — Section 4, Query B

Read FORGE's Section 4 filing. Confirm `ecnl_verified` column exists with correct default. Record result in `Rankings/ECNL Migration — Option Selection Decision Gate.md` → Checkpoint 2 row.

**If FORGE Section 4 not yet filed:** Note in briefing: "CP2 deferred — FORGE Source Baseline Section 4 not received. Will file CP2 result when available." Do not hold session for this.

---

## Step 7 (Conditional): Boys Option A Results (variable — 20–40 min if data in)

**Check first:** Has Presten filed Boys Option A query results in `Rankings/Boys Option A — Decision Document.md`?

- **Results present:**
  - Open the Decision Document. Fill Q1, Q2, Q3 results tables with Presten's output.
  - Write the two-paragraph ELO Analysis section (Section 3).
  - Select decision: PROCEED / ACTIVATE GIRLS-ONLY FALLBACK / CONDITIONAL PROCEED.
  - Fill Section 5 SENTINEL Authorization Request block.
  - If Q1 auto-fails (any age group > 100 pts divergence): activate Girls-only fallback immediately — update DSS Feature Readiness Tracker Boys cell to ❌ in the same session.
  - Note in briefing: "Boys Option A results filed. Decision: [PROCEED / FALLBACK / CONDITIONAL]."

- **Results not present:**
  - Note in briefing: "Boys Option A: awaiting Presten query results (day [N] of window, closes May 5). Execution package filed at `Rankings/Boys Option A Spot Check — Presten Execution Package.md`."

---

## Step 8 (Conditional): Team Merges Classification (variable — 20–40 min if data in)

**Check first:** Has Presten filed Team Merges Audit output?

- **Output present:**
  - Apply `Rankings/Team Merges — Audit Classification Framework.md` to classify batch results.
  - File classification summary.
  - Note in briefing: "Team Merges: [N] merges classified. High-risk: [N]. Verified: [N]. Pending: [N]."

- **Output not present:**
  - Note in briefing: "Team Merges audit output not yet received from Presten."

---

## Post-Session Filing

After completing all applicable steps:

1. Complete and file `Reports/GA ASPIRE Calibration Stability — Baseline 2026-04-29.md` if not already done in Step 2.
2. Update DSS Feature Readiness Tracker (`Product & Planning/DSS Feature Readiness Tracker — May 2026.md`):
   - Rank Bands calibration cell: ✅ if G3/G4 pass, ⚠️ if G3 borderline, ❌ if G2 fail
   - Event Strength Phase 1 calibration cell: ✅ if all gates pass, ❌ if G2 fail
   - Club Rankings Boys cell: per Step 7 result (✅ / ❌ / ⚠️ pending)
3. Write April 29 ELO briefing. Required content:
   - CP1 result (ECNL RL staleness: pass/fail, days stale)
   - Calibration stability baseline status (filed / deferred)
   - G1–G4 verdict (all pass / specific fail + action taken)
   - Girls Calibration Narrative sections filled (2–6 complete / deferred)
   - Prediction vs. Actual assessment result (consistent / inconsistent — cite age groups)
   - CP2 result (confirmed / deferred — reason)
   - Boys Option A status (decision filed / awaiting results — day N of window)
   - Team Merges status (classified / awaiting output)
   - DSS Readiness Tracker update confirmation
   - Any FAIL conditions flagged at top of briefing before all other content.

---

## Session Time Budget

| Step | Estimated Time |
|------|----------------|
| Pre-session gate | 5 min |
| Step 1: CP1 (ECNL staleness) | 10 min |
| Step 2: Calibration stability baseline | 30 min |
| Step 3: Decision Tree G1–G4 | 20 min |
| Step 4: Girls Calibration Narrative | 30 min |
| Step 5: Prediction vs. Actual | 15 min |
| Step 6: CP2 (ecnl_verified) | 5 min |
| Step 7/8 (conditional) | 0–40 min |
| Briefing | 15 min |
| **Total (steps 1–6 + briefing)** | **~130 min (~2.2 hrs)** |
| **Total (with both conditionals)** | **~170 min (~2.8 hrs)** |

---

## Quick Escalation Reference

| Condition | Action |
|-----------|--------|
| Pre-condition fails (recompute not confirmed) | STOP. File briefing. Do not run any steps. |
| CP1 FAIL (ECNL RL ≥ 30 days) | File Queue item alert immediately. Continue session. |
| G1 FAIL (April 28 fix not confirmed) | STOP. File G1 FAIL in briefing. All downstream steps held. |
| G2 FAIL (Girls distribution wrong direction) | File immediately. Hold Event Strength Phase 1. Do not proceed to Step 4. |
| G3 FAIL (Boys delta > 5 pts) | Flag in briefing. Hold Phase 1. Continue to Step 4. |
| Boys Option A Q1 FAIL (>100 pt divergence) | Activate Girls-only fallback immediately. Same session. |
| CP1 FAIL + G2 FAIL same session | Escalate both to SENTINEL. High-risk briefing. |

---

## References

- `Rankings/ECNL RL Staleness Verification — Presten Execution Package.md` — CP1 query
- `Rankings/ECNL Migration — Option Selection Decision Gate.md` — CP1 and CP2 filing target
- `Reports/GA ASPIRE Calibration Stability — Baseline 2026-04-29.md` — pre-filled template (Step 2)
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — monitoring queries (Step 2)
- `Rankings/April 29 Post-Fix Decision Tree.md` — G1–G4 gates (Step 3)
- `Rankings/April 29 Decision Tree — Verdict Filing Document.md` — verdict filing target (Step 3)
- `Rankings/Girls Calibration Status — Post-April-28 Narrative.md` — narrative template (Step 4)
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — predicted values (Step 5)
- `Rankings/April 29 — Prediction vs Actual Match Assessment.md` — assessment filing target (Step 5)
- `Rankings/Boys Option A — Decision Document.md` — decision template (Step 7)
- `Rankings/Team Merges — Audit Classification Framework.md` — classification framework (Step 8)
- `Rankings/April 29 Analysis Session — ELO Execution Package.md` — detailed query reference for all phases
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — tracker updated post-session

*ELO — 2026-04-26*
