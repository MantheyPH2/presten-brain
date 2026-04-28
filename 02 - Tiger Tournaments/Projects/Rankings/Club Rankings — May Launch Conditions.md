---
title: Club Rankings — May Launch Conditions
type: go-no-go-decision
author: ELO
date: 2026-04-28
status: filed — SENTINEL review requested
due: 2026-05-02
related: "[[Club Rankings — Implementation Specification]]", "[[Club Rankings — Girls-Only Fallback Spec]]", "[[DSS Feature Readiness Tracker — May 2026]]"
tags: [club-rankings, dss, go-no-go, girls, boys, sentinel, evo-draw]
---

# Club Rankings — May Launch Conditions

> **Filed by ELO — 2026-04-28 01:39**
> SENTINEL authorization requested per Section 6. Girls implementation is scheduled to begin May 2. Boys scope is deferred pending Boys Option A verdict (April 29).

---

## Section 1: Current Status Summary

| Item | Status |
|------|--------|
| Implementation spec | Filed ✅ — `Rankings/Club Rankings — Implementation Specification.md` (2026-04-25) |
| Reference club set | Confirmed ✅ — `Rankings/Club Rankings Reference Spot-Check Set.md`; Girls top-5 also confirmed in Girls-Only Fallback Spec |
| Girls-only SQL | Written ✅ — computation SQL in Implementation Plan, Girls gender-filter variant in Girls-Only Fallback Spec; NOT YET TESTED on staging |
| Last test run | Never run — implementation start is May 2 per the implementation timeline |
| Blocking issues | Two: (1) clubs/team_club_assignments/club_rankings tables do not yet exist — created in May 2 session; (2) Boys scope pending Boys Option A verdict April 29. Neither blocks Girls. |

**No design decisions remain open.** All specs, reference clubs, computation SQL, API, and frontend code artifacts are written. The outstanding work is execution, not design.

---

## Section 2: Launch Conditions — Girls Only

| Condition | Required | Current Status |
|-----------|----------|---------------|
| Reference clubs confirmed (Girls set) | YES | **MET** — Eclipse Select SC, PDA, PA Classics, FC Dallas Academy, NJ Wildcats confirmed as reference clubs with expected rank ranges |
| Girls club ranking SQL written and tested on staging | YES | **NOT MET** — SQL written and reviewed; not tested; first test run is May 2 session Phase 2 |
| Output covers ≥ 80% of Girls ECNL, MLS NEXT clubs | YES | **NOT MET** — cannot validate until SQL runs; pre-flight coverage query (Implementation Plan Step 2) will confirm before computation proceeds |
| No club ranking anomalies in top-20 (spot-checked) | YES | **NOT MET** — not testable until computation runs; validation queries V1–V3 and PA Classics check are ready |
| Rank bands integration confirmed for club view | YES | **NOT MET** — frontend not yet built; rank band rendering is in the ClubRankings.jsx spec but untested |
| SENTINEL authorization received | YES | **PENDING** — requested in Section 6 of this document |

**Girls Club Rankings launch: NOT READY**

What must happen before Girls launch is authorized:

| Step | Target Date | Owner |
|------|-------------|-------|
| Phase 1: Create tables and bootstrap clubs | May 2 | Presten + FORGE |
| Phase 2: Run computation script, confirm 50+ qualifying clubs | May 2–3 | Presten + FORGE |
| Phase 3: API endpoints live and tested | May 5–6 | FORGE |
| Phase 4: Frontend pages rendered, gender toggle working | May 7–8 | FORGE |
| Validation: PA Classics ≤ rank 30, V1–V3 pass | May 8–9 | ELO + Presten |
| SENTINEL authorization for production launch | May 9 | SENTINEL |

**No blocking risk from Girls calibration.** All five Girls age groups (U13G–U17G) have sufficient GA ASPIRE-corrected ratings post-April-28 fix for stable club aggregation. U17G is conditionally stable — the top clubs still qualify with 5+ age groups via U13–U16 coverage even if U17G is thin; this does not affect the demo.

---

## Section 3: Launch Conditions — Boys

| Condition | Required | Current Status |
|-----------|----------|---------------|
| Boys Option A verdict: PASS | YES | **PENDING** — April 29 spot check. Q1 = Boys/Girls rating divergence ≤ 100 pts; Q3 = Boys GA ASPIRE game count ≥ 500 |
| Boys reference clubs confirmed | YES | Confirmed ✅ in `Rankings/Club Rankings Reference Spot-Check Set.md` |
| Boys club ranking SQL tested | YES | **NOT TESTED** — same implementation session as Girls; Boys gate (Gate B) and Boys validation query (V4) added after Option A confirms |

**Boys Club Rankings: Deferred until Boys Option A confirmed.**

Earliest Boys launch: **May 10–11, 2026**, assuming:
- Boys Option A returns PASS on April 29
- SENTINEL authorizes Boys scope on April 29 or April 30
- Boys-specific implementation (Gate B, V4 validation) adds ~1.5 hours to the Girls session

If Option A fails by May 5 (the Girls-only fallback trigger date), Boys Club Rankings are cut from the DSS demo. Girls-only scope activates per `Rankings/Club Rankings — Girls-Only Fallback Spec.md`. The fallback narrative — "here are Girls club rankings nationally" — is appropriate for the DSS audience, which skews toward Girls program directors.

---

## Section 4: DSS Demo — Club Rankings Inclusion Decision

**ELO recommendation: CONDITIONAL**

Club rankings should be included in the May 16 DSS demo under the following conditions:

1. Girls implementation completes by May 9 go/no-go date (Phase 1–4 complete, V1–V3 pass)
2. PA Classics appears at rank ≤ 30 in Girls U13–U16 combined club view
3. SENTINEL authorizes Girls production launch by May 9

**Feature sequence for demo if included:**
1. Club leaderboard: Top 10 Girls clubs nationally (name, rank, rating, age groups covered)
2. Search for "PA Classics": click through to club detail page showing age-group breakdown
3. Rank band displayed on club detail — "Silver" or "Gold" — with brief narrative

This sequence takes approximately 90 seconds. It answers "which club is best in my area?" in a format club directors and tournament organizers will immediately understand.

**If Boys Option A passes:** Boys toggle can be shown on the club rankings page during the demo. ELO recommends demonstrating Girls first, then toggling to Boys to show the system covers both — without making any Boys-specific calibration claims beyond Claim B2 from the DSS Ranking Accuracy Claims spec.

**If Boys Option A fails:** Show Girls only. Do not explain the absence of Boys club rankings unless directly asked. If asked, say: "Boys club rankings are in our pipeline — we're finalizing calibration validation before we publish those."

**If Girls implementation is not complete by May 9:** Cut Club Rankings from the demo entirely. Do not attempt a partial or untested demo of this feature.

---

## Section 5: Earliest Launch Date

**Earliest Girls Club Rankings launch date: May 9, 2026**
- Prerequisite: Implementation starts May 2 as scheduled (10-hour session across May 2–8)
- Risk: Any single-day delay in May 2–8 pushes the May 9 validation date; implementation has no buffer

**Earliest Boys Club Rankings launch date: May 10–11, 2026 (conditional on Option A)**
- Prerequisite: Boys Option A PASS on April 29 and SENTINEL authorization by April 30

**Recommended launch sequence:**
1. Girls Club Rankings → production May 9 after SENTINEL authorization
2. Boys Club Rankings → production May 10–11 if Option A confirms
3. Combined club view (single leaderboard with M/F toggle) → already built into the Implementation Plan; activates automatically when both datasets are populated

---

## Section 6: What SENTINEL Should Authorize

**Authorization requests — in priority order:**

**Authorization 1 — Girls Implementation Session (REQUESTED NOW)**
Authorize Girls Club Rankings Phase 1 implementation session to begin May 2. This creates the three tables (`clubs`, `team_club_assignments`, `club_rankings`) and bootstraps the clubs data from existing teams table. No production data is modified — these are additive schema changes. Rollback is a three-table DROP with no downstream impact.

**Authorization 2 — Girls Club Rankings Production Launch (REQUESTED BY May 9)**
Authorize Girls Club Rankings for production display after Phase 4 completes and V1–V3 validation queries pass. Specific thresholds: ≥ 50 qualifying Girls clubs with 5+ age groups; PA Classics ≤ rank 30; no top-20 anomalies per spot-check.

**Authorization 3 — Boys Club Rankings (REQUEST AFTER April 29)**
Defer until Boys Option A results are filed. ELO will update this document on April 29 with the Option A verdict and a Boys authorization request if applicable.

---

## Definition of Done (ELO confirms)

- [x] Document filed at `Rankings/Club Rankings — May Launch Conditions.md`
- [x] All conditions in Section 2 have explicit MET / NOT MET status
- [x] DSS inclusion recommendation is explicit: CONDITIONAL
- [x] Earliest launch dates are specific dates, not relative windows
- [x] SENTINEL authorization is requested in Section 6
- [ ] ELO briefing May 2 references this document and states whether SENTINEL authorization has been received

---

## References

- `Rankings/Club Rankings — Implementation Specification.md` — full implementation with all code artifacts
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — fallback scope if Boys Option A fails
- `Rankings/Club Rankings Reference Spot-Check Set.md` — reference clubs for Girls and Boys validation
- `Rankings/Club Rankings — Boys-Conditional Implementation Timeline.md` — Boys scope timeline
- `Rankings/Club Rankings — Calibration Pre-Assessment.md` — pre-implementation calibration check
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — DSS readiness tracker (Club Rankings row)
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — Boys verdict condition

*ELO — 2026-04-28 01:39*
