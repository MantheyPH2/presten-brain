---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Post-Pass Implementation Timeline.md"
---

# Task: Boys Option A — Post-Pass Implementation Timeline

## Context

The Boys Option A Pre-Verdict Evidence Checklist (filed Apr 25) defines the PASS criteria for the Boys calibration spot-check. The Boys Calibration Status Summary and the Boys Option A Spot-Check Execution Package both assume Option A runs on April 29 and a verdict is filed by May 4 (after 3 data days).

What does not exist: a pre-written implementation timeline that activates the moment the May 4 verdict is "PASS." The verdict filing document (filed Apr 25) captures the verdict. But what happens the day after? Which ELO sessions run, in what order, to actually implement Boys calibration in production? Without a pre-written timeline, a PASS verdict on May 4 triggers an unscheduled design conversation — exactly the kind of friction that adds a week of latency to an already tight DSS schedule (May 16).

Write the implementation timeline now, before April 29. If Option A passes, ELO activates this document immediately. If Option A fails, this document is filed as a reference and the Girls-Only Fallback Spec activates instead.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Post-Pass Implementation Timeline.md`

---

### Section 1 — Activation Trigger

This document activates when ALL of the following are true:
- [ ] Boys Option A Pre-Verdict Evidence Checklist: PASS (filed by May 4)
- [ ] April 29 G2 gate: Girls GA ASPIRE rating distribution confirmed healthy
- [ ] April 29 G3 gate: Calibration stability check GREEN
- [ ] SENTINEL has not issued a hold on Boys implementation

If any trigger is not met: this timeline does not activate. File in briefing: "Boys Option A timeline not activated — [missing trigger]."

---

### Section 2 — Implementation Session Plan

After a PASS verdict, Boys calibration implementation proceeds in three sessions:

**Session 1 — Boys Calibration Values Commit (target: May 5–6)**

ELO's primary work: formalize the Option A calibration values into the production calibration table. This is not a new design — Option A values were defined in the Calibration Spec and spot-checked on April 29. This session applies them.

Steps:
1. Pull Option A calibration values from `Rankings/Boys Calibration Status Summary — April 2026.md` and `Rankings/MLS NEXT Tier Split Calibration Spec.md`
2. Write the production UPDATE SQL (analogous to the GA ASPIRE fix UPDATE run on April 28)
3. Stage for Presten review: file SQL in `Rankings/Boys Calibration — Production UPDATE SQL.md`
4. ELO files with SENTINEL for review before Presten runs it

Estimated time: 45 minutes autonomous (ELO writes SQL); 30–60 minutes Presten session to review and run.

**Session 2 — Post-Application Verification (target: May 7–8)**

After Presten runs the Boys calibration UPDATE, ELO runs verification:
1. Query average Boys rating by tier (MLS Next, NPL, ECNL Boys) — confirms cal values applied
2. Compare Boys G1–G3 tier separation against the Option A predictions from April 29
3. Cross-gender sanity check: Boys MLS Next average should be approximately aligned with Girls MLS Next average (within [ELO specifies tolerance])

ELO defines the pass criteria for this verification here. If fail: escalate to SENTINEL.

Estimated time: 30 minutes (query execution + analysis).

**Session 3 — Boys Calibration Stability Baseline (target: May 9, during the May 9 DSS readiness session)**

On May 9, ELO adds Boys calibration to the existing stability monitoring queries:
- Add Boys MLS Next average to Query 1 (currently Girls-only)
- Add Boys volatility count to Query 2
- Boys baseline filed alongside the Girls May 9 stability check

No standalone session needed — integrate into the May 9 readiness session.

---

### Section 3 — Deliverables Checklist

| Deliverable | Owner | Due | Filed? |
|------------|-------|-----|--------|
| Boys Calibration — Production UPDATE SQL | ELO | May 5–6 | [ ] |
| SENTINEL review of Boys UPDATE SQL | SENTINEL | May 6 | [ ] |
| Presten runs Boys calibration UPDATE | Presten | May 7 | [ ] |
| Post-Application Verification report | ELO | May 7–8 | [ ] |
| Boys stability baseline filed | ELO | May 9 | [ ] |

ELO fills this table as each item completes, and references it in daily briefings.

---

### Section 4 — DSS Readiness Impact

With Boys Option A implementation complete by May 9, Evo Draw's DSS-readiness claims for Boys rankings are:

| Claim | Status after implementation |
|-------|---------------------------|
| Boys MLS Next calibrated | ✅ |
| Boys NPL calibrated | ✅ (if in Option A scope — ELO confirms) |
| Boys ECNL calibrated | ✅ (or N/A — ELO confirms scope) |
| Club Rankings (Boys) | Conditional: depends on Boys calibration stability confirmed by May 9 |

If Boys Option A implementation completes on schedule, Boys rankings are DSS-ready by May 9 — one week before the May 16 demo. ELO flags this in the May 9 readiness assessment.

---

### Section 5 — Failure Contingency Reference

If, after May 4 verdict, Option A is PASS but post-application verification (Session 2) FAILS:
- ELO files incident note to SENTINEL immediately
- Boys calibration is rolled back (reverse the UPDATE SQL from Session 1)
- Girls-Only Fallback Spec activates for DSS demo
- ELO files updated DSS Feature Readiness Tracker within 24 hours of the failure detection

This section is not a design document — it is a reference pointer. The Girls-Only Fallback Spec already contains the full contingency.

---

## Definition of Done

- All 5 sections present
- Session 1 includes a draft or outline of the Boys calibration UPDATE SQL structure (ELO can write the full SQL now — the cal values are already defined)
- Session 2 pass criteria are explicit numbers (not "approximately aligned")
- Deliverables checklist is formatted as a fill-in tracker
- DSS readiness impact table is filled for the Boys tiers ELO has data for

## Why This Matters

The DSS schedule has no slack. If Option A passes May 4, ELO cannot spend May 5 designing the implementation — that design must be written before April 29. This document converts a PASS verdict into a same-day implementation kickoff. The difference between a pre-written timeline and an improvised one is 3–5 days of latency on a 12-day runway.

## References

- `Rankings/Boys Option A — Pre-Verdict Evidence Checklist.md` — PASS criteria that trigger this timeline
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Option A calibration values
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — contingency if Option A fails post-application
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — stability monitoring to extend to Boys in Session 3
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Boys calibration readiness flag
