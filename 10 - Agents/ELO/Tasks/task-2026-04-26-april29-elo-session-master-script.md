---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: urgent
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Rankings/April 29 ELO Session Master Script.md"
---

# Task: April 29 ELO Session Master Script

## Context

ELO's own briefing identifies 8 items for the April 29 session — the highest-density ELO session since April 22. These 8 items have sequential dependencies that are not documented anywhere. Specifically:

- CP1 (ECNL staleness) must run **first** — if CP1 fails (Option 1 disqualified), ELO must escalate before spending time on other tasks
- G1 (recompute confirmed) must pass before calibration stability baseline and Decision Tree G2–G4 can run
- G2 data from the Decision Tree feeds both the Girls Calibration Narrative AND the Prediction vs. Actual Assessment Part B — if ELO writes the narrative before G2 is documented, it has no evidentiary basis
- Boys Option A tables and Team Merges classification are optional (contingent on Presten filing data) — but ELO must check whether data is present before starting main items

Without a sequenced master script, ELO may:
- Run G1–G4 Decision Tree before CP1 (risks wrong option frame if ECNL is disqualified)
- Start filling Girls Calibration Narrative sections before G2 data is documented (out of order)
- Miss the Boys Option A check because it's buried in step 7 of an 8-item list

The master script converts this into a decision-tree execution plan with time budgets.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/April 29 ELO Session Master Script.md`

This is a single linear document. ELO opens it at the start of the April 29 session and works top to bottom. If a step's prerequisite is not met, the script branches explicitly.

---

### Pre-Session Gate (Before Starting)

```
[ ] April 28 execution log confirmed filed (FORGE or Presten confirms)
[ ] April 28 recompute confirmed complete (Step 3 of execution log ✅)
[ ] G1 gate passable: If recompute is not confirmed, STOP — file briefing noting "April 29 Decision Tree blocked at G1. Rescheduling all dependent steps to next session."
```

If G1 is blocked: file briefing, note all dependent steps held, end session.

---

### Step 1: CP1 — ECNL RL Staleness Check (10 min)

Reference: `Rankings/ECNL RL Staleness Verification — Presten Execution Package.md`

Run the staleness query. Document result: [RL staleness in days].

**Branch:**
- **CP1 PASS (< 30 days):** Continue to Step 2. Note "ECNL Option 1 viable" in session log.
- **CP1 FAIL (≥ 30 days):** File CP1 FAIL note in `Rankings/ECNL Migration Evidence Collection.md`. Escalate to SENTINEL Queue immediately: "CP1 FAIL — ECNL Option 1 disqualified. ELO requests SENTINEL escalate ECNL migration option decision to Presten." Then continue to Step 2 (CP1 failure does not block calibration analysis).

File CP1 result in `Rankings/ECNL Migration Evidence Collection.md` — Checkpoint 1.

---

### Step 2: Calibration Stability Baseline (30 min)

Reference: `Rankings/Post-Fix Calibration Stability Monitoring Spec.md`, `Reports/GA ASPIRE Calibration Stability — Baseline 2026-04-29.md` (template pre-filled by ELO April 26)

Open the pre-filled template. Run Queries 1, 2, and 3. Fill the results tables. Label this run "Fix Delta Run — not used for stability comparison. Stability tracking starts April 30."

Complete the baseline status assessment block at the bottom of the file.

**This step's output is required for Step 3 G2.** Specifically: Query 1 results (Girls GA ASPIRE average rating by age group) are the G2 observed rating distribution.

---

### Step 3: April 29 Decision Tree G1–G4 (20 min)

Reference: `Rankings/April 29 Post-Fix Decision Tree.md`, `Rankings/April 29 Decision Tree — Verdict Filing Document.md`

Work through G1 → G2 → G3 → G4. Pull G2 observed values from Step 2 Query 1 results. Pull G3 inputs (Boys rating changes) from the recompute output.

For each gate: fill the verdict line in the Verdict Filing Document. Decision Tree is complete when G4 is filled and verdict is filed.

**Branch after G4:**
- **All gates PASS:** Event Strength Phase 1 authorized pending SENTINEL review. Continue to Step 4.
- **G2 FAIL:** File immediately in next briefing. SENTINEL holds Phase 1. Girls calibration fix may need revision — do not continue to Step 4 until SENTINEL reviews.
- **G3 FAIL:** Boys ratings shifted unexpectedly. File briefing flag and continue — G3 fail does not block Girls calibration narrative but does flag Boys calibration for SENTINEL attention.

---

### Step 4: Girls Calibration Narrative — Sections 2–6 (30 min)

Reference: `Rankings/Girls Calibration Post-April-28 Narrative Template.md`

**Prerequisite:** Step 3 G2 documented (actual Girls GA ASPIRE rating distribution in hand).

Fill Sections 2–6 using G2 data as the evidentiary foundation. Section 2 is the observed rating shift. Section 3 is the distribution comparison (Girls GA vs. ECNL/MLS NEXT). Sections 4–6 are narrative interpretation.

Section 1 (Executive Summary) should be filled LAST — after Sections 2–6 are complete.

---

### Step 5: Prediction vs. Actual Assessment Part B (15 min)

Reference: `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md`, `Rankings/April 29 Prediction vs. Actual — Assessment.md`

Compare Step 2 Query 1 actual results against the Pre-Run Model's Section 3 Predicted Rating Shift Ranges.

For each Girls age group: fill [Predicted Direction, Predicted Magnitude, Actual Direction, Actual Magnitude, Assessment: Consistent / Inconsistent / Within margin].

If any age group shows "Inconsistent": flag in next briefing with one-paragraph analysis. Do not hold further steps — this is documentation, not a gate.

---

### Step 6: CP2 — ecnl_verified Column Confirmation (5 min)

Reference: `Infrastructure/Source Ingestion Baseline Snapshot.md` — Section 4 (filed by FORGE earlier in this session or April 29 morning)

**Prerequisite:** FORGE has filed Source Baseline Section 4.

Confirm ecnl_verified column status from FORGE's Section 4 Query B result. File CP2 result in `Rankings/ECNL Migration Evidence Collection.md` — Checkpoint 2.

CP2 is a 5-minute read, not a query. ELO reads FORGE's filing and records the result.

---

### Step 7 (Conditional): Boys Option A Results (variable)

**Check first:** Has Presten filed Boys Option A query results?
- Query results location: `Rankings/Boys Option A — Decision Document.md` (Presten fills results tables)
- If results present: ELO fills the two-paragraph analysis and selects PROCEED / ACTIVATE FALLBACK / CONDITIONAL PROCEED. File decision.
- If results not present: note in briefing "Boys Option A window open — awaiting Presten query results. Estimated earliest: April 29 afternoon or April 30."

---

### Step 8 (Conditional): Team Merges Classification (variable)

**Check first:** Has Presten filed Team Merges Audit output?
- Output location: `Rankings/Team Merges — Presten Execution Package.md` or a separate report file
- If output present: Apply `Rankings/Team Merges — Audit Classification Framework.md` to batch results. File classification summary.
- If output not present: note in briefing "Team Merges audit output not yet received from Presten."

---

### Post-Session Filing

After completing all applicable steps:

1. Write April 29 briefing. Include: CP1 result, calibration baseline status, G1–G4 verdicts, Girls Calibration Narrative sections filed, Prediction vs. Actual assessment result, CP2 result, Boys Option A status, Team Merges status.
2. Flag any FAIL conditions clearly at top of briefing (SENTINEL reads flags first).
3. Update DSS Feature Readiness Tracker for any items that moved to ✅.

---

### Session Time Estimate

| Step | Estimated Time |
|------|----------------|
| Pre-session gate | 5 min |
| Step 1: CP1 | 10 min |
| Step 2: Calibration baseline | 30 min |
| Step 3: Decision Tree | 20 min |
| Step 4: Girls Calibration Narrative | 30 min |
| Step 5: Prediction vs. Actual | 15 min |
| Step 6: CP2 | 5 min |
| Step 7/8 (conditional) | 20–40 min if data in |
| Briefing | 15 min |
| **Total (without conditionals)** | **~130 min (~2.2 hrs)** |
| **Total (with conditionals)** | **~170 min (~2.8 hrs)** |

---

## Quality Standard

- Every step must have a clear prerequisite — no step may begin if its prerequisite is not met
- Every step must have a documented output location — no step ends without a file being updated
- Conditional steps (7 and 8) must have explicit "data not present" branches that result in a briefing note, not silence
- Time estimates are guides; ELO should not rush Step 2 (baseline accuracy is load-bearing for all subsequent monitoring)

## Why This Matters

SENTINEL flags April 29 as the highest-density ELO session since April 22. Eight items, several with dependencies, several with SENTINEL authorization implications (G4 → Phase 1, CP1 → ECNL escalation). If ELO runs these in the wrong order, SENTINEL receives an out-of-sequence report and cannot confirm that Phase 1 is properly authorized. The master script costs ELO 20 minutes to write and eliminates 3 hours of potential session confusion or re-work.

## References

- `Rankings/ECNL RL Staleness Verification — Presten Execution Package.md`
- `Rankings/ECNL Migration Evidence Collection.md`
- `Reports/GA ASPIRE Calibration Stability — Baseline 2026-04-29.md` (ELO pre-filled template)
- `Rankings/April 29 Post-Fix Decision Tree.md`
- `Rankings/April 29 Decision Tree — Verdict Filing Document.md`
- `Rankings/Girls Calibration Post-April-28 Narrative Template.md`
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md`
- `Rankings/Boys Option A — Decision Document.md`
- `Rankings/Team Merges — Audit Classification Framework.md`
