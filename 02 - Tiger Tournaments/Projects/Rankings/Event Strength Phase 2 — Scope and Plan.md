---
title: Event Strength Phase 2 — Scope and Plan
type: design-spec
author: ELO
date: 2026-04-25
status: scope-complete
related: "[[Event Strength Phase 1 — Full Execution Package]]", "[[Event Strength Ratings — Implementation Plan]]", "[[League Hierarchy]]"
tags: [event-strength, phase-2, design, rankings, evo-draw]
---

# Event Strength Phase 2 — Scope and Plan

> **Purpose:** Scope Phase 2 before Phase 1 deploys, so Phase 1's DDL includes any columns Phase 2 needs. The primary architectural question: does Phase 1's `event_strength` schema need modification to support Phase 2? Answer: **No — Phase 1 DDL is sufficient.** Phase 2 reads `strength_percentile` and `strength_class` directly from Phase 1's table. No schema migration required.

---

## Section 1: Phase 1 → Phase 2 Handoff

### What Phase 1 Delivers

Phase 1 (full execution package in `Event Strength Phase 1 — Full Execution Package.md`) produces:

- `event_strength` table with schema:
  - `event_id`, `age_group`, `season`
  - `avg_team_rating` — average rating of teams participating in the event
  - `strength_class` — 'High' (≥p75), 'Medium' (p33–p74), 'Low' (<p33)
  - `strength_percentile` — 0–100 percentile rank within (age_group, season)
  - `competitive_spread` — standard deviation of participating team ratings
  - `upset_rate` — fraction of games where lower-rated team won
  - `team_count`

- Events classified into three strength tiers (High / Medium / Low)
- No change to the ranking computation pipeline — Phase 1 is read-only from rankings' perspective

### What Phase 2 Adds

**Phase 2 applies `strength_percentile` as a K-factor multiplier during Glicko-2 game processing.** A win at a High-strength event produces a larger rating gain than a win at a Low-strength event, and a loss at a High-strength event produces a smaller rating loss.

The ranking pipeline reads the `event_strength` table during game processing and scales the K-factor per game based on the event's `strength_percentile`.

This is the recommended interpretation over alternatives because:
- K-factor scaling integrates naturally with the existing Glicko-2 formula — no new parameters or post-hoc adjustments
- It applies symmetrically to wins and losses (correct behavior: high-strength events should both reward wins more AND penalize losses less)
- It is auditable: `K_adjusted` can be logged per game for debugging

---

## Section 2: Core Design — Applying Event Strength to Rankings

### Proposed Formula

```
K_adjusted = K_base × event_strength_factor

where:
  event_strength_factor = 0.70 + (strength_percentile / 100) × 0.60

and therefore:
  event_strength_factor ranges from 0.70 (p0, weakest events) to 1.30 (p100, strongest events)
  event_strength_factor = 1.00 at p50 (median events are neutral — no change from baseline)
```

**Worked examples:**

| Event Strength Percentile | strength_class | event_strength_factor | K_base (U13) | K_adjusted |
|--------------------------|---------------|----------------------|-------------|-----------|
| p90 (High) | High | 1.24 | 24 | 28.8 |
| p75 (High threshold) | High | 1.15 | 24 | 27.6 |
| p50 (Median) | Medium | 1.00 | 24 | 24.0 |
| p33 (Medium threshold) | Medium | 0.90 | 24 | 21.6 |
| p10 (Low) | Low | 0.76 | 24 | 18.2 |

For U14 (K_base = 28): p90 event yields K_adjusted = 34.7. Maximum effective K is 1.30 × K_base — bounded by the existing K_base per age group.

### Symmetry

The scaling applies symmetrically to wins AND losses. This is intentional:
- A win at a p90 event gives +28.8 Elo points (vs +24 baseline) — rewards competitive performance
- A loss at a p90 event takes −28.8 points (vs −24 baseline) — penalizes dropping to elite competition

This symmetry prevents the perverse case where teams farm High-strength events for win-credit while the loss penalty stays at baseline.

### Calibration Bounds Reference

The Girls GA ASPIRE overcalibration was ~40 points of distortion — the threshold at which distortion becomes visually wrong in rankings. Phase 2's maximum per-game adjustment (±30% × K_base) produces at most:

- U13 single-game max boost at p100 event: 24 × 0.30 = **+7.2 points above baseline**
- U14 single-game max boost: 28 × 0.30 = **+8.4 points above baseline**

A team playing 10 games at a p90 High-strength event would accumulate at most ~50 bonus points above baseline across those games (assuming all wins). This is within calibration noise range for a team with 50+ total games. The GA ASPIRE distortion case (40 pts) was a systematic per-league offset applied to all games — not per-event K-factor scaling applied proportionally. The Phase 2 adjustment is much gentler in practice.

**Maximum total Phase 2 impact estimate:** For a team that exclusively plays High-strength events across a season (~50 games), the total Phase 2 adjustment vs. Phase 1 baseline is approximately **80–120 points over a full season.** This is significant enough to matter for top-ranked teams and modest enough to be well within calibration margin.

---

## Section 3: Schema Impact Assessment

**Phase 1 DDL does NOT need modification.** Phase 2 reads from Phase 1's existing schema.

The Phase 2 computation reads:
```sql
SELECT es.strength_percentile
FROM event_strength es
WHERE es.event_id = [game.event_id]
  AND es.age_group = [game.age_group]
```

No additional columns are needed in `event_strength` for Phase 2. The `strength_percentile` column (already in Phase 1 schema) is the only input to the K-factor scaling formula.

**Conclusion: Phase 1 can deploy as-is. Phase 2 reads from it without schema modification.**

This is the primary reason this document was written before Phase 1 deployed. The answer is no schema change needed — proceed with Phase 1 DDL as specified.

---

## Section 4: Implementation Estimate

Based on Phase 1 as reference (7 hours for schema + computation + validation):

| Component | Scope | Est. Hours |
|-----------|-------|-----------|
| Schema changes to `event_strength` | None required | 0 |
| Ranking pipeline modification | Modify `compute-rankings.js` (or equivalent) to look up `event_strength.strength_percentile` per game during Glicko-2 update and apply K-factor scaling formula | 3–4h |
| Database index (performance) | Add index on `event_strength (event_id, age_group)` to support per-game lookups in the pipeline | 0.5h |
| Validation | Run Brier score comparison: current pipeline vs Phase 2 pipeline on a held-out game set. Confirm Phase 2 Brier is ≤ Phase 1 Brier for U13G and U14G (primary calibration-sensitive age groups) | 2–3h |
| Spot-check | For 5 known High-strength tournaments: confirm that teams' ratings moved in the expected direction (teams winning at High-strength events should have higher post-season ratings under Phase 2 vs Phase 1) | 0.5h |
| **Total** | | **6–8 hours** |

**Definition of "Phase 2 is working correctly":**
1. Brier score for U13G and U14G is equal to or lower than Phase 1 Brier score on a held-out game set
2. The top 10 U13G teams under Phase 2 are all from High-strength event contexts (ECNL, GA, MLS NEXT) — confirming the signal direction is correct
3. No team's rating changed by more than 200 points when switching from Phase 1 to Phase 2 (would indicate the formula is numerically unstable for some edge case)

---

## Section 5: Timeline Recommendation

### Constraints

- **DSS is May 16** — Phase 1 is the DSS feature; Phase 2 is post-DSS by design
- **Calibration changes deploy May 18–20** — K-factor and RD floor changes to U12/U13/U14/U19. Do NOT run Phase 2 during or immediately after calibration deployment. Phase 2 modifies K-factor behavior. Running Phase 2 while K-factor baseline is also changing makes it impossible to isolate the source of any unexpected Brier movement.
- **Club Rankings session is May 2–5** — Phase 2 is independent of Club Rankings but shares Presten's implementation calendar. Give Club Rankings clear runway before Phase 2 competes for session time.
- **Phase 1 must be validated before Phase 2 begins** — Phase 2 Brier validation requires Phase 1 event strength data to be correct and stable. Phase 1 is not stable until the post-compute validation queries (V1–V4) pass and at least one full nightly pipeline cycle has run after Phase 1 deploys.

### Earliest Phase 2 Start Date

The blocking constraint: calibration deployment May 18–20. Phase 2 should not run until calibration has been stable for at least 14 days post-application. **Earliest Phase 2 start: June 2, 2026.**

### Recommended Target Window

**June 9–15, 2026** — after:
- Calibration changes are applied and monitoring shows no unexpected behavior (May 18–25)
- Post-calibration monitoring spec has been run for the 7-day minimum (`Post-Calibration Monitoring Spec — May 2026.md`)
- Boys Brier analysis is complete (target May 17–30, per `Boys Brier Analysis Scope Spec — May 2026.md`) and any Boys K-factor adjustment is incorporated as the new baseline

This window puts Phase 2 deployment at approximately 3 weeks after DSS, with both calibration and Boys analysis complete. Rankings should be stable and Phase 2's signal can be cleanly measured.

**If Boys Brier analysis reveals Boys GA ASPIRE needs a K-factor change (Option B):** apply that change before Phase 2. Running Phase 2 with an incorrect Boys K-factor baseline would make the Phase 2 Brier validation inconclusive.

---

## Related

- [[Event Strength Phase 1 — Full Execution Package]] — Phase 1 DDL; confirmed as sufficient for Phase 2
- [[Event Strength Ratings — Implementation Plan]] — full feature scope and design
- [[Calibration Production Runbook — May 2026]] — May 18–20 calibration deployment (do not run Phase 2 during this window)
- [[Boys Brier Analysis Scope Spec — May 2026]] — prerequisite for Phase 2 Boys baseline (target May 17–30)
- [[Post-Calibration Monitoring Spec — May 2026]] — 7-day monitoring window required before Phase 2

*ELO — 2026-04-25*
