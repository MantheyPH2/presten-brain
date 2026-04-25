---
title: April 29 Post-Fix Decision Tree
type: decision-tree
author: ELO
date: 2026-04-25
status: ready
related: "[[Post-April-28 Rating Shift Analysis Spec]]", "[[Pre-April-28 Baseline Snapshot Spec]]"
tags: [calibration, ga-aspire, decision, april-29, event-strength, rank-bands, evo-draw]
---

# April 29 Post-Fix Decision Tree

## Purpose

This document is SENTINEL's authorization tool for April 29. After the April 28 GA ASPIRE fix and ECNL dedup fix, three verification processes produce outputs: Presten's 15-minute health check (`Post-April-28 Verification Spec`), FORGE's infrastructure confirmation (`April 29 Pipeline Health Check — Post-Fix Protocol`), and ELO's rating shift analysis (`Post-April-28 Rating Shift Analysis Spec`). This document takes the combined outputs of all three and maps them to a single binary decision: proceed with Event Strength Phase 1 and Rank Bands as planned, or hold. Use this document on April 29 after all three outputs are in hand.

---

## Four Gate Criteria (G1–G4)

Each gate has one source, one pass definition, and one failure definition.

| Gate | Name | Source | Pass | Fail |
|------|------|--------|------|------|
| **G1** | GA ASPIRE distortion resolved | ELO — Post-April-28 Rating Shift Analysis | Girls GA ASPIRE avg rating increased 15–40 pts; no Girls GA ASPIRE cohort moved less than +10 pts | Any Girls GA ASPIRE cohort shifted < +10 pts avg, or the shift was in the wrong direction (downward) |
| **G2** | ECNL dedup flag set correctly | FORGE — April 29 Pipeline Health Check §2A–2B | `ecnl_verified` column confirmed in schema; all ECNL games show `ecnl_verified = true`; duplicate check = 0 rows | Column missing, any ECNL games showing `false`/null, or duplicate check > 0 rows |
| **G3** | Recompute completed successfully | FORGE — April 29 Pipeline Health Check §3A | `rankings.updated_at` for U13G is after April 28 session start time | Timestamp is from before April 28 session; recompute did not run |
| **G4** | No anomalous outliers introduced | ELO — Post-April-28 Rating Shift Analysis §Escalation Thresholds | No team moved > 50 points in an unexpected direction; no non-GA-ASPIRE cohort shifted > 10 pts avg; Boys GA ASPIRE avg change ≤ 15 pts | Any team outside the GA ASPIRE cohort moved > 50 pts; Boys moved > 15 pts avg; non-GA-ASPIRE cohort shifted > 10 pts avg |

**Pass definition applies to each gate independently. All four must pass to authorize Event Strength Phase 1.**

---

## Decision Tree

The primary cases. SENTINEL only needs to locate the applicable row.

| G1 | G2 | G3 | G4 | Decision | Rationale |
|----|----|----|----|-----------|-|
| ✅ | ✅ | ✅ | ✅ | **PROCEED** — authorize Event Strength Phase 1 and Rank Bands | All gates pass; fix confirmed clean |
| ❌ | ✅ | ✅ | ✅ | **HOLD** — Event Strength and Rank Bands | GA ASPIRE distortion not fully resolved; downstream calibration work is premature |
| ✅ | ❌ | ✅ | ✅ | **HOLD** — Event Strength and Rank Bands | ECNL dedup flag incomplete; ingested data integrity is uncertain |
| ✅ | ✅ | ❌ | ✅ | **HOLD — all downstream** | Recompute failed; ratings still reflect pre-fix state; no implementation can proceed |
| ✅ | ✅ | ✅ | ❌ | **PARTIAL HOLD** — Rank Bands hold; Event Strength proceed with flag | Anomaly detected but fix direction is correct; Event Strength can run with ELO monitoring; Rank Bands wait until anomaly is explained |
| ❌ | ❌ | ✅ | ✅ | **HOLD** — Event Strength and Rank Bands | Multiple fix failures; need a second fix session before implementation |
| Any | Any | ❌ | Any | **HOLD — all downstream** | Broken recompute is highest severity; no downstream authorization until FORGE confirms recompute green |
| ✅ | ✅ | ✅ | ⚠️ borderline | **CONDITIONAL PROCEED** — ELO identifies anomaly source before EOD April 29 | Anomaly is borderline; ELO rules it artifact or confirms investigation; SENTINEL authorizes after ELO report |

**If G4 failure is confirmed Boys moving > 15 pts avg:** Stop both Event Strength and Rank Bands. This is an Option A violation — Boys calibration map has an inconsistency that must be understood before any further implementation session. ELO investigates before next authorization.

---

## Escalation Protocol

One block per gate failure. Named next action for each.

### G1 Fails — GA ASPIRE not fully resolved

- **Who acts:** ELO
- **Immediate action:** File a specific note on remaining distortion magnitude. Which cohorts did not move? By how much? Is this a partial fix (some events reclassified, not all) or a miscalibration (wrong calibration value applied)?
- **SENTINEL action:** Hold Event Strength Phase 1. Propose a second fix session date to Presten within 24 hours.
- **Resolution gate:** ELO confirms remaining distortion is < 5 pts avg per cohort, OR a second fix session is complete and ELO re-runs the shift analysis.

### G2 Fails — ECNL dedup flag missing or incomplete

- **Who acts:** FORGE
- **Immediate action:** Re-apply the `ecnl_verified` flag. Run the backfill migration. Confirm via a second pipeline run (repeat §2A–2B from the health check).
- **SENTINEL action:** Hold Event Strength and Rank Bands until FORGE confirms GREEN in a follow-up briefing.
- **ELO action:** Do not re-run any shift analysis until FORGE confirms the ECNL flag is complete. ECNL-affiliated teams' ratings are uncertain until the dedup flag is verified.

### G3 Fails — Recompute incomplete

- **Who acts:** FORGE
- **Immediate action:** Identify root cause in pipeline logs. Was it a timeout, a code error, or an infrastructure failure? Document the specific error.
- **SENTINEL action:** Call a hold on ALL downstream implementation — Event Strength, Rank Bands, and any Club Rankings sessions. Escalate to Presten immediately. This is the highest-severity failure mode.
- **Resolution gate:** FORGE confirms a successful full recompute run with `rankings.updated_at` timestamps updated for all age groups. SENTINEL re-runs gates G1 and G4 after recompute confirms.

### G4 Fails — Anomalous outlier detected

- **Who acts:** ELO
- **Immediate action:** Identify the specific team(s), their leagues, game count, and merge status. Determine whether the anomaly is a data artifact (bad merge, coverage outlier) or a true distortion introduced by the fix.
  - **If artifact:** Proceed with Event Strength. Flag the artifact team to Presten for post-DSS triage queue. Note in shift analysis: "Anomaly identified as [artifact type]. Does not affect fix validity."
  - **If true distortion:** Hold both Event Strength and Rank Bands. ELO writes a specific finding note. New fix session required before implementation proceeds.
- **SENTINEL action:** Hold Rank Bands until ELO's artifact vs. distortion determination is filed. Event Strength can conditionally proceed if ELO rules artifact within April 29 working hours.

---

## SENTINEL Authorization Statement

When all four gates pass (or G4 is confirmed artifact), copy this block into the SENTINEL briefing for April 29:

---

> **Event Strength Phase 1 — AUTHORIZED**
>
> Date: 2026-04-29
> Authorization basis: All four April 29 gates confirmed.
>
> - G1 (GA ASPIRE resolved): ✅ — Girls avg shift: [+X pts], within expected range 15–40 pts
> - G2 (ECNL dedup flag): ✅ — FORGE confirmed `ecnl_verified = true` for all ECNL games
> - G3 (Recompute complete): ✅ — Rankings updated after April 28 session start
> - G4 (No anomalous outliers): ✅ — No non-GA-ASPIRE cohort shifted > 10 pts; Boys change ≤ 15 pts
>
> Event Strength Phase 1 may proceed per the Full Execution Package (`Rankings/Event Strength Phase 1 — Full Execution Package.md`).
> Rank Bands implementation may proceed per the Rank Bands spec.
>
> Authorized by: SENTINEL
> April 29, 2026

---

## References

- `02 - Tiger Tournaments/Projects/Rankings/Post-April-28 Verification Spec.md` — Presten's 15-min health check (produces G1/G4 inputs)
- `02 - Tiger Tournaments/Projects/Rankings/Post-April-28 Rating Shift Analysis Spec.md` — ELO's analytical layer (G1 and G4)
- `02 - Tiger Tournaments/Projects/Infrastructure/April 29 Pipeline Health Check — Post-Fix Protocol.md` — FORGE's infrastructure verification (G2, G3)
- `02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — Full Execution Package.md` — Gate A and B conditions for the implementation session
