---
title: Event Strength Phase 2 — Evidence Gap Analysis
type: gap-analysis
author: ELO
date: 2026-04-27
status: complete
related: "[[Event Strength Phase 2 — Authorization Evidence Package]]", "[[Event Strength Phase 2 — Prerequisites and Scope]]"
tags: [event-strength, phase-2, gap-analysis, authorization, evo-draw]
---

# Event Strength Phase 2 — Evidence Gap Analysis

> **Purpose:** Map every Phase 2 authorization criterion against existing vault evidence. Identify what is satisfied, what is partial, and what is missing. Propose a closure plan so SENTINEL's authorization decision is not delayed by a missing evidence item that could have been prepared in advance.
>
> **Written:** 2026-04-27. Phase 2 earliest start: June 2, 2026. This document tracks evidence readiness between now and that date.

---

## Section 1: Criteria vs Evidence Matrix

Source criteria: `Rankings/Event Strength Phase 2 — Prerequisites and Scope.md` (Section 3) and `Rankings/Event Strength Phase 2 — Authorization Evidence Package.md` (Sections 1–4).

### Phase 1 Completion Gate (Evidence Package Section 1)

| Criterion | Status | Evidence Source | Notes |
|-----------|--------|-----------------|-------|
| Phase 1 Post-Authorization Runbook Steps 1–4 complete | **MISSING** | `Rankings/Event Strength Phase 1 — Post-Authorization Runbook.md` | Phase 1 not yet authorized or executed. SENTINEL authorization pending April 29 gate results. |
| Phase 1 event_strength rows inserted (≥ 50 per season) | **MISSING** | DB query result | Requires Phase 1 to run. Cannot be confirmed before execution. |
| Validation queries V1–V4 all PASS | **MISSING** | Post-Authorization Runbook Step 4 | Requires Phase 1 to run. |
| No rating impact from Phase 1 (Δ avg = 0) | **MISSING** | Runbook Steps 2 and 4 | Requires Phase 1 to run. |
| SENTINEL Phase 1 authorization on record | **MISSING** | Verdict Filing Document | Decision pending post-April 29 gate review by SENTINEL. |

**Summary:** All Phase 1 completion criteria are MISSING — but this is expected and not a planning gap. Phase 1 runs after SENTINEL authorization. ELO's job before June 2 is to ensure Phase 1 is authorized and executed. The runbook exists and is ready; no prep gap here.

---

### Data Coverage Prerequisites (Evidence Package Section 2)

| Criterion | Status | Evidence Source | Notes |
|-----------|--------|-----------------|-------|
| High-strength games in DB ≥ 500 | **MISSING** | DB query | Threshold defined. Query written in evidence package. Requires Phase 1 data to be inserted. |
| Medium-strength games in DB ≥ 1,000 | **MISSING** | DB query | Same dependency. |
| Low-strength games in DB ≥ 500 | **MISSING** | DB query | Same dependency. |
| strength_percentile NULL count = 0 | **MISSING** | DB query | Same dependency. |
| All active leagues have ≥ 1 event_strength entry | **MISSING** | Phase 1 runbook Step 1 coverage list | Partially satisfiable: Phase 1 tier coverage audit exists in `Rankings/Event Strength — League Tier Coverage Audit.md`. Cross-check against the full active league list is possible now. |
| No unmapped event_tiers in game history | **PARTIAL** | Phase 1 runbook Step 1 | `Rankings/Event Strength — League Tier Coverage Audit.md` documents tier mapping coverage. Unmapped tier count is documented there. This evidence exists now and pre-satisfies the criterion if the audit shows 0 unmapped tiers. |

**Autonomously closeable gap:** ELO can read the League Tier Coverage Audit now and verify whether the "all active leagues covered" and "no unmapped tiers" criteria are pre-satisfied from Phase 1 design. This does not require Presten DB time.

---

### Calibration Stability Gate (Evidence Package Section 3)

| Criterion | Status | Evidence Source | Notes |
|-----------|--------|-----------------|-------|
| Girls GA ASPIRE stability clearance filed in briefing | **MISSING** | ELO briefing (post-April-29) | Monitoring spec written. Clearance will be filed when 7-day clean run window completes. Earliest: May 5. |
| 7+ consecutive clean pipeline runs post-April-28 | **MISSING** | Monitoring spec Query 1 run log | April 28 has not occurred yet. Pipeline runs starting April 29. |
| May 18–20 calibration deployment complete | **MISSING** | Post-Calibration Monitoring Spec session record | Deployment not yet run. MLS NEXT Boys tier split deployment is confirmed and ready but must wait until May 18–20 window. |
| 7-day post-calibration monitoring passed | **MISSING** | Post-Calibration Monitoring Spec clearance | Dependency: May 18–20 deployment must complete first. Earliest clearance: ~May 27. |
| Boys avg rating change post-April-28 < 5 pts | **MISSING** | April 29 G4 gate result | April 29 gate has not run. Expected result: Boys avg change ≈ 0 pts (confirmed by Boys Calibration Status Summary). |
| No active calibration instability flags from SENTINEL | **MISSING** | Latest SENTINEL briefing | No flags currently exist in vault. This criterion is pre-satisfied in spirit but requires active confirmation at authorization time. |

**Pre-stageable evidence:** The monitoring spec (`Rankings/Post-Fix Calibration Stability Monitoring Spec.md`) and the post-calibration monitoring spec (`Rankings/Post-Calibration Monitoring Spec — May 2026.md`) are already written. Both monitoring frameworks are ready to run starting April 29. No additional prep needed.

---

### Phase 2 Scope Confirmation (Evidence Package Section 4)

| Item | Phase 2 Spec Says | Current Accuracy | Notes |
|------|-------------------|------------------|-------|
| K-factor formula | `K_adjusted = K_base × (0.70 + strength_percentile/100 × 0.60)` | **CONFIRMED** | Formula matches `Rankings/Event Strength Phase 2 — Prerequisites and Scope.md` Section 1. Unchanged. |
| Factor range | 0.70–1.30, = 1.00 at p50 | **CONFIRMED** | Confirmed in prerequisites spec. |
| Gender scope | Both Boys and Girls (conditional on Boys Option A) | **PARTIAL** | Boys scope confirmed pending Boys Option A verdict (April 29 window). If Boys Option A PASS: both genders. If FAIL: Girls only. Spec already documents this conditional. |
| New schema required | None — reads event_strength.strength_percentile | **CONFIRMED** | Phase 1 DDL sufficient. No additional schema prep needed. |
| Estimated session time | 6–8 hours | **CONFIRMED** | No changes to scope that would materially alter estimate. |
| Reversibility | Full recompute required; schema unchanged | **CONFIRMED** | No changes. |
| Boys Option A prerequisite | Boys verdict must be filed before Phase 2 | **PENDING** | Boys verdict expected April 29 (earliest). Not yet filed. |

**Assessment:** Scope is materially accurate. The only confirmation pending is Boys Option A verdict, which will resolve April 29.

---

## Section 2: Gap Inventory

### MISSING items requiring future execution (not plannable gaps)

| Gap | Evidence Needed | Can ELO Produce Autonomously? | Requires Presten? | Estimated Effort |
|-----|----------------|-------------------------------|-------------------|-----------------|
| Phase 1 completion evidence (all 5 items) | Phase 1 runbook execution results | No | Yes — Presten runs Phase 1 session | 6–8h FORGE + ELO session after SENTINEL authorizes |
| Data coverage counts (High/Medium/Low game counts, NULL check) | DB query results post-Phase-1 | No | Yes — Presten runs 4 queries in psql | ~10 min of Presten DB time, post-Phase-1 |
| Girls GA ASPIRE stability clearance | 7-day monitoring log (April 29–May 5) | Yes — ELO files clearance after 7 clean runs | No (monitoring is ELO autonomous) | ~5 min per monitoring check × 7 runs |
| 7-day post-April-28 pipeline clean run log | April 29–May 5 daily checks | Yes — ELO runs Query 1 and 2 from monitoring spec | No | Autonomous |
| May 18–20 deployment completion | Presten runs MLS NEXT UPDATE and recompute | No | Yes — confirmed as May 18–20 window | ~90 min Presten session (already planned) |
| Post-calibration 7-day stability | May 18–27 monitoring | Yes — ELO monitors autonomously | No | Autonomous |
| Boys avg rating change < 5 pts | April 29 G4 gate result | Yes — ELO files April 29 gate results | No | Already part of April 29 master script |

### PARTIAL items ELO can close now

| Gap | Action | Effort |
|-----|--------|--------|
| League tier coverage pre-check | Read `Rankings/Event Strength — League Tier Coverage Audit.md` and confirm 0 unmapped tiers | 10 min — ELO autonomous |
| Boys Option A verdict pre-staging | Boys Option A decision document is written (`Rankings/Boys Option A — Decision Document.md`). Verdict files April 29 when queries run. | No new prep needed |

---

## Section 3: Evidence Closure Plan

### Priority 1: No action needed (already complete)

- Phase 2 scope confirmed from vault — K-factor formula, range, schema, reversibility all verified
- Monitoring specs written and ready — both Girls stability and post-calibration monitoring are pre-staged
- Boys Option A documentation ready — decision document and execution package exist
- League tier coverage audit exists — ELO can pre-check the unmapped tier criterion

### Priority 2: Requires April 29 (earliest)

- Boys avg rating change confirmation (G4 gate result — filed same day)
- Boys Option A verdict filed (April 29 query window opens)
- Girls GA ASPIRE stability monitoring begins (first run April 29)

### Priority 3: Requires Phase 1 to be authorized and executed

- All Phase 1 completion gates
- All data coverage counts
- Earliest Phase 1 execution: after SENTINEL authorization post-April 29

### Priority 4: Requires May 18–20 deployment

- May 18–20 calibration deployment record
- Post-calibration stability clearance

### What blocks SENTINEL's Phase 2 authorization decision?

The authorization gate is sequential. SENTINEL cannot authorize Phase 2 until:
1. Phase 1 complete and validated → earliest after SENTINEL authorizes Phase 1 (post-April 29)
2. Post-April-28 stability clearance → earliest May 5
3. May 18–20 deployment complete → earliest May 21
4. Post-calibration stability confirmed → earliest May 27
5. Boys Option A verdict → April 29 (already in flight)
6. Boys Brier analysis (if Boys Option A is FAIL or inconclusive) → May 30

**Earliest all criteria met:** ~May 27 (if Boys Option A PASS and Phase 1 authorized immediately after April 29). If Boys Brier analysis required: ~June 1.

**Earliest Phase 2 authorization request to SENTINEL:** May 28–30.

**ELO-controlled timeline:** The only items ELO can accelerate are:
- Filing Girls stability clearance quickly (monitoring autonomously)
- Filing Boys Option A verdict same day (April 29)
- Pre-staging the Section 4 scope confirmation (already done above)

SENTINEL cannot authorize faster than the hard constraint: post-calibration 7-day window after May 18–20 deployment. The June 2 earliest start date in the prerequisites spec is the correct target.

---

## Section 4: Pre-Authorization Summary

> This section is a pre-draft for ELO to hand to SENTINEL when all criteria are met. Fill status fields at authorization time.

---

**Event Strength Phase 2 — Pre-Authorization Summary**
**Date of Summary:** ___________ (fill when criteria met)
**Prepared by:** ELO

| Criterion | Status at Authorization |
|-----------|------------------------|
| Phase 1 complete and validated | ✅ / ❌ [date cleared: ___] |
| Phase 1 data coverage thresholds met | ✅ / ❌ [confirmed ___] |
| Girls GA ASPIRE calibration stable (7-day window) | ✅ / ❌ [clearance filed: ___] |
| May 18–20 calibration deployment complete | ✅ / ❌ [deployed: ___] |
| Post-calibration stability cleared (7-day window) | ✅ / ❌ [clearance filed: ___] |
| Boys Option A verdict filed | ✅ PASS / ❌ FAIL [filed: April 29] |
| Boys Brier analysis complete (if applicable) | ✅ N/A (Option A PASS) / ✅ Complete [date] / ❌ Pending |
| Phase 2 scope accurate | ✅ Confirmed 2026-04-27 — no material changes |
| No active SENTINEL calibration instability flags | ✅ / ❌ |

**All criteria met:** YES / NO

**ELO Recommendation:** AUTHORIZE Phase 2 to run from [proposed date].

**Proposed execution:** FORGE Phase 2 implementation + ELO validation. 6–8 hours. Pre-session requirement: Presten confirms clean nightly pipeline run the day before FORGE begins.

**Boys scope:** BOTH genders (if Option A PASS) / GIRLS ONLY (if Option A FAIL — per Girls-Only Fallback Spec).

**Evidence locations:**
- Phase 1 completion: `Rankings/Event Strength Phase 1 — Post-Authorization Runbook.md`
- Calibration stability clearance: ELO briefing dated ___
- Post-calibration monitoring: `Rankings/Post-Calibration Monitoring Spec — May 2026.md`
- Boys Option A verdict: `Rankings/Boys Option A — Verdict Filing Document.md`
- Boys Brier result (if applicable): `Rankings/Boys-Brier-Pre-May17-Readiness-Check-2026-04-27.md`

---

## References

- `Rankings/Event Strength Phase 2 — Authorization Evidence Package.md` — structured evidence package (ELO fills; SENTINEL reviews)
- `Rankings/Event Strength Phase 2 — Prerequisites and Scope.md` — criteria definitions and timeline
- `Rankings/Event Strength Phase 1 — Post-Authorization Runbook.md` — Phase 1 completion gate source
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — Girls stability clearance criteria
- `Rankings/Post-Calibration Monitoring Spec — May 2026.md` — May 18–20 deployment monitoring
- `Rankings/Boys Option A — Verdict Filing Document.md` — Boys prerequisite gate

*ELO — 2026-04-27*
