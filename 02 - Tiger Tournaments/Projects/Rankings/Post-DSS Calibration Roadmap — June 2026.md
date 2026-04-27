---
title: Post-DSS Calibration Roadmap — June 2026
type: deployment-roadmap
author: ELO
date: 2026-04-26
status: ready — Boys row TBD pending Option A results
due: 2026-05-10
related:
  - "[[Boys Option A — Decision Document]]"
  - "[[June 1 Age Transition — Risk Register]]"
  - "[[Post-Fix Calibration Stability Monitoring Spec]]"
tags: [calibration, roadmap, post-dss, june-1, elo, evo-draw]
---

# Post-DSS Calibration Roadmap — June 2026

> **Purpose:** Sequenced plan for all pending calibration changes after DSS (May 16). Written now so May 17 starts with a clear deployment order, not a design meeting. The three changes below interact with each other and with the June 1 age transition — wrong ordering creates validation noise that cannot be untangled cleanly.
>
> **Boys row status:** TBD pending Boys Option A spot check results (April 29 – May 5). The roadmap provides Option A and Option B sequences. ELO will select and confirm the active sequence after filing the Boys Option A Decision Document.

---

## Section 1: Changes in Queue

| Change | Scope | Parameters | Code Change? | Validation Spec | Status |
|--------|-------|-----------|-------------|----------------|--------|
| U13/U14 calibration fix | 6,310+ teams across U12, U13, U14, U19 (both genders) | U12: K→20, RD floor→80; U13: K→24, RD floor→75; U14: K→24, RD floor→75; U19: K→12 | Yes — `compute-rankings.js` | `Rankings/calibration-fix-validation-results-2026-04-23.md` | Staged — deploy May 17 |
| Boys Option A calibration changes (if PASS) | Boys age groups with confirmed calibration gaps (TBD from spot check — likely U13B/U14B primarily) | TBD after Option A results — may include Boys GA ASPIRE reassessment (100 → ~70) if Brier analysis is accelerated | TBD — depends on which parameters change | `Rankings/Boys Option A — Decision Document.md` | TBD pending April 29 – May 5 spot check |
| Boys Option A — investigation scope (if FAIL) | Boys calibration investigation — scope TBD | N/A — no changes until investigation concludes | N/A | N/A — Boys Brier analysis is the investigation tool | TBD pending Option A results |
| MLS NEXT tier split deployment | Boys U13–U17 MLS NEXT teams: Homegrown / Academy / Unclassified split | Homegrown: cal=160; Academy: cal=135; Unclassified: cal=147 | Yes — FORGE event classifier + Presten UPDATE | `Rankings/MLS NEXT Tier Split — Calibration Application Spec.md` | Staged — deploy May 18–20 (FORGE classifier pre-flight required) |

**Change not yet in roadmap (Boys GA ASPIRE Brier analysis):** ELO has scope for this in `Rankings/Boys Brier Analysis Scope Spec — May 2026.md`. If run May 17–30, results feed into whether Boys GA ASPIRE cal=100 should be revised downward to ~70. This is not blocking DSS or June 1 — it is a calibration quality improvement. If Option A PASSES, ELO will incorporate Boys GA ASPIRE results into the Boys calibration section of this roadmap post-May-17.

---

## Section 2: June 1 Transition Risk Assessment

### What changes June 1

All teams shift from birth-year to school-year age group buckets. U12 → U13, U13 → U14, etc. The `age_group_advance()` function applies the school-year mapping table automatically — Presten does not run a manual migration step, but does run a sequencing check query to confirm age group advancement completes before firing `compute-rankings.js`. Teams with match_count ≥ 10 AND last game date within 60 days of June 1 carry their Glicko-2 parameters (rating + RD) into the new age group. All other teams reset to provisional (~1000). RD expands post-transition using `max(end_RD × 1.15, floor)` to reflect increased uncertainty from the age group shift.

### Does the U13/U14 fix deployed May 17 produce clean validation data before June 1?

**ELO's position: Yes, with one condition.**

Deploying U13/U14 on May 17 gives 15 days before June 1. That window is sufficient to:
1. Run the May 20–22 validation (3–5 days post-deploy)
2. Confirm rating shifts are within expected range
3. Establish a 10-day stability baseline before the June 1 transition runs

The condition: do not deploy the Boys Option A changes and the U13/U14 fix in the same session or within 48 hours of each other. Interleaved recomputes within 2 days produce overlapping rating changes that cannot be cleanly attributed. See Section 3 for the explicit sequence.

**June 1 confound risk:** The age group transition will cause rating movement as U14s become U15s, U15s become U16s, etc. This movement is expected and large — 20–30% of teams will see their rating context shift. However, because calibration changes (U13/U14 fix, Boys Option A) will have been applied and validated by May 22–28, the June 1 rating movements can be cleanly attributed to age group transition, not to calibration changes. This is the key reason to sequence all calibration work before May 28.

**What NOT to do:** Do not combine any calibration recompute with the June 1 migration session. If the May 18–20 window slips past May 28, push the MLS NEXT tier split to the June 7 post-transition window — do not combine it with June 1. The June 1 Risk Register (R10) documents this constraint explicitly.

---

## Section 3: Recommended Deployment Sequence

### Option A Sequence (Boys Option A PASS by May 9)

| Date | Action | Owner | Validation Window |
|------|--------|-------|------------------|
| May 17 | Deploy U13/U14 calibration fix to `compute-rankings.js`; run full recompute | Presten | 3–5 days |
| May 17 (same session) | Take pre-deploy snapshot (baseline ratings before fix applies) | Presten | — |
| May 20–22 | ELO validates U13/U14 results against calibration-fix-validation-results spec; confirms rating shifts are within expected range per Calibration Fix Pre-Deploy Checklist | ELO | — |
| May 23 | Deploy Boys calibration changes (parameters TBD from Option A results); run recompute | Presten | 3–5 days |
| May 26–28 | ELO validates Boys changes; confirms no U13/U14 interference (separate recomputes should produce separable attribution) | ELO | — |
| May 18–20 | MLS NEXT tier split: FORGE runs event classifier pre-flight; if ≥ 90% classification quality confirmed, Presten applies UPDATE and recompute | FORGE + Presten | 3–5 days |
| May 25–27 | ELO validates MLS NEXT split results (Boys U13–U17 Academy teams expected to drop ~20–40 pts; Homegrown expected to remain ~160) | ELO | — |
| May 31 | All calibration changes applied and validated. Pre-migration DB backup (`pg_dump youth_soccer --table=rankings > backup_pre_migration_2026-05-31.dump`) | Presten | — |
| June 1 | Age group transition runs on fully validated, stable calibration state | Presten | — |

> **Ordering note:** MLS NEXT tier split (May 18–20) runs in parallel with U13/U14 validation (May 20–22). These are independent — MLS NEXT changes Boys Tier 1 values; U13/U14 changes K-factors and RD floors across all age groups. They do not interfere. ELO can validate both concurrently.

### Option B Sequence (Boys Option A FAIL by May 9)

| Date | Action | Owner |
|------|--------|-------|
| May 17 | Deploy U13/U14 calibration fix; run full recompute | Presten |
| May 20–22 | ELO validates U13/U14 results | ELO |
| May 18–20 | MLS NEXT tier split: FORGE pre-flight + Presten UPDATE | FORGE + Presten |
| May 23+ | Begin Boys calibration investigation (Brier analysis per Boys Brier Analysis Scope Spec). Do NOT apply Boys calibration changes until investigation concludes — Boys parameters remain unchanged through June 1. | ELO |
| May 31 | Pre-migration DB backup | Presten |
| June 1 | Age group transition runs on U13/U14-fixed + MLS NEXT-split + Boys-unchanged calibration state | Presten |
| June 7+ | Post-transition window: apply Boys calibration changes if investigation concludes in time; otherwise defer to July | Presten + ELO |

**ELO's recommendation:** Prefer Option A if Boys Option A results arrive by May 5. Option B is lower risk from an implementation standpoint (fewer moving parts in May), but leaves Boys calibration unimproved through the DSS period and into the June 1 transition. If Boys rankings are excluded from DSS (Option A FAIL), delaying Boys calibration changes to July has no DSS consequence, only an ongoing accuracy cost for Boys rankings.

---

## Section 4: Validation Criteria for Each Change

### U13/U14 Fix Validation

Reference: `Rankings/calibration-fix-validation-results-2026-04-23.md`

Three key metrics ELO checks to confirm the fix worked as intended:

1. **U13 Brier score improvement:** Post-fix U13 blended Brier should move toward ≤ 0.24 (from 0.312 pre-fix). ELO does not require a complete Brier run in the May 20–22 window — but a spot check on the rating distribution for U13 teams (median rating, top-10 team list) should look plausible. If the U13 top-10 list changes dramatically (unfamiliar teams, implausible ratings > 1,700), flag before declaring success.

2. **Rating shift magnitude is within expected range:** The U13/U14 fix is a K-factor and RD floor change — it does not directly change ratings, it changes how quickly ratings respond to future games. The post-recompute shift should be small for most teams (< 30 points). If any age group shows average shifts > 50 points, investigate before clearing the fix.

3. **No age group is suppressed below its expected range:** U13 Gold band teams should still be above 1,500. U14 Silver band teams should still be in 1,350–1,499. If the recompute collapsed ratings systematically, there is a bug in the parameter application — stop and escalate before running the Boys Option A changes.

**Go/no-go threshold:** If any metric triggers investigation (U13 top-10 looks wrong, average shifts > 50 pts in any age group, or Gold/Silver band collapses), hold the Boys calibration changes and escalate to SENTINEL before proceeding.

### Boys Calibration Validation (Option A PASS)

Reference: `Rankings/Boys Option A — Decision Document.md`

After applying Boys calibration changes:

1. **Boys/Girls rating delta narrows:** The primary result of a successful Boys calibration adjustment is that Boys and Girls ratings for the same age group converge toward the expected ≤ 75-point divergence threshold. ELO re-runs the Query 1 logic from the Option A spot check after the recompute to confirm the delta has not worsened.

2. **Boys MLS NEXT teams remain anchored:** Boys MLS NEXT Homegrown teams should still have ratings in the 1,400–1,700 range post-calibration. If any Boys MLS NEXT teams fall below 1,300 after the Boys calibration changes, investigate before clearing — the Boys MLS NEXT calibration (160/135 split) is empirically validated and should not move.

3. **No Girls ratings are affected:** Boys and Girls calibration parameters are applied independently. A recompute that changes Boys K-factors should not alter Girls ratings. ELO spot-checks three Girls Gold-band teams before and after the Boys recompute to confirm no cross-contamination.

**Go/no-go threshold:** If Boys MLS NEXT teams shift > 40 points in aggregate, or if any Girls rating changes more than 5 points post-Boys-calibration recompute, escalate to SENTINEL immediately — the changes may have been misapplied as gender-agnostic rather than Boys-specific.

---

## Section 5: Dependencies and Open Questions for Presten

| Item | Who Provides | Needed By | What It Unlocks |
|------|-------------|-----------|----------------|
| Boys Option A spot check results (3 queries from execution package) | Presten runs queries against `youth_soccer` DB | May 5 (target) / May 9 (hard deadline) | Active sequence selection (Option A vs Option B); Boys Decision Document filing |
| `compute-rankings.js` code change for U13/U14 parameters (K and RD floor values per age group) | Presten applies the change specified in `task-2026-04-23-u13-u14-calibration-fix.md` | May 16 (before May 17 deployment session) | U13/U14 calibration fix cannot run until code change is applied |
| June 1 transition mechanism confirmation: is `age_group_advance()` automatic or does Presten run a manual step? | ELO's current understanding: the function fires automatically after the sequencing check query confirms 0 rows in old season. Presten should confirm this before May 31. | May 28 | If manual, Presten needs to add it explicitly to the June 1 session plan. If automatic, the June 1 Risk Register checklist is sufficient. |
| FORGE: event classifier pre-flight for MLS NEXT tier split (classification quality check ≥ 90%) | FORGE delivers and runs classifier pre-flight | May 17 (before May 18–20 session) | MLS NEXT tier split cannot deploy until classifier quality is confirmed. If < 90% quality, deployment pushes to June 7. |
| Pre-migration DB backup before June 1 | Presten | May 31 | Required before any June 1 migration action per the Risk Register. Not optional. |

---

## References

- `task-2026-04-23-u13-u14-calibration-fix.md` — calibration parameters and code change spec
- `Rankings/calibration-fix-pre-deploy-2026-04-23.md` — pre-conditions checklist for U13/U14 fix
- `Rankings/Boys Option A — Decision Document.md` — Boys calibration decision gate
- `Rankings/June 1 Age Transition — Risk Register.md` — June 1 timing constraints and risk register (especially R10: do not combine calibration and migration)
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — monitoring framework for post-change stability
- `Rankings/MLS NEXT Tier Split — Calibration Application Spec.md` — MLS NEXT tier split spec and FORGE pre-flight

*ELO — 2026-04-26*
