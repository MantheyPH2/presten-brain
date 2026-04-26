---
title: Event Strength Phase 2 — Prerequisites and Scope
type: prerequisites-spec
author: ELO
date: 2026-04-25
status: complete
related: "[[Event Strength Phase 2 — Scope and Plan]]", "[[Event Strength Phase 1 — Full Execution Package]]", "[[Event Strength Phase 1 — SENTINEL Authorization Criteria]]"
tags: [event-strength, phase-2, prerequisites, scope, evo-draw]
---

# Event Strength Phase 2 — Prerequisites and Scope

> **Purpose:** Define Phase 2 scope, DSS relevance, prerequisites, and timeline so SENTINEL can determine whether Phase 2 needs to be on the May 9 readiness checklist. Short answer: **Phase 2 is post-DSS.** Full design is in `Rankings/Event Strength Phase 2 — Scope and Plan.md`.

---

## Section 1: Phase 2 Scope Definition

**What Phase 2 changes relative to Phase 1:**

Phase 1 computes an `event_strength` table classifying events as High / Medium / Low based on the average rating of participating teams. Phase 1 is read-only — it does not change how rankings are computed.

Phase 2 applies `event_strength.strength_percentile` as a K-factor multiplier during Glicko-2 game processing. A win at a High-strength event produces a larger rating gain than a win at a Low-strength event:

```
K_adjusted = K_base × event_strength_factor
event_strength_factor = 0.70 + (strength_percentile / 100) × 0.60
→ ranges from 0.70 (weakest events) to 1.30 (strongest events)
→ = 1.00 at p50 (median events: no change from current baseline)
```

This scaling applies symmetrically to wins and losses.

**Gender scope:** Both Boys and Girls. Phase 2 reads `strength_percentile` from Phase 1; Boys and Girls are separate age group pools within Phase 1's computation.

**New data inputs required:** None. Phase 2 reads the `event_strength.strength_percentile` column already produced by Phase 1. No new source fields or cross-league win-rate updates needed.

**Reversibility:** Phase 2 is reversible by removing the K-factor lookup from `compute-rankings.js` and re-running the ranking computation. Full recompute required — not a single-record fix. Reversing Phase 2 has the same scope as applying it (~6–8h FORGE session).

**Estimated computation time:** 6–8 hours for implementation + validation (full breakdown in `Event Strength Phase 2 — Scope and Plan.md` Section 4).

**Schema impact:** None. Phase 1 DDL is sufficient for Phase 2. Phase 2 reads from `event_strength.strength_percentile` with no additional columns needed. This was the primary architectural question resolved before Phase 1 deployed.

---

## Section 2: DSS Relevance Assessment

**Phase 2 verdict: POST-DSS. Not required for May 16.**

Reasoning:

Phase 1 already provides the feature visible at DSS — the event strength card with High/Medium/Low classification, strength percentile, avg_team_rating, upset_rate. This is the demo feature. Phase 2's K-factor adjustment is a backend computation change that changes team ratings, not a new UI feature.

Checking against DSS Ranking Accuracy Claims:

| Authorized Claim | Phase 1 sufficient? | Phase 2 required? |
|-----------------|--------------------|--------------------|
| "Events are rated by competitive strength" | ✅ Phase 1 delivers this | No |
| "Rankings account for schedule strength" | ⚠️ Phase 1 gives event-level classification; Phase 2 adjusts the weight — Phase 1 is directionally sufficient for the claim, Phase 2 is the full implementation | Phase 2 makes the claim more precise, but Phase 1 supports it |
| "High-strength event performance is rewarded" | ⚠️ Phase 1 shows the strength context; Phase 2 applies it to ratings | Phase 2 makes this claim strictly accurate |

**Assessment:** Phase 2 strengthens the accuracy of claims Phase 1 already supports. No authorized DSS claim becomes false if Phase 2 is deferred — Phase 1 alone is sufficient for all demo claims. Deferring Phase 2 to post-DSS does not require removing any feature from the demo or weakening any accuracy claim below the authorized threshold.

**If Phase 2 were rushed to pre-DSS:** It introduces K-factor changes during the period when rankings must be stable for demo. Risk of unexpected rating shifts in the 2–3 weeks before May 16 is not acceptable. The stability concern outweighs any incremental DSS value.

---

## Section 3: Prerequisites

| # | Condition | Source of Confirmation | Pass Threshold | Notes |
|---|-----------|----------------------|----------------|-------|
| 1 | Phase 1 complete and validated | Phase 1 post-compute validation note (V1–V4 queries pass) | ELO clearance filed; all 4 validation queries pass | Phase 2 cannot run before Phase 1 data is stable — Phase 2 Brier validation requires correct Phase 1 `strength_percentile` values |
| 2 | Post-fix calibration stable (April 28 GA ASPIRE fix) | Post-Fix Calibration Stability Monitoring Spec — minimum 7-day clean-run window | ≥ 7 consecutive clean nightly pipeline runs post-April-28 | Phase 2 introduces a second source of K-factor variation. System must be stable before Phase 2 adds another variable |
| 3 | Calibration deployment complete (May 18–20) | Post-Calibration Monitoring Spec — May 2026 | May 18–20 session complete + ≥ 7 days stable post-deployment | K-factor/RD floor changes deploy May 18–20. Running Phase 2 concurrently with calibration changes makes Brier validation inconclusive — cannot isolate Phase 2 signal from calibration change signal |
| 4 | Post-calibration stability confirmed | Post-Calibration Monitoring Spec — May 2026 | ≥ 7-day monitoring window after May 18–20 deployment passes clean | Must complete before Phase 2 so the K_base Phase 2 scales is the validated post-calibration baseline |
| 5 | Boys Option A verdict filed | Boys Option A — Verdict Filing Document | PASS or FAIL declared (not pending) | If Boys Option A is still under investigation, Phase 2 should not run — Phase 2's Boys K-factor scaling is only meaningful over a validated Boys baseline. Boys PASS → validate Phase 2 over the cleared baseline. Boys FAIL → investigate Boys calibration first |
| 6 | Boys Brier analysis complete (if Boys Option A = FAIL or inconclusive) | Boys Brier Analysis Scope Spec — May 2026 | Boys GA ASPIRE cal value confirmed (retain 100 or revise to ~70) | Phase 2's K-factor scaling should run over a confirmed Boys K_base. If Boys K_base changes post-Brier analysis, Phase 2 parameters need re-validation |

**Hard blockers — Phase 2 must NOT run under any of these conditions:**
- Any active RED pipeline state (nightly pipeline failing)
- Within 7 days of calibration parameter changes (K-factor/RD floor deployment May 18–20)
- Phase 1 validation not cleared (any V1–V4 query failing)
- Boys Option A investigation still active (INCONCLUSIVE verdict pending ELO targeted check)

---

## Section 4: Timeline

**Phase 2 is post-DSS.** No timeline risk to May 9 gate or May 16 demo.

**Blocking constraint:** The K-factor/RD floor calibration deployment is May 18–20. Phase 2 modifies K-factor behavior. Running Phase 2 during or immediately after a K-factor change makes Brier validation inconclusive. Phase 2 must wait until the post-calibration monitoring window clears (7 days minimum after May 18–20 deployment = earliest clear by ~May 27).

**Second constraint:** Boys Brier analysis window is May 17–30. If Boys Brier analysis finds Boys GA ASPIRE needs a K-factor revision (Option B, cal=70), that revision must be applied before Phase 2 — Phase 2 scaling runs over the corrected K_base. If Boys Brier runs May 17–25 and any revision is applied May 26–30, Phase 2 cannot start until the revision is stable.

**Earliest Phase 2 start:** June 2, 2026.

**Recommended target window:** June 9–15, 2026 — after:
- Calibration changes applied and stable (May 18–20 + 7-day monitoring → ~May 27)
- Boys Brier analysis complete and any Boys K_base revision applied (target May 30)
- Phase 1 has run at least 4 full weekly pipeline cycles (Phase 1 stability confirmed)

**Risk if Phase 2 slips past June 15:** None for DSS (already demo'd). Low risk for summer calibration — Phase 2 is a precision improvement, not a known miscalibration fix. The June 1 age group transition is a higher-priority item in the June window. If Phase 2 competes with transition work, defer Phase 2 to early July.

---

## Section 5: SENTINEL Authorization Gate (Skeleton)

_Structure mirrors Phase 1 Authorization Criteria document. ELO fills when Phase 2 is ready to run. SENTINEL adds disposition block._

```
Phase 2 Authorization Request — ELO
Date: ___________

Prerequisite check:
  Phase 1 complete and validated: ___________  ✅/❌ [date cleared]
  Post-fix calibration stability (7-day window): ___________  ✅/❌ [date cleared]
  May 18–20 calibration deployment complete: ___________  ✅/❌ [date]
  Post-calibration stability (7-day window): ___________  ✅/❌ [date cleared]
  Boys Option A verdict filed: [PASS/FAIL] on ___________  ✅/❌
  Boys Brier analysis complete (if applicable): ___________  ✅/❌ [date cleared]

Phase 2 scope: Apply event_strength.strength_percentile as K-factor multiplier
  (K_adjusted = K_base × (0.70 + strength_percentile/100 × 0.60)) during
  Glicko-2 game processing in compute-rankings.js.

Expected Phase 2 effects:
  - Maximum per-game K adjustment: ±30% of K_base
  - Maximum total season impact: ~80–120 pts for teams playing exclusively High-strength events
  - Expected Brier improvement: ≥ 0 (Phase 2 should not degrade Brier vs. Phase 1)
  - Schema changes: None

Validation plan: Brier score comparison (Phase 1 vs. Phase 2 pipeline) on held-out game set.
  Pass: Phase 2 Brier ≤ Phase 1 Brier for U13G and U14G.
  Top 10 U13G under Phase 2 all from High-strength contexts (ECNL/GA/MLS NEXT).
  No team rating change > 200 pts when switching Phase 1 → Phase 2.

SENTINEL authorization request: Phase 2 authorized to run from [date].
ELO notes: ___________
```

```
SENTINEL disposition block:
Phase 2 authorized: YES / NO
If YES: authorized to run from ___________
If NO: hold reason: ___________
FORGE session scheduled: ___________
```

---

## References

- `Rankings/Event Strength Phase 2 — Scope and Plan.md` — full Phase 2 technical design (formula, schema, implementation estimate, worked examples)
- `Rankings/Event Strength Phase 1 — Full Execution Package.md` — Phase 1 DDL and validation spec
- `Rankings/Event Strength Phase 1 — SENTINEL Authorization Criteria.md` — gate format this document replicates
- `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md` — claims assessed in Section 2
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Phase 2 feature cell (marked post-DSS)
- `Rankings/Boys Brier Analysis Scope Spec — May 2026.md` — Boys K_base prerequisite (Section 3 item 6)

*ELO — 2026-04-25*
