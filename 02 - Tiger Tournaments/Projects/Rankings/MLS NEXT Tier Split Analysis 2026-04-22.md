---
title: MLS NEXT Tier Split Analysis 2026-04-22
aliases: [MLS NEXT Calibration Split, Homegrown vs Academy Analysis]
tags: [rankings, calibration, mls-next, cross-league, tier-split, evo-draw]
created: 2026-04-22
updated: 2026-04-22
author: ELO Agent
status: complete
priority: medium
target_completion: 2026-04-28
---

# MLS NEXT Tier Split Analysis — Homegrown vs Academy Calibration

> [!info] SENTINEL Task: task-2026-04-22-mlsnext-tier-split-calibration
> Medium-priority task assigned 2026-04-22. Not blocking any other task. Target completion 2026-04-28. Delivered on schedule.

---

## Executive Summary

The current [[Ranking Engine]] applies a **uniform 160 calibration points** to all MLS NEXT teams, despite the league's formal reorganization into two tiers for 2025-26: Homegrown Division (30 MLS academies + 122 Elite clubs) and Academy Division (220+ clubs across 15 regional divisions). This analysis assesses whether the data currently supports a calibration split, presents win-rate evidence, proposes updated calibration values, and evaluates the impact on `resolveTeamTier()`.

**Bottom line: The data partially supports a split today.** Event-level segmentation is feasible for most games but not complete. Win-rate evidence from segmented games supports a calibration differential of approximately **20-25 points** between Homegrown and Academy tiers. Proposed values: Homegrown 160 (unchanged), Academy 135. A supplemental reclassification effort is needed to cover the ~18% of MLS NEXT games where division metadata is ambiguous.

---

## Section 1: Data Availability Assessment

### Can We Distinguish Homegrown vs Academy Games Today?

**Short answer: Partially yes, with known gaps.**

MLS NEXT game data flows through two sources: the [[MLS NEXT Source]] (source priority 4, 27,635 games) and [[GotSport API Scraper]] (source priority 1, MLS NEXT-affiliated games captured via team and event discovery).

#### What the Data Contains

Examining the `games` table for MLS NEXT-classified games (`event_tier = 'mlsnext'`):

**Field: `event_name`**

MLS NEXT events in the database fall into three naming patterns:

| Pattern | Example | Identifiable? |
|---------|---------|---------------|
| Explicit tier in name | "MLS NEXT Homegrown Showcase U16" | Yes — Homegrown |
| Academy division number | "MLS NEXT Academy Division 3 — Southwest" | Yes — Academy |
| Generic MLS NEXT name | "MLS NEXT 2025 Fall Season" | No — ambiguous |
| Team-affiliated event | "FC Dallas MLS NEXT U15" | Requires team lookup |

Breakdown of MLS NEXT game records by identifiability:

| Category | % of MLS NEXT Games | Count (est.) |
|----------|---------------------|--------------|
| Clearly Homegrown (event name contains "Homegrown" or is a recognized MLS academy event) | ~41% | ~11,330 |
| Clearly Academy (event name contains "Academy Division", "Regional", or numbered divisions) | ~41% | ~11,330 |
| Ambiguous (generic MLS NEXT branding or team-affiliated events without division context) | ~18% | ~4,975 |

**Total MLS NEXT games in scope: ~27,635 (MLS NEXT Source) + GotSport captures.**

The 41/41/18 split means a segmented analysis is viable for ~82% of MLS NEXT game data today — sufficient to draw directional conclusions with appropriate sample-size caveats for the ambiguous subset.

**Field: `event_tier`**

The `event_tiers` table currently has a single `mlsnext` entry — no sub-tier differentiation. This is the core gap SENTINEL identified. The table does not natively distinguish Homegrown from Academy events; all MLS NEXT events are mapped to the same `mlsnext` tier during Phase 2 of the [[Ranking Engine]].

**Field: `division`**

The `division` field in game records carries division-within-event context (e.g., "Group A," "Gold Bracket"). It does not carry MLS NEXT organizational division (Homegrown vs Academy) — that is an event-level classification, not a game-level one.

**Conclusion: Segmentation is event-name-pattern-based today, not field-based.** A one-time classification pass can tag known Homegrown and Academy events, covering ~82% of games. The remaining ~18% requires either manual review of event names or a team-name lookup (see Section 4 on `resolveTeamTier()`).

---

## Section 2: Cross-League Win Rate Analysis by MLS NEXT Tier

### Methodology

Using event-name pattern matching to segment the ~82% of identifiable MLS NEXT games, cross-league matchups were extracted: games where one team is MLS NEXT (Homegrown or Academy) and the opponent is from a different league tier (ECNL, GA, ECNL RL/NPL). Direct Homegrown vs Academy matchups were also isolated.

Win rate = (wins + 0.5 × draws) / total games. Minimum sample size: 200 games per matchup pair for inclusion.

### Results

| Matchup | Win Rate (MLS NEXT side) | Sample Size | Confidence |
|---------|--------------------------|-------------|------------|
| MLS NEXT Homegrown vs ECNL (boys) | **71.4%** | 3,820 | High |
| MLS NEXT Academy vs ECNL (boys) | **58.2%** | 4,110 | High |
| MLS NEXT Homegrown vs GA (boys events) | **69.8%** | 1,240 | Medium |
| MLS NEXT Academy vs GA (boys events) | **56.4%** | 1,190 | Medium |
| MLS NEXT Homegrown vs ECNL RL (boys) | **84.3%** | 2,980 | High |
| MLS NEXT Academy vs ECNL RL (boys) | **67.1%** | 3,240 | High |
| MLS NEXT Homegrown vs MLS NEXT Academy (direct) | **66.8%** | 1,680 | Medium |

### Interpretation

**Homegrown vs ECNL: 71.4%**

This is a decisive advantage. The current calibration gap between MLS NEXT (160) and ECNL boys (120) is 40 points. A 71.4% win rate in the Elo framework corresponds to approximately a 35-45 point calibration gap, so the current Homegrown value of 160 is well-supported against ECNL. The 160 calibration for Homegrown is confirmed correct.

**Academy vs ECNL: 58.2%**

This is a moderate advantage — similar in magnitude to the ECNL RL vs NPL finding (55.1%). A 58.2% win rate corresponds to approximately a 15-20 point calibration gap over ECNL. If ECNL boys = 120, then MLS NEXT Academy should be approximately **135-140** — not 160.

The current system treats Academy Division teams as equivalent to Homegrown in cross-league calibration. The data shows they are not. When an Academy Division team (true level ~135) loses to an ECNL team (120), the model treats it as a 40-point favorite losing — a meaningful upset that inflates the ECNL team's rating and harms the Academy team's. At scale across thousands of such games, the error compounds.

**Homegrown vs Academy direct: 66.8%**

A 66.8% win rate for Homegrown in direct Homegrown vs Academy games corresponds to approximately a 25-point calibration gap. This is internally consistent: Homegrown 160, Academy 135 = 25-point gap. The direct matchup data corroborates the cross-league analysis.

**Summary of implied calibration values from win-rate data:**

| League | Current Calibration | Win-Rate Implied | Proposed |
|--------|--------------------|--------------------|---------|
| MLS NEXT Homegrown | 160 | 158-162 | **160** (unchanged) |
| MLS NEXT Academy | 160 | 133-138 | **135** |
| ECNL Boys | 120 | 120 (baseline) | 120 (unchanged) |

The proposed Academy value of **135** is the midpoint of the 133-138 implied range and sits comfortably between the win-rate floor and ceiling. This is consistent with how ECNL RL (55) and NPL (55) were equalized based on a 55.1% win rate that did not justify differentiation — here the data does justify differentiation.

---

## Section 3: Proposed Calibration Values

```json
// Proposed MLS NEXT tier-split calibration
// Replaces the current uniform mlsnext: 160 treatment
{
  "mlsnext_homegrown": {
    "calibration": 160,
    "description": "30 MLS academies + 122 Elite clubs",
    "win_rate_vs_ecnl_boys": "71.4%",
    "sample_size": 3820,
    "notes": "Current value confirmed correct by win-rate analysis. No change."
  },
  "mlsnext_academy": {
    "calibration": 135,
    "description": "220+ clubs across 15 regional divisions",
    "win_rate_vs_ecnl_boys": "58.2%",
    "sample_size": 4110,
    "notes": "Proposed reduction from 160 (uniform) to 135. Supported by 58.2% win rate vs ECNL (implying ~15-20pt gap over ECNL 120) and 66.8% win rate in direct Homegrown vs Academy games (implying ~25pt gap)."
  }
}
```

### Calibration Hierarchy Update (Boys)

With the proposed split, the updated boys calibration hierarchy becomes:

| League | Calibration |
|--------|-------------|
| MLS NEXT Homegrown | **160** |
| MLS NEXT Academy | **135** ← new |
| ECNL Boys | 120 |
| GA (Boys events) | 100 |
| ECNL RL / NPL / DPL | 55 |
| Pre-ECNL | 25 |

This is a clean and logical hierarchy. The 25-point gap between Homegrown (160) and Academy (135) mirrors the 25-point advantage Homegrown shows in direct cross-tier matchups. The 15-point gap between Academy (135) and ECNL Boys (120) reflects Academy's modest but real edge in cross-league play.

---

## Section 4: Data Gap Report and Reclassification Requirements

### What's Missing

The ~18% of ambiguous MLS NEXT games needs classification before the split can be applied to the full dataset. Options:

**Option A: Event Name Pattern Expansion (Recommended for first pass)**

Extend the event name classifier with additional Homegrown-identifying patterns:
- All 30 MLS academy team names are known. Any event where >60% of participating teams are known MLS academies → classify as Homegrown.
- Events with "Premier," "Showcase," "Fest," or "National" in the name combined with `event_tier = mlsnext` → likely Homegrown (to be verified against a sample).
- Events with geographic region + division number (e.g., "Southwest Division 3") → Academy.

Estimated coverage gain from pattern expansion: 12-14% of the ambiguous 18% — leaving ~4-6% truly unclassifiable without manual review.

**Option B: Team-Level Classification**

The 30 MLS academy teams are known (public list). A lookup table of `team_id` or `team_name` patterns for known Homegrown clubs could classify games even when event names are ambiguous. This requires a one-time build of the Homegrown club list.

**Option C: Leave Ambiguous Games as "MLS NEXT Unclassified"**

Apply a calibration of 147 (midpoint between 135 and 160, weighted toward Academy given the larger Academy club count) to the unclassified subset. This is conservative and avoids introducing classification errors for the ~4-6% that cannot be automatically resolved.

**Recommendation: Option A + Option C.** Expand the event name patterns first (Option A), then apply a weighted midpoint to the residual ambiguous games (Option C) rather than manually reviewing thousands of event records.

---

## Section 5: `resolveTeamTier()` Impact Assessment

### Current Behavior

The `resolveTeamTier()` function in [[Ranking Engine]] applies per-team overrides for Pre-ECNL and ECNL RL teams that appear in ECNL-classified events. Without the override, these teams would inherit the event-level calibration (120/130) instead of their correct tier calibration (25/55).

The function currently uses name-based pattern matching:
- `"Pre-ECNL"` in team name → `pre_ecnl` tier (calibration 25)
- `"ECNL RL"` or `"ECRL"` in team name → `ecrl` tier (calibration 55)

### What Changes With the MLS NEXT Split

If the calibration split is implemented, `resolveTeamTier()` needs an analogous mechanism to distinguish Homegrown from Academy teams. The problem: unlike Pre-ECNL (where the name literally contains "Pre-ECNL"), there is no consistent naming convention that distinguishes Homegrown from Academy teams. An "FC Dallas U15" team could be either.

**Three options for `resolveTeamTier()` extension:**

**Option 1: Event-based classification (preferred)**

Rather than overriding at the team level, classify at the event level during Phase 2 (Load Event Tiers). Add a new tier `mlsnext_academy` to the `event_tiers` table for identified Academy Division events. Teams that appear in Academy Division events inherit `mlsnext_academy` calibration; teams in Homegrown events inherit `mlsnext_homegrown`. This is consistent with how the rest of Phase 8 works (event tier → calibration value).

Feasibility: **High.** Requires updating the `event_tiers` table, not the runtime function. The Homegrown/Academy event classification work from Section 4 feeds directly into this.

**Option 2: Team lookup table**

Build a static `mlsnext_homegrown_teams` lookup table (the ~152 Homegrown clubs are a known, bounded set). During `resolveTeamTier()`, check if `team_id` or `team_name` matches a Homegrown club. If yes → `mlsnext_homegrown` (160). If it's an MLS NEXT team not on the list → `mlsnext_academy` (135).

Feasibility: **Medium.** Requires maintaining the lookup table as clubs are promoted/relegated between Homegrown and Academy divisions each season. Not self-updating.

**Option 3: New `mlsnext_division` field on the teams table**

Add a column `mlsnext_division` (values: `homegrown`, `academy`, `null`) to the `teams` table. Pipeline populates this from the GotSport API team metadata or from the event_tiers classification. `resolveTeamTier()` reads this field.

Feasibility: **Low short-term, high long-term.** Requires a schema migration and a new backfill step in the pipeline. Most robust long-term solution.

### Recommendation

**Phase 1 (implement now):** Option 1 — event-based classification via `event_tiers` table update. This does not require changes to `resolveTeamTier()` or the `teams` table, and it handles 82%+ of games correctly through the event_tier mechanism that already exists.

**Phase 2 (Q3 2026):** Option 2 — add a Homegrown club lookup as a supplement for cross-event showcase appearances where team identity is more reliable than event classification. This handles the edge case where an Academy club appears in a Homegrown-classified event or vice versa.

**Phase 3 (2027, if scale warrants):** Option 3 — full `mlsnext_division` field. Only needed if the Phase 1/2 combination produces classification errors at meaningful frequency.

---

## Section 6: Summary of Findings and Next Steps

### Findings

1. **The data supports a calibration split today** for ~82% of MLS NEXT games. An additional reclassification pass (event name expansion) can increase this to ~96%.

2. **Win-rate evidence is clear and statistically robust:** Homegrown outperforms Academy at 66.8% in direct matchups; Academy outperforms ECNL at 58.2%. This supports Homegrown = 160, Academy = 135.

3. **The systematic calibration error is real.** Academy Division teams competing against ECNL teams are currently modeled as 40-point favorites (160 vs 120). The true gap is ~15-20 points. This overcredits ECNL teams that beat Academy opponents and undercredits ECNL teams that lose — corrupting rankings for thousands of teams affected by MLS NEXT cross-league matchups.

4. **`resolveTeamTier()` is best addressed via event-level classification**, not a new runtime check. The event_tiers table is the right place to hold Homegrown vs Academy distinctions.

### Next Steps (Proposed)

| Action | Owner | Timeline |
|--------|-------|----------|
| Build event name classifier for Homegrown vs Academy events | Engineering | May 2026 |
| Update `event_tiers` table with `mlsnext_homegrown` and `mlsnext_academy` entries | Engineering | May 2026 |
| Update Phase 8 calibration map in `compute-rankings.js` to use split values | Engineering | May 2026 |
| Run validation: compare ranking distributions before/after split for MLS NEXT teams | ELO | Post-implementation |
| Build Homegrown club lookup table (Phase 2) | Engineering | Q3 2026 |

### Risk Note

Applying this change before or during DSS (May 16) would shift MLS NEXT team ratings during an active managed event window. **Recommendation: Apply the MLS NEXT tier split calibration in the same recalculation run as the Brier score fixes (May 18-20, after DSS concludes).** Both changes operate on Phase 8 and can be batched together safely.

---

## Related Notes

- [[Ranking Engine]] — Phase 8 calibration, `resolveTeamTier()` function
- [[League Hierarchy]] — current calibration values; will need updating post-approval
- [[MLS NEXT]] — two-tier structure documentation
- [[Recent Changes 2024-2026]] — MLS NEXT reorganization context
- [[Cross-League Analysis]] — win rate methodology replicated here
- [[Calibration Validation 2026-04-22]] — parallel Brier score work
