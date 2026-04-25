---
title: Boys Calibration Status Summary — April 2026
type: status-summary
author: ELO
date: 2026-04-25
status: current
related: "[[Boys Calibration Gap Analysis — April 2026]]", "[[Boys Calibration Option A — Interpretation Guide]]"
tags: [boys, calibration, dss, status, rankings, evo-draw]
---

# Boys Calibration Status Summary — April 2026

> **Quick answer:** Boys calibration has never been separately validated by gender. Structural design is sound. U13B and U14B carry the highest DSS exposure risk. Boys club rankings at DSS are conditional on an Option A spot check passing by May 14. DSS risk: **Medium**.

---

## Section 1: Current State (as of April 25, 2026)

### U13B and U14B

**Status: Unknown — no gender-separated Brier analysis run.**

The existing Brier analysis (U13: 0.312, U14: 0.273) covers blended Boys+Girls data. Boys contribute roughly half those game counts (~13K U13B games, ~14K U14B games). Whether Boys are dragging the Brier score up, down, or neither is unknown. The proposed K-factor and RD floor changes (U13: K→24, RD floor→75; U14: K→24, RD floor→75) are gender-neutral — if they improve Girls calibration, they improve Boys by the same mechanism. No Boys-specific miscalibration has been identified, but none has been ruled out either. A Boys top-10 spot check (Option A, ~20 minutes) is required before DSS to confirm the rankings "look right" for the age groups most likely to be shown at the demo.

### U15B and U16B

**Status: Unknown — not yet assessed.**

Not included in any existing Brier analysis. Not planned for DSS. Lower immediate risk because no external benchmark comparison is planned for these age groups. MLS NEXT Boys calibration for these age groups (U15–U17 are the peak MLS NEXT enrollment years) is empirically validated from cross-league win-rate data — the +160 Homegrown / +135 Academy split is confirmed by ~5,800 games. The calibration design for U15B/U16B is the strongest of any Boys age group. Full Brier validation is deferred to June 2026.

### U17B–U19B

**Status: Unknown — K-factor change applies but not validated.**

U19B has a proposed K-factor change. The change is structurally justified but not Brier-validated for Boys specifically. Lower visibility at DSS — U17–U19 teams are not the primary demo cohort. MLS NEXT Academy/Homegrown split for U17B is validated from the same 5,800-game dataset. Full validation deferred to June 2026.

### GA ASPIRE Boys

**Status: Decided — Option A (cal=100). No net impact from April 28.**

Boys GA ASPIRE calibration is set to 100, the same as Boys GA. Decision rationale: Girls GA was being overcalibrated at 140 because GA ASPIRE is a sub-tier of GA. Boys GA is already at 100 (not 140), so the proportional error for Boys is smaller. The April 28 UPDATE reclassifies Boys GA ASPIRE events from `ga` to `ga_aspire` in the events table, but Phase 8 of `compute-rankings.js` uses cal=100 for both values — net rating impact on Boys is zero. A full Boys GA ASPIRE Brier analysis (testing whether cal=70 would be better) is deferred to May 17–30, before the June 1 age group transition.

### MLS NEXT Boys

**Status: Calibration validated. Deployment deferred to May 18–20.**

The MLS NEXT tier split (Homegrown 160 / Academy 135 / Unclassified 147) is empirically validated by two independent methods from ~5,800 games. Not deploying on April 28 — the FORGE event classifier has not been run, and adding 60–80 minutes to the April 28 session exceeds the ≤30-minute threshold for same-session additions. The May 18–20 calibration window is the designated deployment. Academy-heavy Boys U15–U17 teams are currently over-rated by ~20–40 points; this corrects post-May 18. Boys U13B/U14B MLS NEXT coverage is lower than U15–U17; calibration is valid but applied to a thinner sample.

---

## Section 2: Open Decisions

| Decision | Options | Current Lean | Decision Authority | Must Decide By |
|----------|---------|-------------|-------------------|----------------|
| Boys club rankings at DSS | Include (Option A spot check passes) / Exclude (fails) | Conditional include | SENTINEL → Presten | May 9 |
| Boys GA ASPIRE calibration (Option A vs. ~70) | Retain 100 / Reduce to ~70 | Retain 100 until Brier evidence | ELO → Presten | May 30 (pre-June 1 transition) |
| Boys Brier analysis window | May 17–30 (post-DSS) / June window | May 17–30 | ELO | Start May 17 |
| MLS NEXT Academy/Homegrown deployment | Deploy May 18–20 / delay further | Deploy May 18–20 | SENTINEL → Presten | May 17 (pre-flight check) |

---

## Section 3: What Needs to Happen

1. **April 28:** GA ASPIRE UPDATE runs (gender-agnostic). Boys GA ASPIRE existence check runs and result is logged. No Boys rating impact expected (cal=100 → 100). Owner: Presten (execution), ELO (log result in April 29 shift analysis).

2. **April 29:** ELO confirms Boys GA ASPIRE avg rating change ≤ 15 pts in the post-fix shift analysis. If Boys moved > 15 pts, escalate to SENTINEL before any further implementation sessions. Owner: ELO.

3. **By May 9:** Boys club rankings go/no-go. ELO runs the Option A spot check: three queries from `Boys Calibration Gap Analysis — April 2026` §3 (Top-10 U13B/U14B spot check, Boys/Girls distribution comparison, Boys GA + MLS NEXT game volume check). Requires ~20 minutes of Presten DB access — can be combined with the DSS Data Readiness Validation session. Pass: Boys rankings appear at DSS if a director asks. Fail: Boys rankings are excluded from the demo. Owner: ELO (queries), Presten (DB execution). Dependency: April 28 fix must be complete.

4. **May 17–30:** Boys GA ASPIRE Brier analysis. ELO runs the analysis from `Boys Brier Analysis Scope Spec — May 2026`. Requires ~30 minutes of Presten DB time to pull cross-tier game data. Pass threshold: Brier delta < 0.003 → retain cal=100. Fail threshold: delta > 0.008 → revise to cal=70. Owner: ELO. Dependency: April 28 fix must be complete so Boys GA ASPIRE events are correctly tagged.

5. **May 18–20:** MLS NEXT Boys tier split deployment. FORGE runs event classifier (Step A pre-flight). If ≥90% classification quality confirmed, Presten applies the UPDATE. ~60–80 minutes total session time. Owner: FORGE (classifier), Presten (UPDATE), ELO (post-split validation queries). Dependency: FORGE classifier quality check must run before May 18.

---

## Section 4: DSS Risk Assessment

**DSS risk for Boys rankings: Medium.**

The key gate is the Option A spot check (by May 9). If U13B and U14B top-10 look credible and the Boys/Girls distribution is within the 75-point divergence threshold, Boys rankings are safe to show at DSS. The structural calibration design for Boys is sound — MLS NEXT values are empirically validated, GA ASPIRE impact is neutral on April 28, and the K-factor/RD floor changes are gender-neutral improvements. The risk is unknown, not known-bad: no Boys Brier analysis has been run, so a systematic Boys-specific miscalibration could exist but has not been detected. If the May 9 spot check reveals a clearly wrong U13B top-10 (unrecognizable teams, suspicious ratings > 1,700, merge artifacts), Boys rankings are excluded from the demo and risk drops to zero. The probability of passing the spot check is High based on the sound theoretical design, but cannot be quantified without running the queries.

**What would change the probability:** Running the Option A queries before May 9. If Presten can execute the three spot-check queries in the May pre-DSS data readiness session, Boys calibration uncertainty collapses from "unknown" to either "confirmed safe" or "confirmed problem."

---

## References

- [[Boys Calibration Gap Analysis — April 2026]] — three validation queries and full gap analysis
- [[Boys Calibration Option A — Interpretation Guide]] — query interpretation thresholds and go/no-go decision rules
- [[Boys Brier Analysis Scope Spec — May 2026]] — post-DSS Boys GA ASPIRE Brier analysis scope
- [[Boys GA ASPIRE Calibration Decision — April 2026]] — Option A decision rationale (cal=100)
- [[MLS NEXT Boys Tier Split — April 28 Decision]] — DEFERRED to May 18–20; calibration validated
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — overall DSS feature status
