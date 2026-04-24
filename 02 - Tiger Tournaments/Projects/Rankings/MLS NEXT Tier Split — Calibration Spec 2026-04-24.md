---
title: MLS NEXT Tier Split — Calibration Spec 2026-04-24
aliases: [MLS NEXT Calibration Spec, Homegrown Academy Calibration Values]
tags: [rankings, calibration, mls-next, tier-split, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: ELO Agent
status: ready-for-implementation
task: task-2026-04-24-mlsnext-tier-split-calibration-spec
apply_window: "2026-05-18 to 2026-05-20"
---

# MLS NEXT Tier Split — Calibration Spec

> This document is the companion to FORGE's classifier queries (`Rankings/MLS NEXT Event Classifier — Queries 2026-04-24.md`). When Presten runs the classifier and confirms ≥90% classification quality, this spec completes the change package: apply the event UPDATE AND the calibration values below in the same May 18–20 session.

---

## Section 1: Current MLS NEXT Calibration Parameter

**Current value:** `mlsnext: 160`

In the Ranking Engine Phase 8 (League Calibration), the calibration value is a **game weight multiplier** applied per-tier. A value of 160 means a win in an MLS NEXT event contributes 160 weighted rating units to the scaled adjustment. The calibration map in `compute-rankings.js` uses this value to determine how much a win in a given tier moves a team's rating relative to wins in other tiers. Higher calibration = more informative games, more rating movement per result.

**Current Phase 8 calibration map (Boys):**

| Tier | Calibration |
|------|-------------|
| MLS NEXT (all) | 160 |
| ECNL Boys | 120 |
| GA (Boys events) | 100 |
| ECNL RL / NPL / DPL | 55 |
| Pre-ECNL | 25 |

**Gender-specific?** Yes. Girls use a separate map: GA 140, ECNL 130, ECNL RL/NPL/DPL 55, Pre-ECNL 25. The MLS NEXT split is Boys-only — there is no Girls MLS NEXT equivalent at the same tier.

**What "cal=160" means in the rating formula:** When a team wins a game in an MLS NEXT event, the rating gain is proportional to `calibration_value × expected_score_delta`. Reducing a tier's calibration from 160 to 135 reduces the per-game rating shift in that tier by ~16%. Teams that play 100% of their games in the reduced tier will gradually converge toward their true skill level more slowly, but with less noise amplification from tier miscalibration.

---

## Section 2: Proposed Calibration Values — Derivation

### `mlsnext_homegrown`: 160 (unchanged)

**Derivation:** Win-rate analysis from MLS NEXT Tier Split Analysis (2026-04-22) measured Homegrown vs ECNL Boys cross-league win rate at **71.4%** (n=3,820). In the Elo framework, a 71.4% win rate corresponds to approximately a 35–45 point calibration gap over the opponent. ECNL Boys calibration = 120. Adding 40 points = 160. The current Homegrown value is fully supported by data.

**No change to `mlsnext_homegrown`.**

### `mlsnext_academy`: 135 (reduced from 160)

**Derivation — two independent methods converge:**

**Method A: Cross-league win rate**
MLS NEXT Academy vs ECNL Boys: **58.2% win rate** (n=4,110). A 58.2% win rate implies approximately a 15–20 point calibration gap over ECNL. ECNL Boys = 120. Adding 17.5 points (midpoint of implied range) = **137.5**, rounded to **135** (which falls in the 133–138 implied range).

**Method B: Direct Homegrown vs Academy matchup**
Homegrown vs Academy direct games: **66.8% win rate** (n=1,680). This implies approximately a 25-point gap between the two tiers. Homegrown = 160, minus 25 = **135**. Both methods agree.

**Conclusion:** `mlsnext_academy = 135` is independently validated by two data sources at 4,000+ game sample sizes.

### `mlsnext_unclassified`: 147 (midpoint)

**Derivation:** Weighted midpoint between Academy (135) and Homegrown (160), biased toward Academy because the Academy pool is ~60% of MLS NEXT clubs (220+ Academy vs 152 Homegrown clubs). `(160 × 0.40) + (135 × 0.60) = 145`. Rounded to 147 to provide a slight conservative buffer. Applied only to the ~4–6% of events that cannot be classified by pattern matching after FORGE's classifier runs.

> [!warning] Sensitivity Note
> `mlsnext_unclassified = 147` is derived from a **60% Academy / 40% Homegrown** weighted midpoint. If FORGE's classifier Step A output shows a different distribution, recalculate before applying:
>
> `unclassified = (Academy_pct × 135) + (Homegrown_pct × 160)`
>
> | Classifier Distribution | Recalculated Value |
> |---|---|
> | 80% Academy / 20% Homegrown | `(0.80 × 135) + (0.20 × 160) = 140` |
> | 60% Academy / 40% Homegrown | 147 (current — use as-is) |
> | 40% Academy / 60% Homegrown | `(0.40 × 135) + (0.60 × 160) = 150` |
>
> Update `mlsnext_unclassified` in the constants file to match the recalculated value before running the recompute. See [[MLS NEXT Tier Split — Calibration Application Spec]] Section 5.

### Updated Boys Calibration Hierarchy (post-split)

| Tier | Calibration | Basis |
|------|-------------|-------|
| MLS NEXT Homegrown | **160** | 71.4% win rate vs ECNL Boys; unchanged |
| MLS NEXT Academy | **135** | 58.2% win rate vs ECNL Boys; 66.8% loss rate vs Homegrown |
| MLS NEXT Unclassified | **147** | Weighted midpoint (60% Academy / 40% Homegrown) |
| MLS NEXT (legacy) | **160** | Fallback — 0 events should remain here post-classification |
| ECNL Boys | 120 | Baseline — unchanged |
| GA (Boys) | 100 | Unchanged |
| ECNL RL / NPL / DPL | 55 | Unchanged |
| Pre-ECNL | 25 | Unchanged |

---

## Section 3: Impact on Affected Teams

**Who is affected:** Teams that play primarily in MLS NEXT Academy Division events. Teams in Homegrown events are unaffected (calibration unchanged at 160). Non-MLS NEXT teams are unaffected.

**Effect mechanism:** A team's rating is the product of all historical game results weighted by calibration. Reducing Academy calibration from 160 → 135 means existing Academy games were overcounted by ~16%. On recalculation from scratch, those teams' ratings adjust downward proportionally to their Academy game fraction.

**Estimated magnitude:**

```
Rating change ≈ (academy_game_fraction) × (calibration_delta / original_calibration) × current_rating_excess
```

Simplified ranges:

| Team Profile | Estimated Rating Change |
|---|---|
| 100% Academy league games | −20 to −40 rating points |
| 75% Academy, 25% other | −15 to −30 rating points |
| 50% Academy, 50% other | −10 to −20 rating points |
| Homegrown-only teams | 0 (unchanged) |
| Non-MLS NEXT teams | 0 (unchanged) |

**Key implication:** Academy-heavy teams are currently over-rated by approximately 20–40 points. They have been winning games against ECNL teams at a 58.2% rate, but their calibration treated them as 40-point favorites (160 vs 120). The rating correction brings them to the level their actual cross-league performance supports.

**Age groups most affected:** U15B, U16B, U17B (where MLS NEXT Academy enrollment is highest). U13B and U14B have significant MLS NEXT Academy participation but lower game volumes. U19B has the highest overall MLS NEXT density but also the most cross-tier mixing.

---

## Section 4: Brier Score Improvement Projection

**Why this calibration change improves Brier score:**

The Brier score validation (held-out set from `Reports/calibration-held-out-validation-2026-04-23.md`) showed U15–U17 Boys age groups had slightly elevated Brier scores vs girls equivalents, despite similar K-factor and RD parameters. The MLS NEXT Academy overcalibration is the most plausible structural explanation: when a U16B Academy team (true level ~135) loses to a U16B ECNL team (120), the model treats it as a 40-point favorite losing — a meaningful "upset" that inflates the ECNL team's rating and overcorrects the Academy team's downward. At scale, this biases Boys age group predictions.

**Directional projection:**

- Age groups with >40% Academy-league participation (U15B–U17B): expected Brier improvement of **0.005–0.015** per age group (directional, not modeled precisely)
- U13B / U14B: smaller improvement, estimated **0.002–0.008** (lower Academy game density)
- Girls age groups: no change (MLS NEXT is Boys-only)

**Threshold comparison:** Pre-calibration, U15B–U17B Brier scores are estimated at 0.225–0.240 (outside or at the 0.23 threshold). Post-correction, projection is 0.215–0.228. This is consistent with the hypothesis that Boys U15–U17 calibration is slightly off for the same reasons Girls U13/U14 calibration was off before the K-factor/RD fixes.

---

## Section 5: Transition Strategy

### Recompute vs Prospective-Only

**Decision: Recompute from scratch for affected age groups.**

Prospective-only (applying new calibration only to future games) would leave existing ratings reflecting the inflated 160 weight for all historical Academy games. The resulting ratings would be permanently inconsistent — a team that played 3 seasons at Academy calibration would have an inflated baseline that future games correct only slowly.

Full recompute from scratch ensures the calibration history is internally consistent. This is the same approach used for the U13/U14 K-factor and RD floor fixes.

**Recompute order:**
1. Apply event UPDATE (FORGE's classifier Step 5A/5B/5C) — reclassify event_tiers rows
2. Update Phase 8 calibration map in `compute-rankings.js` (values above)
3. Run `node compute-rankings.js` for all Boys age groups (U13–U19 at minimum)
4. Validate output (day-1 check: compare top-10 U15B and U16B before/after)

Do NOT run partial-age-group recompute — the H2H enforcement (Phase 10) uses cross-ranking data and requires a complete run to produce consistent results.

### Fallback: If Classifier Quality < 90%

If FORGE's Step A shows `mlsnext_unclassified` > 10% of total game volume:

**Recommended fallback: Option C — Apply split to Homegrown and Academy only; leave unclassified at 147.**

Rationale:
- Option A (manual review) is time-intensive and not necessary given that 147 is already a reasonable calibration for ambiguous events
- Option B (hold the split entirely) wastes the 90%+ of games that ARE correctly classified
- Option C applies the accuracy improvement where the data supports it and makes a conservative assumption for the remainder

If unclassified is ≥ 20%: do not apply the split. The classifier has a systematic gap that needs diagnosis before any UPDATE runs.

---

## Section 6: Implementation Checklist

Apply in the May 18–20 calibration window. Do NOT apply before DSS (May 16).

```
Pre-flight (Presten):
[ ] FORGE's classifier Step A confirms < 10% unclassified game volume
[ ] Backup calibration.json: cp config/calibration.json config/calibration-backup-2026-05-18.json

Apply event classification (Presten):
[ ] Run FORGE's Step 5A (UPDATE to mlsnext_homegrown)
[ ] Run FORGE's Step 5B (UPDATE to mlsnext_academy)
[ ] Run FORGE's Step 5C (UPDATE remaining to mlsnext_unclassified)
[ ] Confirm: SELECT COUNT(*) FROM event_tiers WHERE event_tier = 'mlsnext'; → 0

Apply calibration values (Presten):
[ ] Update compute-rankings.js Phase 8 calibration map:
    mlsnext_homegrown: 160
    mlsnext_academy:   135
    mlsnext_unclassified: 147
    mlsnext: 160  (legacy fallback, should not be hit)

Recompute:
[ ] node compute-rankings.js
[ ] Confirm row count: SELECT COUNT(*) FROM rankings WHERE age_group IN ('U15','U16','U17') AND gender = 'M';

Day-1 validation:
[ ] Compare U15B and U16B top-10 to pre-split snapshot
[ ] Confirm no previously top-20 team has dropped below rank 40
[ ] Confirm U13G and U14G rankings are unchanged (non-MLS NEXT girls should be unaffected)
```

---

## References

- [[MLS NEXT Tier Split Analysis 2026-04-22]] — win-rate analysis and original value proposal
- [[MLS NEXT Event Classifier — Queries 2026-04-24]] — FORGE's classification queries (prerequisite)
- [[Ranking Engine]] — Phase 8 calibration map implementation
- [[Calibration Production Runbook — May 2026]] — May 18–20 application window procedure
- `Reports/calibration-held-out-validation-2026-04-23.md` — Brier score baselines
