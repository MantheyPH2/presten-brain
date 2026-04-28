---
title: June 1–2 Sequencing Risk Register
type: risk-register
author: ELO
date: 2026-04-27
status: active
related: "[[June1-Age-Transition-Procedure-2026-04-27]]", "[[ECNL Migration — Option B Technical Risk Deep Dive]]", "[[Post-DSS Sprint — Week 1 Execution Calendar]]"
tags: [june-1, june-2, ecnl-migration, age-transition, risk, sequencing, evo-draw]
---

# June 1–2 Sequencing Risk Register

> **Context:** ELO's April 27 vault research (filed `June1-Age-Transition-Procedure-2026-04-27.md`) confirmed a dependency: the June 1 age group transition migration MUST complete before the June 2 ECNL platform migration (TGS → GotSport). If June 1 slips, June 2 must slip by the same margin. The acceptable slip window is June 7–10. This register formalizes the dependency, analyzes slip scenarios, and defines SENTINEL escalation criteria.

---

## Section 1: Dependency Map

```
Boys Brier analysis verdict (target: May 25–30)
              |
              v (if revision required)
Boys GA calibration change applied + recompute (target: May 26–28)
              |
              v
[7-day pre-migration checklist window]
              |
              v
June 1: Age Group Transition Migration (Presten, ~3 hours)
  - Schema changes (ALTER TABLE)
  - Backfill (UPDATE games, UPDATE rankings)
  - compute-rankings.js full recompute (~65–90 min)
  - Post-migration validation (30 min)
              |
              v
June 2: ECNL Platform Migration (TGS → GotSport)
  - Option 1 or Option 2 per ECNL decision gate
  - Option B: ID re-mapping for ECNL teams (new GotSport IDs)
  - Requires ECNL teams to already be in their new age groups (June 1 must be done)
              |
              v
July 1: New season starts (new GotSport team IDs live, new age groups active)
```

| Step | Owner | Estimated Time | Latest Acceptable Date | Trigger |
|------|-------|----------------|----------------------|---------|
| Boys Brier verdict | ELO | Analysis complete by May 30 | May 30 | Boys Option A window closes May 5 |
| Boys cal change (if needed) | Presten + ELO | 1 session, ~2 hrs | May 28 | Boys Brier verdict recommends revision |
| Pre-migration checklist | Presten | ~1 hr prep | May 29–31 | Scheduled before June 1 session |
| June 1 transition | Presten | ~3 hrs | June 10 | Calendar block June 1; slip window to June 10 |
| June 2 ECNL migration | Presten + FORGE | ~2 hrs | June 11 | June 1 transition confirmed complete |
| July 1 season start | System | Automated | July 1 (hard) | Both migrations confirmed complete |

---

## Section 2: Slip Scenario Analysis

### Scenario A: June 1 → June 7 (1-week slip)

**Cause:** Boys Brier results require a targeted calibration adjustment (Boys GA ASPIRE: cal=100 → ~70). Implementation takes 3–5 days plus stability check, pushing Presten's transition session from June 1 to June 7.

**Impact on June 2 ECNL migration:**
June 2 becomes June 8. June 8 is within the acceptable slip window (≤ June 10). ECNL migration proceeds on June 8 without further escalation.

**Impact on July 1 season start:**
June 8 ECNL migration + July 1 deadline = 23 days. The ECNL migration itself takes ~2 hours plus FORGE validation. Post-migration ELO rating continuity verification adds 1–2 days. Total: June 8 migration completes by June 10 with all checks done. 21 days remain before July 1 — sufficient buffer. **July 1 is safe in Scenario A.**

**SENTINEL action:** If Boys Brier verdict (May 25) indicates > 3-day implementation window, notify Presten by May 25 that June 1 is likely slipping to June 7. Reschedule June 1 block to June 7. No further escalation needed.

---

### Scenario B: June 1 → June 10 (1.5-week slip — edge of acceptable window)

**Cause:** Boys Brier reveals a more significant calibration revision (e.g., MLS NEXT Homegrown calibration also needs adjustment). Implementation + 7-day stability window pushes migration to June 10.

**Impact on June 2 ECNL migration:**
June 2 becomes June 11. June 11 is 1 day outside the June 10 acceptable slip window. This requires Presten confirmation that June 11 is workable given the July 1 constraint.

**Impact on July 1 season start:**
June 11 migration completes by June 13 (allowing 2 days for FORGE validation). 18 days remain before July 1 — tight but workable for routine validation. **July 1 is at risk only if ECNL migration encounters issues after June 11.**

**SENTINEL action:** If it becomes clear by May 28 that the implementation window extends to June 8–10, escalate to Presten with this scenario. Confirm Presten can execute June 10 transition AND June 11 ECNL migration before committing to the slip. Do not wait until June 5 to escalate.

---

### Scenario C: June 1 → June 14+ (slip beyond acceptable window)

**Cause:** Boys Brier reveals a structural calibration problem (e.g., Boys ECNL calibration is wrong, not just Boys GA ASPIRE). Full recalibration cycle required. Investigation + implementation + validation extends past June 10.

**Impact on June 2 ECNL migration:**
June 2 cannot maintain any current date. New date must be negotiated with the June 10 wall in mind. If transition does not complete by June 10, ECNL migration must be rescheduled to June 15+ — and FORGE must assess July 1 feasibility.

**Impact on July 1 season start:**
June 14 transition + June 15 ECNL migration + validation = completion by June 17–18. 13 days remain before July 1. Doable only if ECNL migration is clean (no re-runs, no ID re-mapping issues). Any complication pushes toward July 1. **July 1 is at risk in Scenario C.**

**SENTINEL action:** This is an escalation. As soon as Boys Brier verdict (May 25–30) indicates a structural problem requiring a full recalibration cycle, file a queue item to Presten with the following decision request:
- Confirm whether July 1 is a hard deadline (external commitments) or can slip 1–2 weeks
- Approve whether to defer the June 1 transition entirely and run it in late June (June 20–25 window) to give the recalibration time to complete properly
- If July 1 is hard: Presten decides whether to run a partial calibration fix in time for June 1 and apply the full recalibration post-July 1, accepting temporarily inaccurate Boys rankings for the early season

---

## Section 3: Acceptable Window Definition

| Transition | Latest Acceptable Date | Source |
|------------|----------------------|--------|
| June 1 age transition | June 10 | `Rankings/Age Group Transition Migration Spec 2026-06-01.md` — pre-migration checklist note; "June 1 transition is not a hard public-facing deadline — internal data quality event; 1-week slip is acceptable" |
| June 2 ECNL migration | June 11 | Derived: June 2 follows June 1 by 1 day; the slip preserves that 1-day offset |
| July 1 season start | July 1 (hard) | External deadline — not ELO's to adjust |

**Why these limits:** The July 1 season start requires the new GotSport team IDs to be live in the Evo Draw database. The ECNL migration writes those IDs (via Option B ID re-mapping). Post-migration, FORGE and ELO run validation (1–2 days) and ELO runs a rating continuity spot-check (1 day). Minimum processing time between ECNL migration completion and July 1: **7 days**. Latest workable ECNL migration date: June 24. Latest workable June 1 transition date: June 23 (one day before ECNL migration). If the June 1 transition slips to June 14+, the June 24 outer limit is still achievable — but there is zero margin for error on the ECNL migration itself.

*ELO note: The June 24 outer limit is ELO's derivation. If Presten has external commitments that move the July 1 hard date, update this section with the corrected limit.*

---

## Section 4: Escalation Criteria

| Condition | SENTINEL Action | Timing |
|-----------|----------------|--------|
| Boys Brier verdict indicates > 5-day implementation window (Scenario A edge into B) | Alert Presten — June 1 slip likely to June 7. Reschedule session block. | Same session as Brier verdict (target May 25) |
| Boys Brier verdict indicates > 10-day implementation window (Scenario B → C) | Alert Presten — June 1 slip to June 10 or beyond. Confirm July 1 risk. | Same session as Brier verdict |
| Boys Brier verdict indicates structural calibration problem (full recalibration cycle) | Escalate immediately — July 1 at risk. File decision request per Scenario C SENTINEL action. | Within 2 hours of Brier verdict session |
| June 1 execution session starts but does not complete in planned 3-hour block | Alert Presten — confirm estimated time to completion and June 2 migration hold | Next session after June 1 begins |
| June 1 confirmed delayed by > 3 days from scheduled date | Alert Presten — June 2 must be rescheduled | As soon as delay is confirmed |
| Slip exceeds June 10 acceptable window without Presten confirmation | Escalate — July 1 season start at risk. Do not proceed with ECNL migration without explicit Presten guidance. | Immediately upon confirmation of > June 10 slip |

**SENTINEL rule:** Do not wait for SENTINEL's next scheduled briefing review to escalate Scenarios B or C. File a queue alert immediately when the condition is detected.

---

## Section 5: Open Questions for Presten

ELO researched these from vault documentation. One question requires Presten confirmation; the others are resolved.

**RESOLVED — from `Rankings/Age Group Transition Migration Spec 2026-06-01.md`:**

1. *Is the June 1 transition automatic or manual?* **Manual. Presten runs it. 3-hour block required.** (Confidence: HIGH — explicitly documented with psql commands.)

2. *Can the June 1 transition be executed on a different date within a window?* **Yes.** The spec states June 1 is the target, not a hard system event. Slipping to June 7–10 is documented as acceptable.

3. *Can June 1 and June 2 migrations be run in the same session?* **Not recommended.** The June 1 transition recompute (~65–90 min) outputs a new ranking state that the June 2 ECNL migration reads. Running both in sequence on the same day is technically possible if there are no June 1 validation failures, but increases risk. ELO recommends a minimum 24-hour gap to allow ELO to review June 1 results before the June 2 migration begins.

**REQUIRES PRESTEN CONFIRMATION:**

4. *What is the minimum processing/validation time between June 2 ECNL migration completion and July 1 season start?* ELO estimates 7 days (validation + rating continuity + FORGE confirmation). **Confirm this is sufficient given external commitments tied to July 1.** If Presten has a partner or customer dependency requiring new team IDs by an earlier date, this register needs to be updated with the earlier hard deadline.

---

## References

- [[June1-Age-Transition-Procedure-2026-04-27]] — procedure documentation confirming June 1 is manual
- [[ECNL Migration — Option B Technical Risk Deep Dive]] — June 1 simultaneous-change risk (ID change + age group transition for same teams)
- [[Post-DSS Sprint — Week 1 Execution Calendar]] — Boys Brier window (May 17–30) upstream dependency
- [[Boys Brier Analysis Scope Spec — May 2026]] — upstream analysis whose results trigger slip scenarios B and C
- [[Age Group Transition Migration Spec 2026-06-01]] — primary procedure source; defines acceptable slip window

*ELO — 2026-04-27*
