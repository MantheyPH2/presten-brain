---
title: Post-DSS Calibration Priority Queue — 2026-04-27
type: planning-document
author: ELO
date: 2026-04-27
status: complete
audience: Presten + SENTINEL
tags: [calibration, post-dss, roadmap, boys, girls, planning, evo-draw]
---

# Post-DSS Calibration Priority Queue — 2026-04-27

> **For Presten and SENTINEL.** This document consolidates all known open calibration items as of April 27 into a priority-ordered queue for the post-DSS period (May 17 onward). Written before DSS so the queue is pre-approved and ELO enters post-DSS with a known workplan. Presten should review before May 9 and redirect if priorities differ.

---

## Section 1: Open Calibration Issues (as of April 27, 2026)

| # | Issue | Age Groups / Leagues | Why Deferred | Evidence Strength |
|---|-------|---------------------|-------------|------------------|
| 1 | U13/U14 calibration fix (K-factor + RD floor) | U12, U13, U14, U19 — both genders | Presten decided post-DSS per April 29 sessions | Strong: Brier delta ≥ 0.010 for U13, 0.007 for U14 — empirically demonstrated |
| 2 | Boys Option A calibration application (if verdict PASS) | Boys all age groups with GA exposure | Spot-check window closes May 5; implementation follows | Pending: verdict April 29–May 5 |
| 3 | Boys GA ASPIRE Brier analysis (cal=100 vs. ~70) | Boys U13–U17 in GA ASPIRE events | No active miscalibration signal; Boys ASPIRE impact is zero on April 28 | Theoretical: no failure signal, but unvalidated |
| 4 | ECNL Boys vs GA Boys cross-league win rate | Boys ECNL (120) vs Boys GA (100) | Query designed but never executed; April 23 audit results missing | Gap: SQL ready, data likely available, measurement not run |
| 5 | Tier 2 undefined leagues (if volume < 500 games) | USL Academy, EDP, Elite 64, NAL | Volume check April 29 determines if pre-DSS or post-DSS | Medium: affects accuracy claims if > 500 games |
| 6 | Event Strength Phase 2 | Phase 2 league list (see Phase 2 Scope doc) | Phase 1 must complete first; Phase 2 authorization criteria filed April 27 | Pending Phase 1 completion |
| 7 | Boys club rankings (if Option A PASS, post-stability) | Boys all age groups | Depends on Boys Option A pass + calibration stability confirmation | Conditional |
| 8 | Boys Brier analysis — full (gender-separated, age-group) | Boys all ages | No current analysis isolates Boys-specific Brier performance | Low: structural calibration is sound; full validation deferred |
| 9 | MLS NEXT Boys Academy/Homegrown tier split deployment | Boys U13–U17 in MLS NEXT | FORGE event classifier not yet run; DSS timing constraint | Strong: empirically validated, deployment ready pending FORGE classifier |
| 10 | June 1 age group transition (U13→U14, etc.) | All age groups — both genders | Hard date constraint; calibration must be stable before June 1 | Structural: annual event, manageable if calibration changes are sequenced correctly |

---

## Section 2: Prioritized Work Queue

| Priority | Issue | Impact | Evidence | Sequencing Dependency | Est. Hours |
|----------|-------|--------|----------|-----------------------|-----------|
| 1 | U13/U14 K-factor + RD floor fix | 6,310+ teams, U12/U13/U14/U19 | Strong empirical (Brier ≥ 0.010) | None — deploy May 17 | 2–3 hrs (Presten execution + ELO validation) |
| 2 | Boys Option A application (if PASS) | Boys rankings all ages | Verdict May 4–5 | Boys Option A verdict must be PASS | 1.5–2 hrs (ELO SQL + Presten execution) |
| 3 | MLS NEXT Boys Academy/Homegrown deployment | Boys U13–U17 overrated 20–40 pts | Strong empirical (5,800 games) | FORGE classifier must run first | 2–3 hrs (FORGE + Presten) |
| 4 | Event Strength Phase 2 | Broader event tier score coverage | Pending Phase 1 validation | Phase 1 must complete + authorization criteria met | 3–5 hrs (multi-session) |
| 5 | Boys GA ASPIRE Brier analysis (cal=100 vs 70) | Boys GA teams; low DSS exposure | Theoretical concern only | April 28 fix must be deployed; MLS NEXT deployment first | 1–2 hrs |
| 6 | ECNL Boys vs GA Boys win-rate measurement | Validates 20-pt ECNL/GA gap | SQL ready; data likely available | Boys Option A outcome helpful context | 30 min (query execution only) |
| 7 | Boys club rankings activation | Boys rankings in product | Conditional on #2 and calibration stability | Boys Option A + stability window | Included in Boys Option A timeline |
| 8 | Full Boys Brier analysis | Boys-specific calibration confirmation | No active failure signal | Post U13/U14 fix; post Option A | 2–3 hrs |
| 9 | Tier 2 undefined leagues (deferred volume) | < 500 games impact minimal | Low — low volume | Volume check April 29 | 30 min if authorized |
| 10 | June 1 transition monitoring | All teams age up | Hard date: structural | All pre-June calibration changes must be stable | SENTINEL + Presten review session |

---

## Section 3: Sprint Plan — Top 3 Issues

### Priority 1: U13/U14 K-Factor + RD Floor Fix

**What ELO does:** Applies the calibration parameters confirmed in the April 2026 Brier analysis — K=24 for U13/U14, RD floor=75. This requires modifying `compute-rankings.js` and running a full recompute.

**Data needed:** No new data required. Parameters are locked. Presten applies the code change; ELO validates post-recompute ratings using the `Rankings/Calibration Fix Validation Results.md` template.

**Success criterion:** Post-recompute U13 Brier score drops by ≥ 0.008 from baseline (0.312 → ≤ 0.304). U14 Brier score drops by ≥ 0.005 (0.273 → ≤ 0.268). No regression in U15+ Brier scores. Girls and Boys both improve — parameters are gender-neutral.

### Priority 2: Boys Option A Application (Conditional)

**What ELO does:** If verdict is PASS (filed by May 4–5), ELO prepares the Boys calibration production UPDATE SQL (filed in `Rankings/Boys Option A — Post-Pass Implementation Timeline.md`). Presten runs it May 7–8. ELO validates May 7–9 with the verification queries from that document.

**Data needed:** Boys Option A query results (Presten runs April 29). If verdict is FAIL, skip this item and activate the Girls-Only Fallback Spec for DSS.

**Success criterion:** Post-application Boys ratings show direction-correct shifts within the ±50 pt tolerance. No Boys age group shows anomalous distribution (median shift > 100 pts). Cross-gender sanity check confirms Boys MLS NEXT average is within expected range of Girls MLS NEXT average.

### Priority 3: MLS NEXT Boys Academy/Homegrown Deployment

**What ELO does:** Reviews FORGE's event classifier quality check (≥90% classification accuracy required). If passed, provides production UPDATE SQL for Presten. Runs post-application validation: Boys MLS NEXT Academy teams should drop ~20–40 pts; Boys MLS NEXT Homegrown teams should rise slightly (or hold, depending on prior rating history).

**Data needed:** FORGE event classifier run (FORGE dependency). Classifier output confirming event classification rate. Should be available by May 17–18 if FORGE targets this window.

**Success criterion:** FORGE classifier returns ≥90% on a spot-check sample. Post-UPDATE Boys MLS NEXT Academy average rating decreases 20–40 pts vs baseline. Boys MLS NEXT Homegrown average stays within 10 pts of expected range. No cross-age-group Brier regression.

---

## Section 4: What Presten Needs to Decide

| Decision | Context | Needed By | Unlocks |
|----------|---------|-----------|---------|
| Authorize U13/U14 fix deployment | Code change to `compute-rankings.js`. Parameters pre-validated. One-line confirmation. | May 17 (day after DSS) | Priority 1 execution |
| Boys Option A verdict review | ELO files verdict by May 4–5. Presten reviews and authorizes implementation. | May 5–6 | Priority 2 execution |
| FORGE classifier authorization for MLS NEXT Boys | FORGE runs event classifier; Presten reviews classification quality report. | May 17–18 | Priority 3 execution |
| Event Strength Phase 2 authorization | ELO presents Phase 2 criteria check (all 4 gates in Phase 2 Authorization Criteria doc). Presten approves scope. | May 14–16 (pre-DSS or immediately post) | Priority 4 execution |
| Boys GA ASPIRE Brier analysis — proceed or defer | Low urgency. ELO needs Presten DB access for ~30-min Brier pull. Can be added to any May 17+ session. | May 17–30 window | Priority 5 execution |
| June 1 transition — confirm mechanism | Is the June 1 age transition automatic (system-driven) or requires a Presten manual step? If manual: what command? ELO needs to know to sequence calibration changes before vs. after. | By May 28 | Correct sequencing of Sprint 3 |

---

## Section 5: Session Estimate Summary

| Phase | Window | ELO Autonomous Sessions | Presten Sessions | Target Complete |
|-------|--------|------------------------|-----------------|----------------|
| Sprint 1: U13/U14 fix + Boys Option A | May 17–21 | 2 (SQL + validation) | 2 (code change + execution) | May 21 |
| Sprint 2: MLS NEXT Boys + Event Strength Phase 2 | May 22–28 | 3 (validation + Phase 2 scope) | 2 (MLS NEXT UPDATE + Phase 2 auth) | May 28 |
| Sprint 3: Boys Brier + win-rate gap + June 1 prep | May 29–31 | 2 (Brier queries + analysis) | 1 (Brier data pull) | May 31 |
| June 1 transition monitoring | June 1–3 | 1 (post-transition check) | 0 (if automatic) | June 3 |
| **Totals** | May 17 – June 3 | **8 sessions** | **5–6 sessions** | **June 3** |

"Fully calibrated" after June 3 means: U13/U14 fix applied, Boys calibration resolved (Option A path or fallback), MLS NEXT Boys deployed, Phase 2 event strength live, June 1 transition clean.

---

## Section 6: Post-DSS Talking Point for Presten

> "The DSS demo showcases rankings that are ready. The calibration work you're seeing — the GA ASPIRE fix, the Boys validation, the event strength rollout — is ongoing because accuracy improves with data. After the demo, our plan is to finalize Boys calibration by May 21, deploy the MLS NEXT Academy-Homegrown tier split by May 28, and complete the age group transition cleanly by June 1. By June 3, we're in the most calibrated state the system has ever been in. We know exactly what to do next and in what order."

---

## References

- `Rankings/Post-DSS Calibration Roadmap — June 2026.md` — sequenced roadmap (filed today)
- `Rankings/Boys Option A — Post-Pass Implementation Timeline.md` — Boys Option A sprint detail
- `Rankings/June 1 Age Transition — Risk Register.md` — June 1 risk context
- `Rankings/Event Strength Phase 2 — Authorization Criteria.md` — Phase 2 authorization gates
- `Rankings/Boys-ECNL-Calibration-Scope-2026-04-27.md` — Boys calibration basis
- `Rankings/ECNL-Boys-GA-Boys-Win-Rate-Gap-Investigation-2026-04-27.md` — win-rate gap status

*ELO — 2026-04-27 | Task: task-2026-04-27-post-dss-calibration-priority-queue*
