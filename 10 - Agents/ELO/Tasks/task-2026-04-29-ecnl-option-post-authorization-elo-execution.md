---
type: agent-task
assigned_by: SENTINEL
assigned_to: ELO
date: 2026-04-29
priority: high
due: "Within 30 minutes of Presten authorization"
status: pending
topic: ecnl-option-post-authorization-elo-execution
---

# Task: ECNL Option — Post-Authorization ELO Execution (30-Minute Ready)

## Objective

Pre-stage ELO's action steps for both ECNL migration options so that ELO can execute within 30 minutes of receiving Presten's authorization decision. The Decision Brief recommends Option 1 (No Re-tag), but ELO must be ready for either.

Authorization deadline: **April 30 EOD**. ELO should have both paths staged before that deadline.

## Background

ELO filed `Rankings/ECNL Migration Option — Decision Brief.md` recommending Option 1. FORGE filed `Infrastructure/ECNL Migration — Option 1 Implementation Card.md` covering FORGE's actions under Option 1. ELO's specific post-authorization actions are staged in `task-2026-04-28-ecnl-option-a-implementation-prep-package.md` but there is no unified 30-minute execution card that covers both options in one place.

## Deliverable

File `Rankings/ECNL Migration — ELO Post-Authorization Execution Card.md` with:

### Header

"Authorization received from Presten: Option [1/2]. ELO executes the relevant path below within 30 minutes."

### Path A — Option 1 Authorized (No Re-tag)

**ELO actions:**
1. Update `ECNL Migration Option — Decision Brief.md` with authorization date and decision (2 min)
2. Confirm ECNL Migration Handoff Checklist (`task-2026-04-28-ecnl-migration-elo-forge-handoff-checklist.md`) reflects Option 1 path — no team re-tagging, CP1 monitors for ingestion continuity only (5 min)
3. Set CP1 checkpoint date in vault: June 1 + 48 hrs (FORGE confirms game ingestion, then ELO runs CP1 analysis)
4. Update May 9 DSS Risk Register R3 (ECNL migration rating continuity) to "OPTION 1 SELECTED — LOW RISK" (3 min)
5. Notify SENTINEL in next briefing: "ECNL Option 1 authorized. ELO CP1 set for June 3." (1 min)

**Total: ~15 min**

### Path B — Option 2 Authorized (Re-tag to ecnl_legacy)

**ELO actions:**
1. Update `ECNL Migration Option — Decision Brief.md` with authorization date and decision (2 min)
2. File CP1 checkpoint plan: ELO reviews rating distribution before and after schema change. Requires FORGE to run ALTER TABLE and UPDATE SQL first. (10 min to draft CP1 criteria)
3. File ELO ECNL Rating Continuity Spec (`task-2026-04-26-ecnl-migration-rating-continuity-spec.md`) as active — activate CP1 and CP2 checkpoint sequence
4. Update May 9 DSS Risk Register R3 to "OPTION 2 SELECTED — MEDIUM RISK (schema dependency)" (3 min)
5. File a SENTINEL Queue item: "Option 2 selected — FORGE schema change creates CP1 dependency. ELO cannot complete CP1 until FORGE ALTER TABLE runs. Estimated CP1 window: June 1–3." (5 min)
6. Notify SENTINEL in next briefing (1 min)

**Total: ~25 min**

### Decision Record Section

```
Authorization received: [ ] April 29  [ ] April 30  [ ] Not yet
Option authorized: [ ] Option 1 — No Re-tag  [ ] Option 2 — Re-tag ecnl_legacy
Authorized by: Presten
ELO execution start time: _______________
ELO execution complete: _______________
SENTINEL notified: _______________
```

## Notes

- This card is ELO's equivalent of FORGE's Option 1 Implementation Card — standalone, one-page, no document cross-reference required at execution time.
- If authorization arrives with conditions or clarifications outside Options 1 and 2 as defined, ELO flags to SENTINEL before executing.
- If no authorization by April 30 EOD, ELO files a SENTINEL Queue item noting the missed deadline and the June 1 preparation window that is now at risk.
