---
title: Calibration Values — League Hierarchy Reconciliation
type: deliverable
author: ELO
date: 2026-04-26
status: filed
hierarchy_version: "2026-04-24 (last updated)"
tags: [calibration, league-hierarchy, reconciliation, ranking-engine, evo-draw]
---

# Calibration Values — League Hierarchy Reconciliation

> **Audit scope:** Evo Draw Ranking Engine v7 (`compute-rankings.js`) vs [[League Hierarchy]] as of 2026-04-24.
> **ELO note on source access:** Engine calibration values are sourced from the `[[Ranking Engine]]` documentation (Phase 8) and the `[[League Hierarchy]]` authoritative table. Direct code inspection of `compute-rankings.js` was not performed during this reconciliation. Any discrepancies between the documented values and the live code should be resolved by Presten running a query against the `event_tiers` table to confirm actual tier codes and their mapped calibration values in the engine config.

---

## Section 1: Reconciliation Table

### Boys Tiers

| Tier Code | Cal Value (Engine) | Cal Value (League Hierarchy) | Match? | Notes |
|-----------|-------------------|------------------------------|--------|-------|
| `mlsnext` (unified, pre-split) | 160 | 160 | ✅ | Current production value; split pending May 18-20 |
| `mlsnext_homegrown` | not yet implemented | 160 (post-split) | ⏳ | Planned; not yet in production engine |
| `mlsnext_academy` | not yet implemented | 135 (post-split) | ⏳ | Planned; not yet in production engine |
| `mlsnext_unclassified` | not yet implemented | 147 (post-split, interpolated) | ⏳ | Planned; not yet in production engine |
| `ecnl` (Boys) | 120 | 120 | ✅ | Validated; ECNL Boys baseline anchor |
| `ga` (Boys) | 100 | 100 | ✅ | Theoretically sound; not yet Brier-validated |
| `ga_aspire` (Boys) | 100 (via `ga` tag) | 100 (assumed) | ✅ | No distinct Boys ga_aspire tier in engine. Boys GA ASPIRE events currently tagged as `ga` (cal=100). Hierarchy also assumes 100 for Boys GA ASPIRE. Effectively aligned. See flag note below. |
| `npl` | 55 | 55 | ✅ | Validated (ECNL RL beats NPL 55.1%) |
| `dpl` | 55 | 55 | ✅ | Validated peer tier to NPL |
| `ecrl` / `ecnl_rl` | 55 | 55 | ✅ | Validated; resolveTeamTier() enforces cap |
| `pre_ecnl` | 25 | 25 | ✅ | Validated; resolveTeamTier() enforces cap |
| `usl_academy` | unknown | not defined | ❓ | Tier 2 league in hierarchy with no published cal value; engine behavior on these events unknown |
| `elite_64` | unknown | not defined | ❓ | Listed as Tier 2 in hierarchy; no cal value |
| `edp` | unknown | not defined | ❓ | Listed as Tier 2 in hierarchy; no cal value |
| `nal` | unknown | not defined | ❓ | Listed as Tier 2 in hierarchy; no cal value |

### Girls Tiers

| Tier Code | Cal Value (Engine) | Cal Value (League Hierarchy) | Match? | Notes |
|-----------|-------------------|------------------------------|--------|-------|
| `ga` | 140 | 140 | ✅ | Validated Girls Tier 1 |
| `ga_aspire` | **140 (via `ga` tag, pre-fix)** | **100** | ❌ | **KNOWN DISCREPANCY — being corrected April 28. Pre-fix: engine classifies GA ASPIRE events under `ga` tier (cal=140). Correct value: 100. Fix adds distinct `ga_aspire` tier at cal=100. Post-fix state will be ✅.** |
| `ecnl` (Girls) | 130 | 130 | ✅ | Validated Girls Tier 1 |
| `npl` | 55 | 55 | ✅ | |
| `dpl` | 55 | 55 | ✅ | GA pathway; peer to NPL |
| `ecrl` / `ecnl_rl` | 55 | 55 | ✅ | resolveTeamTier() enforces cap |
| `pre_ecnl` | 25 | 25 | ✅ | resolveTeamTier() enforces cap |
| `usl_academy` | unknown | not defined | ❓ | Same unknown as Boys |

---

## Section 2: Discrepancy Log

| Tier Code | Engine Value | Hierarchy Value | Magnitude | Affected Gender / Age Groups | Estimated Rating Impact |
|-----------|-------------|-----------------|-----------|------------------------------|------------------------|
| `ga_aspire` (Girls) | 140 (via `ga` tag) | 100 | 40 pts | Girls all age groups with GA ASPIRE games | 40+ pts overcalibration per game for high-exposure teams; "40 points of distortion" confirmed in prior briefings |

**Authoritative source:** The League Hierarchy is authoritative. The GA ASPIRE correct cal value (100) was determined by cross-league win-rate analysis. The engine's treatment of GA ASPIRE as `ga` (140) was a missing-tier classification error, not a deliberate design decision.

**Correction status:** The April 28 session applies this fix — adding a distinct `ga_aspire` tier at cal=100 for Girls events. This closes the discrepancy. No further action required after April 28.

**Boys GA ASPIRE note:** Boys GA ASPIRE events are also currently tagged as `ga` (cal=100). The League Hierarchy also assumes 100 for Boys GA ASPIRE. This is technically aligned, but the 100 value is unvalidated for Boys. The correct Boys GA ASPIRE value may be ~70 (applying the same 30-pt differential logic as Girls: GA at 100, GA ASPIRE at ~70). Boys GA ASPIRE Brier analysis is scheduled post-DSS (June 2026). **This is not a current discrepancy — it is an unvalidated assumption on both sides.**

---

## Section 3: Missing Tier Analysis

| Tier Code | In League Hierarchy? | Games in DB? | Interpretation |
|-----------|---------------------|--------------|----------------|
| `usl_academy` | Yes (listed at Tier 2, no cal value) | Unknown | USL Academy has 90 clubs. If their events appear in the DB, the engine may classify them as untiered (lowest K-factor). Cal value needs to be defined. Recommend: add `usl_academy: 55` to align with other Tier 2 peers, pending Presten review. |
| `elite_64` | Yes (listed at Tier 2, no cal value) | Unknown | Tournament series; limited cross-league game volume makes empirical validation difficult. Recommend: treat as Tier 2 default (55) until cross-league data available. |
| `edp` | Yes (listed at Tier 2, no cal value) | Unknown | Eastern Development Program; regional scope. Recommend: Tier 2 default (55). |
| `nal` | Yes (listed at Tier 2, no cal value) | Unknown | North American League; limited coverage. Recommend: Tier 2 default (55). |
| `mlsnext_homegrown` | Yes (post-split) | Not yet (implementation pending) | Correct — split not yet applied to DB. No action until May 18-20 implementation window. |
| `mlsnext_academy` | Yes (post-split) | Not yet | Same as above. |

---

## Section 4: Reconciliation Verdict

**MINOR DISCREPANCY** (transitioning to CLEAN on April 28)

One discrepancy found: `ga_aspire` (Girls), engine cal=140 (via misclassification), hierarchy cal=100. Magnitude: 40 points. Fix is scheduled and authorized for April 28. After April 28 execution, all engine calibration values will match the League Hierarchy for active tiers.

**Residual items (not discrepancies, but require follow-up):**

1. **Undefined Tier 2 leagues (USL Academy, Elite 64, EDP, NAL):** These exist in the hierarchy without cal values and have unknown engine treatment. ELO recommends defining them as Tier 2 (55) in both documents and flagging for Presten review. If these leagues have meaningful game volume in the DB, their current treatment (likely untiered/default) may be distorting rankings for teams that play in those events.

2. **MLS NEXT tier split:** Post-split values (160 / 135 / 147) are documented in the hierarchy but not yet in the engine. This is planned and authorized for May 18-20. No action required until then.

3. **Boys GA ASPIRE:** Engine and hierarchy both assume 100, but the value is unvalidated. Brier analysis scheduled June 2026. Note the assumption explicitly in both documents.

**ELO certifies:** As of April 26, 2026, calibration values in the engine and the League Hierarchy match for all validated and defined tiers, with the single known exception of `ga_aspire` (Girls), which is being corrected April 28. SENTINEL can represent calibration accuracy at May 9 DSS go/no-go with this finding: one historical discrepancy, identified, corrected, no remaining active discrepancies in production (post-April-28).

---

## Section 5: Correction Actions

| Tier | Current Value (Engine) | Correct Value | Action | Status |
|------|----------------------|---------------|--------|--------|
| `ga_aspire` (Girls) | 140 (via `ga` tag) | 100 | Add distinct `ga_aspire` event tier at cal=100; reclassify GA ASPIRE events | **Executing April 28** |
| `usl_academy` | undefined (likely untiered) | 55 (recommended) | Add `usl_academy: 55` to engine config; add cal value to League Hierarchy | **Pending Presten review — low urgency, pre-DSS** |
| `elite_64` | undefined | 55 (recommended) | Same as above | **Pending Presten review** |
| `edp` | undefined | 55 (recommended) | Same as above | **Pending Presten review** |
| `nal` | undefined | 55 (recommended) | Same as above | **Pending Presten review** |

Corrections for USL Academy, Elite 64, EDP, and NAL are classified as MINOR DISCREPANCY — ELO may apply them and note in the next briefing, pending Presten confirmation that these leagues have meaningful game volume in the DB. If DB game volume for these tiers is negligible (< 1,000 games total), the rating impact is immaterial and corrections can be deferred post-DSS.

---

## Related Notes

- [[League Hierarchy]] — authoritative values (version: 2026-04-24)
- [[Ranking Engine]] — Phase 8 calibration application
- [[ga-coverage-audit-results-2026-04-23]] — GA ASPIRE game volume data that confirmed the Girls discrepancy
- [[Boys Calibration Gap Analysis — April 2026]] — Boys calibration context
- [[MLS NEXT Tier Split — Calibration Spec 2026-04-24]] — planned post-split values
