---
title: Girls Full Calibration Run — Post-DSS Specification
type: calibration-run-spec
author: ELO
date: 2026-04-28
status: filed — pending SENTINEL pre-authorization
sprint_target: Sprint 1 (May 17–21)
sentinel_pre_authorization_requested: yes
related: "[[Post-DSS Calibration Sprint Plan]]", "[[Brier Score Completion]]", "[[Post-DSS Calibration Roadmap — June 2026]]", "[[Division Calibration]]"
tags: [calibration, girls, post-dss, brier, u13, u14, sprint, evo-draw]
---

# Girls Full Calibration Run — Post-DSS Specification

> **Filing note:** This spec is the pre-authorization document for the Girls calibration run in Sprint 1 (May 17–21). SENTINEL pre-authorizes the run based on this document. Once pre-authorized, ELO does not need a separate authorization request during the sprint. Presten executes the parameter changes; ELO validates. Filing this spec is the task — execution runs Sprint 1.

---

## Section 1: Calibration Run Scope

### What Is Changing

| Age Group | Parameter | Current Value | Proposed Value | Source | Expected Impact |
|-----------|-----------|--------------|----------------|--------|-----------------|
| U13 (Girls component) | K-factor | 32 | 24 | Brier score analysis (`Brier Score Completion.md`, Section 2) — U13 Brier=0.312; excessive K amplified GA ASPIRE overcalibration signal | Downward stabilization: slower rating volatility per game; ratings converge to true level rather than overshooting |
| U13 (Girls component) | RD floor | 50 | 75 | Calibration Validation 2026-04-22, Section 2 — RD floor too low causes premature confidence inflation; ratings locked in after insufficient game count | Higher RD floor keeps confidence intervals wider longer; predictions more honest during first-season exposure |
| U14 (Girls component) | K-factor | 32 | 28 | Brier score analysis — U14 Brier=0.273; K=32 producing overcorrection post-loss in high-exposure age group | Reduced rating volatility; post-loss drops less severe; convergence faster |
| U14 (Girls component) | RD floor | 50 | 65 | Calibration Validation 2026-04-22, Section 2 | Consistent with U13 fix; prevents premature certainty lock-in at higher U14 game volumes |
| U12 | RD floor | 50 | 60 | Brier score analysis — U12 Brier=0.241; RD floor is the sole miscalibration source (low severity) | Marginal: slightly wider confidence intervals for teams with limited U12 data; low magnitude change |
| U19 | K-factor | 32 | 24 | Brier score analysis — U19 Brier=0.238; K reduction is sole fix needed for this age group | Reduces over-reaction to single game results at U19 level where roster turnover creates noise |

**Girls-only application:** These parameter changes apply to Girls age groups only. Boys K-factor and RD floor changes are deferred until Boys Option A verdict is filed and Boys calibration pathway is confirmed. The `compute-rankings.js` implementation will apply these values to gender-separated calibration tables (Girls rows only). Presten confirms gender separation in the code before execution.

### What Is NOT Changing

| Parameter | Reason Held Fixed |
|-----------|------------------|
| Girls U9–U11 K-factor and RD floor | Brier=0.214–0.218 — well-calibrated; no fix warranted. Changing stable parameters risks introducing noise. |
| Girls U15–U17 K-factor and RD floor | Brier=0.211–0.219 — all acceptable; holding fixed. U15/U16/U17 are the strongest-calibrated age groups in the system. |
| GA ASPIRE event_tier calibration value (cal=100) | Applied April 28. Locked in. This spec does not re-open that decision. |
| Boys parameters (any age group) | Boys Option A result determines Boys pathway. Girls run does not touch Boys rows. |
| Event strength weights | Event Strength Phase 1 executes separately; these are event-tier calibration values, not per-team K/RD parameters. Do not intermix in same execution session. |
| League hierarchy calibration values | Tier 2 undefined leagues (USL Academy, EDP, Elite 64, NAL) are a separate SQL update. Not part of this run. |

---

## Section 2: Prerequisites

Required conditions before this run executes. Check each at Sprint 1 session open (May 17):

- [ ] **Event Strength Phase 1 completed.** Event tier values locked in. Required because Event Strength affects expected win probabilities; running K/RD parameter changes after event weights are set avoids a compounding recompute sequence.
- [ ] **ECNL migration implemented (FORGE, post-April-30 decision).** ECNL teams must carry correct league classification before Girls calibration run, or ECNL team ratings will be recalculated against misclassified opponents.
- [ ] **Girls data freshness: minimum 300 Girls games ingested post-April-28.** This ensures the post-April-28 GA ASPIRE recompute has propagated through enough games before parameter changes layer on top. Expected threshold met by May 7 based on typical pipeline volume.
- [ ] **Tier 2 undefined leagues calibration applied** (if authorized at April 29 volume check — see `Tier 2 Undefined Leagues — Calibration Recommendation.md`). If deferred post-DSS, apply Tier 2 values BEFORE the Girls K/RD run to avoid two closely-spaced recomputes.
- [ ] **Pre-run snapshot taken.** ELO baseline snapshot of Girls ratings before parameter changes — backup point for rollback if run fails.

**If prerequisites not met:** ELO defers the run to the next sprint window and notifies SENTINEL. Minimum fallback: Event Strength Phase 1 must be complete. ECNL migration and Tier 2 calibration delays extend the run date but do not cancel it.

---

## Section 3: Expected Brier Score Improvement

| Age Group | Pre-Fix Brier (Baseline) | Target Post-Run Brier | Minimum Acceptable Improvement | Basis |
|-----------|-------------------------|----------------------|---------------------------------|-------|
| U13 | 0.312 (blended, pre-April-28) | 0.210–0.220 | ≥ −0.050 improvement vs. pre-fix | Combined GA ASPIRE fix + K/RD parameter change; GA ASPIRE fix (April 28) contributes −0.015 to −0.025 Girls component; K/RD change contributes −0.070 to −0.080 |
| U14 | 0.273 (blended, pre-April-28) | 0.215–0.225 | ≥ −0.030 improvement vs. pre-fix | Combined; K/RD primary driver −0.040 to −0.050; GA ASPIRE contributes −0.010 to −0.020 Girls component |
| U12 | 0.241 (blended) | ~0.226 | ≥ −0.010 improvement vs. pre-fix | RD floor only; low-severity fix with narrow expected range |
| U19 | 0.238 (blended) | ~0.218 | ≥ −0.015 improvement vs. pre-fix | K-factor reduction only |

**Baseline note:** Pre-fix Brier values above are blended (Boys + Girls combined) from `Brier Score Completion.md`, Section 2. Gender-separated Brier has not yet been run. Post-run validation will run Girls-only Brier for the first time — actual Girls scores may differ from blended values. ELO accepts this uncertainty and will update the Brier Completion document with actual gender-separated values after the run.

**If post-April-28 Brier has been run before Sprint 1:** Update the "Pre-Fix Brier" column with actual post-April-28 Girls-only values before executing. The GA ASPIRE fix will already have improved U13G and U14G, so the K/RD parameter changes will be evaluated against the improved baseline, not the 0.312/0.273 pre-fix floor.

---

## Section 4: Run Execution Steps

Execute in strict sequence. Do not skip steps or reorder.

1. **Snapshot current Girls ratings.** Export Girls ratings for U12–U19 before any changes: timestamp, team_id, age_group, rating, rd. File in `reports/girls-calibration-pre-sprint1-snapshot-YYYY-MM-DD.txt`. This is the rollback point.

2. **Apply U13 Girls parameter changes.** In `compute-rankings.js` or relevant calibration config: set Girls U13 K=24, RD floor=75. Do not apply Boys U13 changes in the same commit.

3. **Apply U14 Girls parameter changes.** Set Girls U14 K=28, RD floor=65.

4. **Apply U12 Girls parameter change.** Set U12 RD floor=60. (If U12 is not gender-separated in the current config: apply to U12 blended — document in the run results that U12 RD floor applies to both genders, and Boys U12 Brier is not separately verified.)

5. **Apply U19 Girls parameter change.** Set Girls U19 K=24. (Same note on gender separation as U12 — document if blended.)

6. **Trigger Girls rankings recompute.** Partial recompute covering U12G, U13G, U14G, U19G. Full recompute is not required unless any parameter change affects shared calibration tables that also drive Boys calculations. ELO confirms scope with Presten before triggering.

7. **Extract post-run Brier scores.** Run Girls-only Brier computation for U12, U13, U14, U19. Compare against Section 3 targets.

8. **Spot-check top-20 Girls U13G and U16G.** Pull the post-run top-20 Girls U13G list and top-20 Girls U16G list. Confirm top teams are recognizable high-quality programs. Flag any unknown or clearly miscalibrated teams.

9. **File run results document.** File `Rankings/Girls Full Calibration Run — Results Report.md` with: parameter changes applied, actual post-run Brier values, spot-check findings, PASS / FAIL verdict, and any anomalies.

---

## Section 5: Pass/Fail Criteria

**RUN PASSES if all three conditions are met:**

1. Brier score improvement equals or exceeds minimum delta from Section 3 for at least U13 and U14 (primary fix targets). U12 and U19 improvements are secondary — passing those without U13/U14 does not constitute a pass.

2. Spot-check top-20 Girls U13G and U16G contain no more than 2 anomalous entries (unrecognizable programs in top 5, or known low-tier teams in top 10). ZERO tolerance for a clearly wrong team at rank 1–3.

3. No individual Girls team's rating shifts by more than 150 points unexpectedly (not explainable by the parameter change). Rating continuity spec reference: any individual shift > 150 pts triggers an investigation before declaring the run complete.

**RUN FAILS if any of the following:**

- Post-run Brier is worse than pre-run baseline for any target age group (regression)
- Spot-check reveals > 2 anomalous entries in top 20 of any target age group
- Any Girls team's rating shifts > 150 pts with no explanation traceable to the parameter change
- Gender separation was not maintained (Boys parameters inadvertently changed)

**If FAIL:** ELO files failure report to SENTINEL within 30 minutes. Reverts to pre-run snapshot (Step 1 file). Proposes revised parameters with root cause analysis. No re-run without SENTINEL authorization.

---

## Section 6: Post-Run Filing

After a successful run, ELO files (in sequence, same session):

1. **Girls Full Calibration Run — Results Report** at `Rankings/Girls Full Calibration Run — Results Report.md`
   - Actual post-run Brier values for U12, U13, U14, U19
   - Parameter changes confirmed applied
   - Spot-check observations
   - PASS verdict and rationale

2. **Update Brier Score Completion document** (`Rankings/Brier Score Completion.md`):
   - Section 4: Replace projections with actual post-run values
   - Remove "[Projection]" labels from U13 and U14 rows
   - Add actual Girls-only Brier values as new rows (U13G, U14G)

3. **Update Pre-May-9 Calibration State Dashboard** (`Rankings/Pre-May-9 Calibration State Dashboard.md`):
   - Girls full calibration row: → **MET**
   - Evidence statement: "Girls K/RD parameter changes applied [date]; post-run Brier U13=[value], U14=[value]; all targets met."

4. **Notify SENTINEL:** Queue item or briefing note: "Girls Full Calibration Run complete. Parameters applied, Brier targets met. Girls calibration item is MET for May 9 gate." No separate authorization request needed — pre-authorization via this spec is sufficient.

---

## References

- `Rankings/Brier Score Completion.md` — Brier baseline values and projection basis
- `Rankings/Post-DSS Calibration Sprint Plan.md` — sprint sequencing and Week 1 scope
- `Rankings/Post-DSS Calibration Roadmap — June 2026.md` — roadmap this sprint executes
- `Rankings/Calibration Validation 2026-04-22.md` — original K-factor and RD floor analysis
- `Rankings/Pre-May-9 Calibration State Dashboard.md` — dashboard this run feeds (due May 8)
- `task-2026-04-28-pre-may9-calibration-state-dashboard.md` — dashboard task context
- `task-2026-04-23-u13-u14-calibration-fix.md` — original K/RD fix task (parameter source)

*ELO — 2026-04-28 06:13*
