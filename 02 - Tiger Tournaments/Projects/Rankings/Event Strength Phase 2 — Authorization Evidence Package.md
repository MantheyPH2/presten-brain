---
title: Event Strength Phase 2 — Authorization Evidence Package
type: authorization-evidence
author: ELO
date: 2026-04-26
status: awaiting-phase1-completion
related: "[[Event Strength Phase 2 — Prerequisites and Scope]]", "[[Event Strength Phase 1 — Post-Authorization Runbook]]", "[[Post-Fix Calibration Stability Monitoring Spec]]"
tags: [event-strength, phase-2, authorization, evidence, evo-draw]
---

# Event Strength Phase 2 — Authorization Evidence Package

> **Purpose:** Structured evidence package for ELO to fill before requesting Phase 2 authorization from SENTINEL. SENTINEL reviews this document — not narrative briefing text — to authorize Phase 2. ELO fills Sections 1–5 when prerequisites are met; SENTINEL fills the disposition block.
>
> **Phase 2 earliest start:** June 2, 2026. This document is not expected to be fully filled before that date.

---

## Section 1: Phase 1 Completion Gate

Phase 2 cannot start until Phase 1 is fully validated. ELO confirms each row before proceeding.

| Gate | Required | Status | Evidence |
|------|----------|--------|----------|
| Phase 1 Post-Authorization Runbook Steps 1–4 complete | All 4 steps ✅ | ⬜ PENDING | Path: `Rankings/Event Strength Phase 1 — Post-Authorization Runbook.md` |
| Phase 1 event_strength rows inserted | ≥ 50 rows per season (2025-2026) | ⬜ PENDING | SQL: `SELECT COUNT(*) FROM event_strength WHERE season = '2025-2026'` |
| Phase 1 validation queries V1–V4 all PASS | All 4 ✅ | ⬜ PENDING | Filed in Post-Authorization Runbook Step 4 |
| No rating impact from Phase 1 (pre/post comparison) | Δ avg rating = 0 across all age groups | ⬜ PENDING | Pre/post table from Runbook Steps 2 and 4 |
| SENTINEL Phase 1 authorization on record | SENTINEL signed Verdict Filing Document | ⬜ PENDING | Date of authorization: ___________ |

**Phase 2 proceeds only if all 5 rows are ✅.** Any ❌ holds Phase 2 regardless of all other prerequisites.

---

## Section 2: Data Coverage Prerequisites

Phase 2 applies the `event_strength.strength_percentile` K-factor multiplier to game processing. The data coverage checks confirm that `strength_percentile` values are populated and usable before Phase 2 reads them.

| Prerequisite | Required Threshold | Current Status | Evidence |
|--------------|--------------------|----------------|----------|
| High-strength tier games in DB (2025-2026) | ≥ 500 games with `strength_class = 'High'` | ⬜ PENDING | `SELECT COUNT(*) FROM event_strength WHERE strength_class = 'High' AND season = '2025-2026'` |
| Medium-strength tier games in DB | ≥ 1,000 games with `strength_class = 'Medium'` | ⬜ PENDING | Same query, class = 'Medium' |
| Low-strength tier games in DB | ≥ 500 games with `strength_class = 'Low'` | ⬜ PENDING | Same query, class = 'Low' |
| `strength_percentile` populated for all classified events | NULL count = 0 | ⬜ PENDING | `SELECT COUNT(*) FROM event_strength WHERE strength_percentile IS NULL AND season = '2025-2026'` |
| All active leagues have at least one event with a strength_class | 100% of event_tiers in Phase 1 tier mapping have ≥ 1 entry | ⬜ PENDING | Cross-check Phase 1 runbook Step 1 tier coverage list |
| No major league missing event_strength value | 0 unmapped event_tiers in game history | ⬜ PENDING | Query from Phase 1 Runbook Step 1 |

ELO note: threshold counts above are minimums for a meaningful K-factor distribution. If High-strength count < 500, the percentile spread at the top of the scale compresses — Phase 2 would apply K_adjusted values near 1.30 to very few games, reducing its precision benefit.

---

## Section 3: Calibration Stability Gate

Phase 2 introduces a second source of K-factor variation into the computation. Running it before the underlying rating baseline is stable makes Brier validation inconclusive — ELO cannot isolate Phase 2 signal from pre-existing drift.

| Gate | Required | Status | Evidence |
|------|----------|--------|----------|
| Girls GA ASPIRE calibration stable per Stability Monitoring Spec | Stability clearance filed in an ELO briefing | ⬜ PENDING | Briefing date of clearance: ___________ |
| 7+ consecutive clean pipeline runs post-April-28 | ≥ 7 runs with volatility count below threshold | ⬜ PENDING | From monitoring spec Query 1 run log |
| May 18–20 calibration deployment complete | K-factor/RD floor changes applied | ⬜ PENDING | Post-Calibration Monitoring Spec session record |
| 7-day post-calibration monitoring window passed | ≥ 7 clean runs after May 18–20 session | ⬜ PENDING | Post-Calibration Monitoring Spec clearance filed |
| Boys avg rating post-April-28 change < 5 pts | Boys unaffected by GA ASPIRE fix | ⬜ PENDING | From April 29 G4 gate result |
| No active calibration instability flags from SENTINEL | 0 open flags | ⬜ PENDING | Check latest SENTINEL briefing before filling Phase 2 request |

**Hard block:** Phase 2 must NOT run within 7 days of any K-factor or RD floor change. The May 18–20 calibration session means Phase 2 earliest clear date is ~May 27. Phase 2 must also wait for Boys Brier analysis (May 17–30) if Boys GA ASPIRE cal value changes — Phase 2's K_base must be the post-Brier confirmed value.

---

## Section 4: Phase 2 Scope Confirmation

ELO confirms Phase 2 scope is still current before authorization. If scope has changed since `Event Strength Phase 2 — Prerequisites and Scope.md` was written, ELO documents the delta here.

| Item | Phase 2 Spec Says | Still Accurate? | If Changed: Notes |
|------|-------------------|-----------------|-------------------|
| K-factor formula | `K_adjusted = K_base × (0.70 + strength_percentile/100 × 0.60)` | ⬜ Confirm at fill time | |
| Factor range | 0.70 (weakest) to 1.30 (strongest), = 1.00 at p50 | ⬜ Confirm at fill time | |
| Gender scope | Both Boys and Girls | ⬜ Confirm at fill time | Conditional: Boys scope depends on Boys Option A verdict — see Prerequisite 5 |
| New schema required | None — reads `event_strength.strength_percentile` | ⬜ Confirm at fill time | |
| Estimated session time | 6–8 hours (FORGE implementation + ELO validation) | ⬜ Confirm at fill time | |
| Reversibility | Full recompute required (~6–8h); schema unchanged | ⬜ Confirm at fill time | |
| Boys Option A prerequisite | Boys verdict (PASS or FAIL) must be filed before Phase 2 | ⬜ Confirm at fill time | If Boys FAIL, Phase 2 runs Girls only; adjust scope accordingly |

SENTINEL will not authorize Phase 2 against stale scope. If any row in this table shows a material change, ELO updates the Phase 2 Scope and Plan document before submitting the authorization request.

---

## Section 5: Authorization Request

ELO completes this section when Sections 1–4 are fully filled and all gates are ✅.

```
Phase 2 Authorization Request
Date: ___________
From: ELO
To: SENTINEL

Phase 1 Completion: [CONFIRMED / INCOMPLETE — see Section 1]
Data Coverage: [PASS / FAIL — see Section 2]
Calibration Stability: [PASS / FAIL — see Section 3]
Scope Accurate: [YES / CHANGED — see Section 4]
Boys Option A Verdict: [PASS / FAIL / PENDING]

All prerequisites met: YES / NO

If YES:
ELO requests Phase 2 authorization to proceed on [proposed date].
Estimated session time: 6–8 hours.
Pre-session requirement: Presten confirms clean nightly pipeline run on [date − 1] before FORGE begins Phase 2 implementation.
Boys scope: [BOTH genders / GIRLS ONLY — per Boys Option A verdict]

If NO:
Held pending: [list outstanding items].
Expected resolution: [date].

ELO notes (optional):
```

---

## SENTINEL Disposition

> SENTINEL fills — do not pre-fill.

```
Phase 2 authorization package reviewed: [ ]
Evidence in Sections 1–4 reviewed: [ ]
Phase 2 authorized: YES / NO
If YES: authorized to run from ___________
If NO: hold reason: ___________
Boys scope authorized: BOTH / GIRLS ONLY / HOLD
FORGE session to be scheduled by: ___________
SENTINEL sign-off: ___________
Review date: ___________
```

---

## Why This Matters

Phase 2 is a higher-stakes execution than Phase 1 — it writes K-factor adjustments into the live ranking computation, not just a classification table. SENTINEL authorizing Phase 2 without a structured evidence package means authorizing on narrative alone. This document converts Phase 2 authorization into a checkpoint review: all prerequisites logged, evidence cited, ELO's recommendation stated. If any prerequisite fails, the hold reason is explicit and traceable. When Phase 2 is eventually challenged ("why did ratings shift in June?"), this document is the audit record.

---

## References

- `Rankings/Event Strength Phase 2 — Prerequisites and Scope.md` — prerequisite definitions and timeline
- `Rankings/Event Strength Phase 2 — Scope and Plan.md` — full Phase 2 technical design
- `Rankings/Event Strength Phase 1 — Post-Authorization Runbook.md` — Phase 1 completion evidence source
- `Rankings/April 29 Decision Tree — Verdict Filing Document.md` — Phase 1 authorization record
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — stability clearance criteria
- `Rankings/Boys Option A — Decision Document.md` — Boys prerequisite gate

*ELO — 2026-04-26*
