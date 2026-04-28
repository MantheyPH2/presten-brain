---
title: Post-DSS Calibration Sprint Plan
type: sprint-plan
author: ELO
date: 2026-04-27
status: pre-DSS — activate May 17
related: "[[Post-DSS Calibration Roadmap — June 2026]]", "[[June 1 Age Transition — Risk Register]]", "[[Boys Option A — Post-Pass Implementation Timeline]]"
tags: [calibration, sprint, post-dss, june-1, roadmap, evo-draw]
---

# Post-DSS Calibration Sprint Plan

> **Purpose:** Ordered, executable sprint plan for all calibration work after DSS (May 16). Converts the Post-DSS Calibration Roadmap into a sequenced queue with session estimates, authorization dependencies, and June 1 constraints. Presten and SENTINEL use this to know what comes next and how long it takes.
>
> **Activation date:** May 17, 2026 (day after DSS). ELO and FORGE begin Sprint 1 immediately — no new design session needed.

---

## Section 1: Deferred Items Inventory

All confirmed post-DSS calibration items:

| Item | Why Deferred | Dependency | Estimated Sessions | June 1 Risk |
|------|-------------|------------|-------------------|-------------|
| U13/U14 calibration fix (K-factor + RD floor changes) | DSS stability freeze — 6,310+ teams affected; too large a change pre-demo | None (all pre-conditions met) | 1 Presten execution + 1 ELO validation | **High** — must complete before May 28 to validate before June 1 |
| Boys Option A calibration execution | Boys spot check window April 29–May 5; results needed before implementation | Option A PASS verdict filed | 1 ELO design + 1 Presten execution + 1 ELO validation | **Medium** — if verdict received by May 5, implementable May 6–10 with full validation window before June 1 |
| MLS NEXT Boys tier split deployment | FORGE event classifier not yet run; separate from Option A | FORGE classifier pre-flight ≥ 90% quality | 1 FORGE classifier run + 1 Presten execution + 1 ELO validation | **Medium** — must complete by May 28 (not combine with June 1 transition) |
| Boys GA ASPIRE Brier analysis (cal=100 vs. ~70) | Brier analysis requires post-fix stable Boys baseline | April 28 fix complete + Boys baseline stable | 1 ELO analysis + potential 1 ELO recommendation + 1 Presten execution if change warranted | **Low** — if Brier recommends change, must apply before May 31 |
| Tier 2 undefined leagues cal (if deferred from April 29) | Possible if April 29 volume < 500 games | April 29 volume check result | 1 Presten session (30 min) | **Low** — low volume means minimal June 1 interaction |
| Event Strength Phase 2 | Phase 1 must be complete + calibration stable post-May-18-20 | Phase 1 validated + 7-day post-cal stability window | 1 FORGE implementation + 1 ELO validation | **None** — Phase 2 targeted June 9–15 |
| Boys Club Rankings (if Option A deferred) | Boys calibration must be stable first | Boys Option A verdict | Depends on Option A timing | **Medium** — if deferred to Girls-only at DSS, Boys can be added in June |
| Team merges high-priority audit (incomplete) | April 29 was dense; audit is ongoing | Presten session availability | 1–2 Presten sessions | **Low** — merge corrections are continuous, not sprint-dependent |
| Post-May-1 Rankings Impact analysis (in progress) | Requires 3 days of live pipeline data (May 1–3) | May 4 completion (already planned) | 1 ELO analysis session | **None** — completing during pre-DSS window |

---

## Section 2: Sprint Order

### Sprint 1 — May 17–21 (First week post-DSS)

**Items:** Authorization-ready the moment DSS closes. No new analysis required.

| Item | Action | Owner | Session Estimate |
|------|--------|-------|-----------------|
| U13/U14 calibration fix | Apply code change to `compute-rankings.js` (K-factor + RD floor params); run full recompute | Presten (execution), ELO (validation) | 1 execution session (Presten: ~60–90 min); 1 validation session (ELO autonomous: ~45 min) |
| Boys Option A execution (if PASS by May 5) | Apply any confirmed Boys calibration changes per Decision Document PASS branch; or file "no change needed" confirmation | ELO (design + SQL), Presten (execution), ELO (verification) | ~45 min ELO + 30–60 min Presten session; see Boys Option A Post-Pass Implementation Timeline |
| MLS NEXT Boys tier split — FORGE pre-flight | FORGE runs event classifier pre-flight; confirms ≥ 90% classification quality | FORGE | FORGE session (pre-authorized per Post-DSS Roadmap) |

**Sprint 1 ELO workload:** 2 autonomous validation sessions (~45 min each) + review FORGE pre-flight results.

**Sprint 1 Presten workload:** 1 execution session for U13/U14 fix (~60–90 min) + 1 session for Boys calibration (30–60 min if changes needed).

**Sprint 1 entry condition:** U13/U14 code change applied to `compute-rankings.js` by May 16 (Presten applies the specific K-factor and RD floor changes per `task-2026-04-23-u13-u14-calibration-fix.md` before May 17 deployment session).

---

### Sprint 2 — May 22–28

**Items:** Depend on Sprint 1 results. Execute after Sprint 1 validation clears.

| Item | Action | Owner | Session Estimate | Dependency |
|------|--------|-------|-----------------|------------|
| U13/U14 validation (ELO clears) | Validate U13/U14 rating shifts are within expected range; confirm no age group suppressed below its expected range | ELO | Autonomous — ~45 min | U13/U14 recompute complete May 17–21 |
| MLS NEXT Boys tier split deployment | Presten applies UPDATE and recompute after FORGE confirms ≥ 90% classifier quality | FORGE + Presten + ELO | FORGE: 1 session; Presten: ~60–80 min; ELO validation: ~30 min | FORGE pre-flight PASS (Sprint 1) |
| Boys Option A validation (if execution ran Sprint 1) | Confirm Boys/Girls delta ≤ 75 pts; MLS NEXT anchor stable; Girls unchanged | ELO | Autonomous — ~30 min | Boys execution complete Sprint 1 |
| Boys GA ASPIRE Brier analysis | Run gender-separated Brier on Boys GA ASPIRE games to confirm cal=100 vs. potential cal=70 | ELO (analysis), Presten (DB queries) | 1 Presten query session (~30 min DB); 1 ELO analysis session (~45 min) | Boys baseline stable (April 29 fix + Boys Option A execution) |
| Event Strength Phase 2 — preparation | ELO files Phase 2 authorization request after Phase 1 validation cleared | ELO | Autonomous — ~30 min | Phase 1 validated; post-April-28 stability 7-day window complete; post-May-18-20 stability 7-day window complete |

**Sprint 2 hard constraint:** Do not run any recompute within 48 hours of a previous recompute. Interleaved recomputes create attribution noise. Sequence: U13/U14 validation (May 20–22) → MLS NEXT deployment (May 22–24) → MLS NEXT validation (May 24–27) → Boys GA ASPIRE Brier (May 25–28).

---

### Sprint 3 — May 29–31 (Pre-June-1 window)

**Items:** Final calibration state before June 1 age group transition.

| Item | Action | Owner | Session Estimate |
|------|--------|-------|-----------------|
| Boys GA ASPIRE Brier analysis verdict | ELO files PASS (retain cal=100) or FAIL (revise to cal=~70) | ELO | Filing only if analysis complete by May 28 |
| Boys GA ASPIRE cal revision (if Brier FAIL) | Apply cal change and targeted Boys recompute | Presten (execution), ELO (validation) | 30–45 min Presten; 30 min ELO |
| Pre-migration DB backup | `pg_dump youth_soccer --table=rankings > backup_pre_migration_2026-05-31.dump` | Presten | ~15 min |
| All-clear confirmation | ELO confirms all active calibration flags cleared; calibration is in the validated state for June 1 | ELO | Briefing note only |

**Sprint 3 output:** Calibration is fully deployed, validated, and stable. June 1 runs on a clean baseline.

---

## Section 3: June 1 Transition Risk Register

Items that interact with the June 1 age group transition (`age_group_advance()`):

| Calibration Item | Must Complete Before June 1? | Risk if Missed | Safe to Defer Past June 1? |
|-----------------|-----------------------------|-----------------|-----------------------------|
| U13/U14 calibration fix | **Yes — by May 28** | If deployed after June 1, validation is confounded: cannot separate calibration rating changes from age-transition rating changes. U13 teams transitioning to U14 on June 1 would carry ratings shaped partly by the old K-factor, partly by the new — making post-transition validation ambiguous. | No. Must deploy and validate before May 31. |
| Boys Option A execution | **Yes — by May 28** | Boys age groups transition June 1. If Boys calibration is mid-investigation or mid-deployment at June 1, the transition creates a second source of Boys rating movement. Post-transition Boys ratings cannot be cleanly attributed to calibration or transition. | Option A confirms (Case A): no change needed, no risk. Option A confirms targeted change (Case B): must apply and validate before May 31. |
| MLS NEXT Boys tier split | **Yes — by May 28** | Same as Boys Option A: MLS NEXT Boys U14 teams transitioning to U15 on June 1 should carry validated, correctly-split ratings. Deploying the tier split after June 1 means the U15 cohort has a mix of split and unsplit ratings. | Deferrable to June 7 if FORGE classifier pre-flight fails by May 17. Apply immediately post-transition, before the June 7 pipeline run. |
| Boys GA ASPIRE Brier analysis revision | **Target May 30** | If cal=70 revision applied after June 1, Boys GA ASPIRE teams transitioning age groups carry ratings computed at cal=100. Post-transition validation is confounded. | Lower risk than the above — Boys GA ASPIRE is a precision improvement, not a known miscalibration. If Brier analysis completes June 3–7, apply the revision before the June 14 pipeline run with acceptable validation window. |
| Event Strength Phase 2 | **No — post-DSS, targeted June 9–15** | Phase 2 is post-transition by design. The June 1 transition is not a risk for Phase 2 — Phase 2 runs on the post-transition baseline. | Yes — Phase 2 should wait until June 9–15 per the Phase 2 Prerequisites spec. |

**Hard constraint (from Post-DSS Calibration Roadmap §2):** Do NOT combine any calibration recompute with the June 1 migration session. If any Sprint 3 item slips past May 31, defer to June 7 — do not run it concurrently with June 1.

---

## Section 4: Authorization Roadmap

| Item | Authorization Required | Suggested Timing | Authorization Format |
|------|----------------------|-----------------|---------------------|
| U13/U14 calibration fix code change | Presten applies the code change to `compute-rankings.js` | Before May 17 deployment session | Code change already specified in `task-2026-04-23-u13-u14-calibration-fix.md`; Presten applies per that spec |
| Boys Option A execution (if changes needed) | Presten one-line approval in session or briefing response | After ELO files Boys UPDATE SQL (May 5–6) | "Authorized to apply Boys calibration change per Option A Decision Document PASS branch." |
| MLS NEXT tier split deployment | Presten approves after FORGE confirms ≥ 90% classifier quality | After FORGE pre-flight PASS (Sprint 1) | "FORGE classifier confirmed ≥ 90%. Authorized to deploy MLS NEXT tier split." |
| Boys GA ASPIRE Brier — cal revision (if needed) | SENTINEL reviews; Presten authorizes | After ELO Brier analysis complete (May 25–30) | ELO files recommendation with specific cal value; Presten approves in next session |
| Event Strength Phase 2 | SENTINEL authorization per Phase 2 Authorization Criteria doc | After all Phase 2 prerequisites met (~June 5–9) | ELO submits formal authorization request per `Rankings/Event Strength Phase 2 — Authorization Criteria.md` |
| Team merge audit corrections | Presten reviews and approves specific merge flags | Ongoing during Presten sessions | Per-merge authorization in session |

---

## Section 5: Session Estimate Summary

| Phase | ELO Sessions (autonomous) | Presten Sessions | Target Completion |
|-------|--------------------------|-----------------|------------------|
| Sprint 1 (May 17–21) | 2 validation sessions (~45 min each) | 2 execution sessions (~90 min + ~60 min) | May 21 |
| Sprint 2 (May 22–28) | 3 validation sessions + 1 Brier analysis (~45 min each) | 1 MLS NEXT execution (~80 min) + 1 Boys Brier query session (~30 min) | May 28 |
| Sprint 3 pre-June-1 (May 29–31) | 1 all-clear session (~15 min) + 1 Boys GA ASPIRE verdict (if Brier complete) | 1 pre-migration backup + 1 Boys GA ASPIRE fix execution (if needed, ~30 min) | May 31 |
| June 1 transition | Monitor only (no calibration sessions on June 1) | June 1 transition execution (~30 min per Risk Register) | June 1 |
| Post-transition (June 2–15) | Phase 2 validation + Boys club rankings (if Girls-only at DSS) | Phase 2 FORGE session + Boys club rankings execution | June 15 |
| **Total (calibration complete)** | **~8–10 ELO sessions** | **~6–8 Presten sessions** | **~June 15** |

**The answer to "how long does post-DSS calibration take?":** 4 weeks of active work from May 17 to June 15, with the most intensive period May 17–28 (4–5 Presten sessions + 5–6 ELO sessions). After June 15, calibration is fully deployed and monitoring transitions to the standing weekly review cadence.

---

## Section 6: One-Paragraph DSS Demo Summary

> "After this demo, we have several calibration improvements staged and ready to deploy. In the two weeks following DSS, we'll be applying a K-factor improvement for U13 and U14 age groups, deploying the MLS NEXT tier split for Boys — which will correct a ~20–40 point overcalibration for Academy-track Boys teams — and completing the Boys calibration spot check that gates Boys club rankings. All of these changes are already designed and validated at the spec level; they're deferred only to keep rankings stable for the demo. Our target is to have all calibration work completed by June 1, when the annual age-group transition occurs. After June 1, we'll be running daily pipeline with fully validated, gender-separated calibration covering all major leagues."

---

## References

- `Rankings/Post-DSS Calibration Roadmap — June 2026.md` — source roadmap this sprint plan operationalizes
- `Rankings/June 1 Age Transition — Risk Register.md` — June 1 timing constraints (especially R10: no combined calibration + migration)
- `task-2026-04-23-u13-u14-calibration-fix.md` — U13/U14 K-factor and RD floor parameters
- `Rankings/Boys Option A — Post-Pass Implementation Timeline.md` — Boys session detail
- `Rankings/Event Strength Phase 2 — Authorization Criteria.md` — Phase 2 authorization gate
- `Rankings/Boys Brier Analysis Scope Spec — May 2026.md` — Boys GA ASPIRE Brier analysis scope
- `Rankings/MLS NEXT Tier Split — Calibration Application Spec.md` — MLS NEXT tier split spec

*ELO — 2026-04-27*
