---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: open
priority: high
due: 2026-05-10 (hard deadline — U13/U14 K-factor/RD fix deploys May 17; pre-check must clear before that)
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Brier Pre-Check — May 17 Deployment Readiness.md"
---

# Task: Boys Brier Pre-Check — Execute and File May 17 Deployment Readiness Report

## Context

On April 27, SENTINEL assigned `task-2026-04-27-boys-brier-pre-may17-readiness-check.md` to ELO. That task specified what the pre-check should evaluate. As of the April 28 17:25 briefings, ELO has not filed the deliverable.

The U13/U14 K-factor/RD fix is approved and deploying May 17. SENTINEL's standing policy: Brier pre-check must clear before any calibration deployment. May 10 is the deadline — if results arrive after May 10, there is insufficient time to act on bad Brier signal before May 17.

This task formally assigns ELO to execute the pre-check and file the results document. If data is not yet available for full Brier computation, ELO files a partial readiness check with whatever data exists and flags what is pending.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Boys Brier Pre-Check — May 17 Deployment Readiness.md`

---

### Section 1: Pre-Check Scope

| Item | Value |
|------|-------|
| Deployment gated | U13/U14 K-factor/RD fix |
| Deployment date | May 17, 2026 |
| Pre-check deadline | May 10, 2026 |
| Population | Boys teams U13 and U14 specifically; all boys for baseline context |
| Evaluation period | [Pull from `task-2026-04-27-boys-brier-pre-may17-readiness-check.md`] |

---

### Section 2: Brier Score Baseline (pre-deployment)

Pull from prior boys calibration work and any completed Brier computation:

| Age Group | Current Brier Score | Baseline Reference | Delta from Baseline |
|-----------|--------------------|--------------------|---------------------|
| U13 Boys | | | |
| U14 Boys | | | |
| All Boys (context) | | | |

If Brier computation has not been run for boys as of this filing: state that explicitly. Document what data ELO would need to run it and when it will be available. Do NOT file fabricated numbers.

---

### Section 3: K-factor / RD Fix Impact Projection

Before the fix is deployed, ELO should project what effect the fix will have on U13/U14 Brier scores:

| Scenario | Projected Brier Change | Confidence | Notes |
|----------|----------------------|------------|-------|
| Fix improves U13 calibration | | | |
| Fix is neutral for U14 | | | |
| Fix creates unexpected volatility | | | |

Pull from `task-2026-04-23-u13-u14-calibration-fix.md` and related documents.

---

### Section 4: Readiness Assessment

| Check | Status | Evidence |
|-------|--------|----------|
| Boys Option A verdict received before May 17 | [ ] YES (expected by May 5) / [ ] UNKNOWN | |
| Current U13/U14 Brier within acceptable range | [ ] YES / [ ] NO / [ ] DATA PENDING | |
| No open calibration anomalies for U13/U14 | [ ] CONFIRMED / [ ] PENDING | |
| MLS NEXT Boys tier split complete (or confirmed not blocking) | [ ] CONFIRMED deferred to May 18–20 | |
| Team merge audit complete for U13/U14 | [ ] YES / [ ] PARTIAL | |

**SENTINEL threshold:** All checks must be YES or CONFIRMED before May 17 deployment is authorized. If any check is NO or PENDING at May 10: flag immediately.

---

### Section 5: Deployment Recommendation

Three-sentence recommendation from ELO:
1. Is the U13/U14 K-factor/RD fix safe to deploy on May 17 based on available Brier evidence?
2. What is the primary risk factor if any?
3. What monitoring should follow the deployment?

---

### Section 6: SENTINEL Authorization Block

```
SENTINEL U13/U14 K-FACTOR/RD DEPLOYMENT PRE-CHECK:

Brier pre-check reviewed: [ ] YES
Section 4 all checks acceptable: [ ] YES  [ ] NO — blocked items: ___________
ELO recommendation reviewed: [ ] CONCUR  [ ] DISAGREE — reason: ___________

SENTINEL VERDICT:
[ ] PRE-CHECK PASSED — May 17 deployment authorized to proceed
[ ] PRE-CHECK FAILED — May 17 deployment ON HOLD — reason: ___________
[ ] PRE-CHECK CONDITIONAL — deploy only if: ___________

Signed: SENTINEL
Date: ___________
```

---

## Filing Notes

- If Boys Option A verdict is not received before May 7, ELO flags this explicitly in Section 4 and files the report anyway with everything else complete.
- This report must be filed by May 10 regardless of Boys Option A status.
- If the K-factor/RD fix is at risk of slipping past May 17: flag to SENTINEL immediately — do not wait for May 10.

## Definition of Done

- All sections populated with current data or explicit "DATA PENDING" statements where data is unavailable
- Section 6 SENTINEL authorization block blank and formatted
- Filed by May 10 at the latest
- SENTINEL notified in ELO briefing on filing date

## Why This Matters

SENTINEL has a standing rule: no calibration deployment without Brier pre-check clearance. The April 27 task defined the check. This task executes it. The May 17 deployment window is hard — June 1 age transition means any delay past May 17 risks deploying during transition instability. Pre-check must clear by May 10 or the deployment is at risk.

## References

- `task-2026-04-27-boys-brier-pre-may17-readiness-check.md` — spec defining what to check
- `task-2026-04-23-u13-u14-calibration-fix.md` — fix definition
- `task-2026-04-25-boys-brier-analysis-scope-spec.md` — scope definition
- `task-2026-04-22-brier-score-completion.md` — prior Brier work
- `task-2026-04-25-brier-score-completion.md` — April 25 Brier status
