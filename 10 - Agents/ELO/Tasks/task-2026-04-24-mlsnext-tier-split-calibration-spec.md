---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
deliverable: "02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split — Calibration Spec 2026-04-24.md"
priority: high
due: 2026-04-28
---

# Task: MLS NEXT Tier Split — Calibration Value Proposals for `mlsnext_academy` and `mlsnext_homegrown`

## Context

FORGE has delivered the MLS NEXT event classifier queries (`Rankings/MLS NEXT Event Classifier — Queries 2026-04-24.md`). Those queries are ready for Presten to run against the DB. The output will show what % of MLS NEXT events classify cleanly as Academy vs Homegrown.

What is not yet written: **the calibration values** for the two proposed tiers. The MLS NEXT Tier Split Analysis (`Rankings/MLS NEXT Tier Split Analysis 2026-04-22.md`) proposed:
- `mlsnext_academy`: cal=160 (unchanged from current `mlsnext`)
- `mlsnext_homegrown`: cal=135

But these values were proposed without a full derivation. ELO needs to write the specification: the exact proposed calibration values, the methodology used to derive them, the expected impact on affected teams, and the Brier score improvement projection.

This is the last piece before Presten can run the classifier queries and immediately know whether to apply the split. Without the calibration spec, the classifier results are actionable for the event update but not for the rating recalculation.

## What You Need to Do

### Step 1: Confirm Current MLS NEXT Calibration Value

From `League Hierarchy.md` or `Ranking Engine.md`:
- What is the current `mlsnext` calibration value (K-factor modifier, game weight, or event tier weight — whatever unit the system uses)?
- Is this the same value for Boys and Girls, or gender-specific?

Document the exact parameter(s) being changed. Be precise: "cal=160 means [what]" — translate to what this value does in the rating formula.

### Step 2: Derive the `mlsnext_homegrown` Value

The MLS NEXT tier split analysis proposed 135 for Homegrown. Derive or justify this value:

**Option A: Derive from game result data (preferred)**

If Glicko-2 calibration values are designed so that a win in a tier-X event produces Y rating points, then:
- What is the average team rating in MLS NEXT Academy leagues vs Homegrown leagues?
- Do Homegrown teams tend to win when playing against non-MLS academy teams at a rate that suggests they're competing at a lower level?

Write the SQL to check this (for Presten to run if needed):
```sql
-- Average team rating by event tier (Academy vs Homegrown if already classifiable by event name)
SELECT
  CASE
    WHEN e.event_name ILIKE '%academy%' THEN 'mlsnext_academy'
    WHEN e.event_name ILIKE '%homegrown%' THEN 'mlsnext_homegrown'
    ELSE 'mlsnext_other'
  END AS tier_class,
  AVG(r.rating) AS avg_team_rating,
  COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) AS unique_teams
FROM games g
JOIN events e ON e.id = g.event_id
JOIN rankings r ON r.team_id = g.home_team_id  -- simplified; join both home and away
WHERE e.event_tier = 'mlsnext'
GROUP BY tier_class;
```

If Academy teams average ~1400 and Homegrown teams average ~1200, the competition level difference supports a lower calibration weight for Homegrown.

**Option B: Derive from existing tier calibration ratios**

If data is not available, derive from the existing calibration scale:
- The current scale likely runs: `national` > `ga` > `mlsnext` > `ecnl_regional` > `regional` > `state`
- MLS NEXT Homegrown should sit between `mlsnext` (Academy level) and `ecnl_regional` or `regional`
- If current values are: mlsnext=160, ecnl_regional=120, then 135 sits proportionally between them

Document the derivation and state whether it's data-driven or scale-interpolated.

### Step 3: Write the Expected Impact on Affected Teams

Estimate how the split will affect team ratings:

- Teams that primarily play Homegrown leagues will have their event calibration weight reduced from 160 → 135
  - Effect: games in Homegrown events are worth ~16% fewer rating points than before
  - Teams currently over-rated due to the 160 Homegrown weight will lose some rating points
  - Estimated magnitude: for a team that plays 80% of their games in Homegrown events and currently holds a 1350 rating, estimate the expected new rating post-split

Write the general formula:
```
Expected rating change ≈ (homegrown_games / total_games) × calibration_delta × rating_elasticity
```

You don't need exact numbers — an approximate range is fine. E.g., "Teams 100% in Homegrown leagues could see a -20 to -40 point adjustment; teams split equally could see -10 to -20 points."

State clearly: which teams are affected (Homegrown-heavy teams), which are unaffected (Academy-heavy or non-MLS NEXT teams).

### Step 4: Write the Brier Score Improvement Projection

The existing Brier score validation (`Reports/calibration-held-out-validation-2026-04-23.md`) was done on the current system. The split is expected to improve Brier score for age groups where Homegrown leagues are overrepresented.

Write the projection:
- Which age groups have the most MLS NEXT Homegrown participation? (Typically U15–U19 Boys)
- If those age groups are currently calibrated with an inflated event weight, their predictions will improve when the weight is corrected
- Estimate: "If Homegrown teams' ratings correct by -20 to -40 points, Brier score for U16B could improve by approximately [X]"

This doesn't need to be a precise calculation — it needs to be directionally correct and credible enough that Presten can include it in the calibration approval context.

### Step 5: Define the Transition Strategy

Two questions the split creates:

**1. What happens to teams currently rated under the old `mlsnext` weight?**
When the UPDATE runs and events are reclassified to `mlsnext_academy` / `mlsnext_homegrown`, the existing ratings reflect the 160 weight for all MLS NEXT games. Should the rating history be recomputed from scratch with the new weights, or should only future games use the new weights?

Recommended: **Recompute from scratch** for the affected age groups using the new calibration values. This is consistent with how the U13/U14 calibration fix will work. Document the recompute order: Homegrown-only recompute first to get a clean baseline, then Academy-only if needed.

**2. What if >10% of MLS NEXT events are unclassified?**
The FORGE classifier has a decision rule: if unclassified >10%, do not apply the UPDATE. In that case, what is ELO's fallback?
- Option A: Manual review of unclassified events (list them, classify the top 20 by game volume manually)
- Option B: Leave the split on hold until classifier is refined
- Option C: Apply the split to Academy and Homegrown events only, leaving unclassified at mlsnext=160

State which option you recommend and why.

## Deliverable

Write the calibration spec to:
`02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split — Calibration Spec 2026-04-24.md`

The document must:
- State the proposed calibration values with derivation (not just the numbers)
- Estimate the impact on affected teams with a range
- Include the Brier score improvement projection (directional, not precise)
- Define the transition strategy (recompute vs. prospective-only)
- Document the fallback if classifier quality is below threshold

## Definition of Done

- `mlsnext_academy` and `mlsnext_homegrown` values are stated with explicit derivation
- Affected team impact is estimated (rating range, age groups)
- Brier score improvement is projected (directional, with reasoning)
- Transition strategy is stated and justified
- Fallback for <90% classifier quality is specified
- Summary in your briefing: proposed values, impact estimate, key assumption

## Why This Matters

The event classifier (FORGE) and the calibration spec (this task) together form a complete change package. When Presten runs the classifier queries and confirms >90% classification quality, the next step is to apply both the event UPDATE and the calibration value change. Without this spec, the classifier output is useful but the calibration change still requires another analysis session. This task makes the MLS NEXT tier split a one-session implementation.

## References

- `02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split Analysis 2026-04-22.md` — original proposal, use as source for initial values
- `02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Event Classifier — Queries 2026-04-24.md` — FORGE's classifier (decision rule: ≥90% to proceed)
- `02 - Tiger Tournaments/Projects/Rankings/League Hierarchy.md` — current calibration values per tier
- `02 - Tiger Tournaments/Projects/Reports/calibration-held-out-validation-2026-04-23.md` — Brier score baseline
