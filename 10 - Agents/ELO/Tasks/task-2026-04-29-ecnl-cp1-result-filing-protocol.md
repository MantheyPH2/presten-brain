---
type: agent-task
assigned_by: SENTINEL
assigned_to: ELO
date: 2026-04-29
time: "11:15"
priority: high
due: "April 30 (ready before CP1 result arrives)"
status: completed
completed: 2026-04-29
topic: ecnl-cp1-result-filing-protocol
---

# Task: ECNL CP1 Result — Filing Protocol

## Objective

Pre-structure the CP1 result filing so ELO can complete it within 30 minutes of receiving CP1 data on April 30. Includes the Option Comparison Matrix update step and ELO recommendation confirmation or revision if CP1 data changes the picture.

## Background

ECNL CP1 slipped from April 29 to April 30 and is the first action of that session. CP1 tests whether Option 1 (No Re-tag) is viable from an ECNL data continuity standpoint. The result may confirm ELO's Option 1 recommendation or introduce evidence that shifts the recommendation toward Option 2. Either way, ELO must file the result immediately — Presten's April 30 EOD authorization decision depends on it.

The ECNL Decision Brief (`Rankings/ECNL Migration Option — Decision Brief.md`) is filed and clean. The Options Comparison Matrix is included in that brief. This protocol specifies what ELO updates and files on CP1 receipt.

## Deliverable

**File:** `Rankings/ECNL CP1 — Checkpoint Results.md`

## Required Content

### Section 1 — CP1 Test Results

| Check | Expected | Actual | Pass/Fail |
|-------|----------|--------|-----------|
| ecnl_verified backfill row count | > X rows | (fill) | (fill) |
| Teams correctly marked ecnl_verified = TRUE | All ECNL teams active this season | (fill) | (fill) |
| No ECNL teams incorrectly excluded | Zero false negatives | (fill) | (fill) |
| Rating continuity: sample team rating pre/post | No change (Option 1 = no re-tag) | (fill) | (fill) |

ELO fills the Expected column from `Rankings/ECNL Rating Continuity Spec.md` before CP1 runs. The Actual column fills on April 30.

### Section 2 — CP1 Verdict

**CP1 PASS:** All 4 checks pass → Option 1 remains the correct recommendation. No change to Decision Brief. Notify SENTINEL: "CP1 PASS — Option 1 recommendation confirmed. Presten can authorize."

**CP1 CONDITIONAL:** One check fails non-critically (e.g., count differs by <5%) → ELO notes the deviation, states whether it affects Option 1 viability, and recommends whether Presten should still authorize Option 1 or wait for CP2.

**CP1 FAIL:** A critical check fails (e.g., rating continuity broken, significant false negatives) → ELO immediately: (1) updates the Decision Brief with a STOP note at the top, (2) notifies SENTINEL, (3) files CP1 Results with FAIL verdict and recommended pivot to Option 2 analysis.

### Section 3 — Decision Brief Update (if needed)

If CP1 PASS: no update to Decision Brief required. Note in Section 3: "Decision Brief unchanged. CP1 PASS on [date]."

If CP1 CONDITIONAL or FAIL: ELO edits `Rankings/ECNL Migration Option — Decision Brief.md` Options at a Glance table to reflect the new data. Append a CP1 Results Addendum section at the bottom of the Decision Brief.

### Section 4 — ELO Recommendation Post-CP1

One sentence confirming or revising the Option 1 recommendation based on CP1 evidence. ELO does not hedge — state the recommendation clearly.

### Section 5 — SENTINEL Notification

"CP1 result filed. Recommendation: [Option 1 / Option 2 / HOLD pending CP2]. Presten authorization can proceed / should pause pending [reason]."

## Filing Sequence on April 30

1. April 30 session opens → CP1 check runs (FORGE executes per migration scope)
2. FORGE confirms ecnl_verified backfill row count and schema state
3. ELO receives FORGE confirmation → runs rating continuity check
4. ELO fills Sections 1–5 of this document
5. ELO updates Decision Brief if needed
6. ELO notifies SENTINEL via next briefing

## Notes

- ELO must fill the Expected column of Section 1 before April 30 — this is the only part of the document completable before CP1 runs; do it as part of filing this task
- The CP1 result does not change the April 30 EOD authorization deadline — if CP1 runs at any point April 30, ELO files within 30 minutes
- If the April 30 session does not open, ELO files a Queue item to SENTINEL noting CP1 slip to next session and the impact on authorization timing
