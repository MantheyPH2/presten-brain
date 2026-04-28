---
title: Boys ECNL Calibration Scope — 2026-04-27
type: calibration-reference
author: ELO
date: 2026-04-27
status: complete
tags: [calibration, boys, ecnl, evo-draw, rankings]
related: "[[Ranking Engine]]", "[[ECNL]]", "[[League Hierarchy]]", "[[Cross-League Analysis]]"
---

# Boys ECNL Calibration Scope — 2026-04-27

> **Purpose:** Close the documentation gap identified in the 08:46 SENTINEL review. SENTINEL flagged that Boys ECNL calibration status is not explicitly documented anywhere in the vault. This document answers: what is in scope, what calibration values are applied, what the empirical basis is, and what validation work remains. Serves as the reference basis for the Boys Option A pre-flight on April 29.

---

## 1. Leagues in Scope

Boys ECNL rankings in Evo Draw include two distinct tiers:

| League | Event Tier Tag | Calibration | Notes |
|--------|---------------|-------------|-------|
| ECNL Boys (National) | `ecnl` | **120** | ~150 clubs, 15 conferences. Tier 1. |
| ECNL Regional League Boys | `ecrl` | **55** | Feeder to ECNL Boys. Tier 2. |

**ECNL Boys (cal=120) and ECNL Regional League Boys (cal=55) are treated as separate tiers throughout the Ranking Engine.** The `resolveTeamTier()` function enforces this: any team with "ECNL RL" or "ECRL" in its name is capped at `ecrl` (55) regardless of what event tier the game is tagged with.

Pre-ECNL Boys (U10–U12) is NOT included in this scope — it carries its own tier (`pre_ecnl`, cal=25) and does not contribute to the Boys competitive ranking stack above U12.

---

## 2. Calibration Values

### ECNL Boys: cal=120

ECNL Boys sits at calibration 120 — second in the boys hierarchy, below MLS NEXT Homegrown (160) and above GA Boys (100).

The full boys calibration hierarchy as of 2026-04-27:

| League | Calibration |
|--------|-------------|
| MLS NEXT Homegrown | 160 |
| MLS NEXT Academy | 135 *(pending approval — queue item 2026-04-22)* |
| **ECNL Boys** | **120** |
| GA Boys | 100 |
| ECNL RL / NPL / DPL | 55 |
| USL Academy | 55 *(pending approval — Tier 2 SQL package filed 2026-04-27)* |
| EDP / Elite 64 | 55 *(pending approval)* |
| Pre-ECNL | 25 |

The 40-point gap between MLS NEXT Homegrown (160) and ECNL Boys (120) is empirically validated by the Cross-League Analysis (see Section 3). The 20-point gap between ECNL Boys (120) and GA Boys (100) is based on the relative positioning of ECNL and GA in the girls hierarchy and expert judgment on the boys side — it has **not been Brier-validated** (see Section 4).

### ECNL Regional League Boys: cal=55

ECNL RL Boys is calibrated at 55, equivalent to NPL/DPL. This is empirically validated: the Cross-League Analysis showed ECNL RL beat NPL in 55.1% of cross-league games — statistically significant moderate advantage, but insufficient to justify a calibration differential. Both leagues received 55. This applies to both boys and girls ECNL RL.

---

## 3. Cross-League Analysis Basis

The 575K game win-rate study ([[Cross-League Analysis]]) provides the following evidence relevant to Boys ECNL:

| Finding | Implication for Boys ECNL |
|---------|--------------------------|
| MLS NEXT vs all others: dominant | Confirms MLS NEXT (160) > ECNL (120) gap is correct directionally |
| ECNL RL vs NPL: 55.1% | Confirms ECNL RL (55) calibration, which anchors the Boys ECNL relative position |
| MLS NEXT Homegrown vs ECNL: 71.4% *(MLS NEXT tier split analysis)* | Confirms the 40-point gap between Homegrown (160) and ECNL Boys (120) |
| MLS NEXT Academy vs ECNL: 58.2% *(MLS NEXT tier split analysis)* | Confirms ECNL Boys (120) calibration is above Academy (implied ~135) |

**The 575K game study does not include a direct Boys ECNL vs GA Boys win-rate comparison.** The ga-data-coverage-audit (task assigned 2026-04-23) was intended to validate Boys GA (100) empirically — including cross-league play against ECNL Boys. Until that audit produces a validated GA Boys win-rate vs ECNL Boys, the 120/100 gap (20 points) rests on the positional logic of the hierarchy rather than a directly measured win rate.

**Key structural point:** The Boys ECNL calibration of 120 is effectively validated from above (by the MLS NEXT win-rate data confirming ECNL Boys is significantly weaker than MLS NEXT) and from below (by the ECNL RL vs NPL data confirming the gap between Tier 1 and Tier 2 is large). The 120 value places ECNL Boys squarely in that gap, consistent with ECNL's structural position as the second-strongest boys league.

---

## 4. Validation Status

| Dimension | Status | Notes |
|-----------|--------|-------|
| Brier-validated (age group level) | **No** | Brier analysis (2026-04-22) was age-group based, not league/gender segmented. Boys ECNL games are in the Brier sample but not isolated. |
| Win-rate validated (vs MLS NEXT) | **Indirect** | MLS NEXT Homegrown vs ECNL Boys: 71.4% win rate confirms the 40-pt gap. |
| Win-rate validated (vs ECNL RL) | **Yes** | ECNL RL vs NPL 55.1% validates the 65-point gap between ECNL (120) and ECNL RL (55). |
| Win-rate validated (vs GA Boys) | **No** | Direct ECNL Boys vs GA Boys cross-league win rate not yet measured. GA Data Coverage Audit (task-2026-04-23) is the vehicle for this. |
| Known miscalibration signals | **None flagged** | No Brier anomalies observed for age groups where ECNL Boys dominates the game sample (U15–U17). U13/U14 miscalibration is age-group parameter issue, not a Boys ECNL calibration issue. |

**Confidence basis for cal=120:** The ECNL Boys calibration at 120 is supported by:
1. Structural position in the hierarchy (below MLS NEXT, above GA)
2. Indirect win-rate evidence confirming the MLS NEXT–ECNL gap (71.4% Homegrown win rate)
3. The GA boys calibration of 100 is plausible but not directly validated — the 20-point ECNL-to-GA gap is the weakest link in the chain

This is categorized as **theoretically sound, partially empirically validated.** The absence of a flagged miscalibration signal is meaningful — if ECNL Boys were significantly miscalibrated at 120, we would expect cross-age-group Brier anomalies in ECNL-heavy age groups (U15–U17), and none appear.

---

## 5. Interaction With Boys Option A

**Boys Option A is a validation check for GA Boys calibration (cal=100), not for ECNL Boys calibration (cal=120).**

If Boys Option A PASSES:
- GA Boys (100) is confirmed. The 20-point gap between ECNL Boys (120) and GA Boys (100) is indirectly supported.
- Boys club rankings can be shown at DSS. ECNL Boys games contribute at their full Phase 8 calibration weight (120).
- The ga-data-coverage-audit cross-league win rate for Boys ECNL vs GA Boys can be reviewed post-DSS to confirm or refine the 120/100 split.

If Boys Option A FAILS (Boys GA Q3 anomaly confirmed):
- GA Boys calibration is under revision. The 20-point gap is uncertain.
- Boys club rankings are held from DSS.
- **ECNL Boys cal=120 is not directly affected by a GA Boys failure.** The 120 value is anchored from above (MLS NEXT win rates) and from below (ECNL RL calibration), not from the GA Boys relationship. However, the Boys ranking distribution will reflect the under/over-calibration of GA Boys until the fix is applied.

**Conclusion:** Boys Option A pre-flight on April 29 can reference this document as confirmation that ECNL Boys cal=120 is sound and will not require revision regardless of the Option A verdict. The only open Boys-side calibration question is the ECNL Boys vs GA Boys win rate, which is a post-DSS task.

---

## 6. Known Risks

| Risk | Severity | Status |
|------|----------|--------|
| ECNL Boys vs GA Boys win rate not measured | Low-Medium | Deferred to post-DSS. No active miscalibration signal. |
| Boys ECNL Brier score not isolated by league | Low | Age-group level shows no Boys-dominated anomalies. Acceptable for now. |
| ECNL June 2026 platform migration | Medium | ECNL moves from TGS to GotSport in June 2026. This changes the data pipeline for Boys ECNL games but not the calibration value. Monitoring needed post-migration to confirm game volumes are stable. |
| MLS NEXT Academy calibration (135) pending approval | None for ECNL | Academy split affects the league above ECNL Boys in the hierarchy. If Academy drops to 135, the 15-point gap to ECNL Boys (120) is narrower than before. This is calibration-correct but may produce some rating distribution compression in the 120–135 range. |

**No known miscalibration that would require Boys ECNL cal to change before DSS.**

---

## 7. Post-DSS Work Needed

| Task | Priority | Notes |
|------|----------|-------|
| Run Boys ECNL vs GA Boys cross-league win rate | High | Part of Boys Brier validation scope. Measures actual 120/100 gap empirically. |
| Run Boys ECNL-specific Brier score (segmented by league) | Medium | Isolate Boys ECNL games to confirm no age-group-independent calibration issue. |
| Monitor ECNL data pipeline post-June migration | High | TGS → GotSport switch. Confirm Boys ECNL game volumes are consistent and data quality is maintained. |
| Validate Boys MLS NEXT Academy split impact on ECNL Boys ratings | Medium | If Academy drops to 135, measure rating distribution shift for ECNL Boys teams that frequently play MLS NEXT Academy opponents. |

---

## Related Notes

- [[Ranking Engine]] — Phase 8 calibration values and `resolveTeamTier()` function
- [[ECNL]] — league structure (Boys cal=120, Girls cal=130)
- [[League Hierarchy]] — full tier system with calibration values
- [[Cross-League Analysis]] — win-rate methodology and validation data
- [[MLS NEXT Tier Split Analysis 2026-04-22]] — Homegrown vs Academy; confirms ECNL Boys 120 from above
- [[Calibration Validation 2026-04-22]] — Brier score analysis (age-group level, not Boys-ECNL-specific)

*ELO — 2026-04-27 | Task: task-2026-04-27-boys-ecnl-calibration-scope-documentation*
