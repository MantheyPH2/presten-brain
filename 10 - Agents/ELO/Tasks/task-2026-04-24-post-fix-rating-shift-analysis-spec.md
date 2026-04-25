---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Post-April-28 Rating Shift Analysis Spec.md"
---

# Task: Post-April-28 Rating Shift Analysis Spec

## Context

After April 28, two things change:
1. GA ASPIRE tier fix — events miscategorized as `ga` are now correctly tagged `ga_aspire`, which changes calibration for affected teams
2. Full rating recompute triggers after the fix

The existing `Rankings/Post-April-28 Verification Spec.md` is a 15-minute system health check: did the fix run, did the column exist, did the recompute fire? It confirms the fix was applied, not that it worked correctly.

This task: write ELO's analytical layer — the April 29 spec that confirms the right teams moved in the right direction by the right amount. This is ELO's primary post-April-28 work product.

**This spec must be written before April 28**, so ELO can execute on April 29 without setup time.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Rankings/Post-April-28 Rating Shift Analysis Spec.md`

### Section 1: Expected Shift Profile

State the expected outcome before writing any queries:

**Girls GA ASPIRE teams** (events moved from `ga` tier to `ga_aspire` tier):
- Expected: rating increases. The events were under-counted as `ga` calibration; correcting to `ga_aspire` raises calibration multiplier.
- Expected magnitude: 15–40 points upward (derive from the Girls calibration delta between `ga` and `ga_aspire` in the League Hierarchy).
- Expected rank shift: teams near the boundary of a rank tier (e.g., near Silver ≥1350 threshold) may cross into a higher band.

**Boys GA ASPIRE teams** (Option A: both `ga` and `ga_aspire` at cal=100):
- Expected: no material change.
- Tolerance: ±5 points due to floating-point recompute variance.
- If Boys move > 15 points on average: unexpected. Escalate to SENTINEL before filing analysis.

**All other teams** (no GA ASPIRE involvement):
- Expected: no change (ratings only depend on their own games and opponents' calibration, not events they didn't play in).
- If a non-GA-ASPIRE cohort shifts significantly (> 10 points avg): indirect effect via opponent ratings. Investigate and document — do not suppress.

### Section 2: Girls vs Boys Differential Verification

The most important single check. If Option A is correct, Girls and Boys GA ASPIRE teams should diverge post-fix (Girls go up, Boys stay flat).

```sql
-- Run after April 28 recompute. Compare to pre-april28-ga-aspire-teams.txt baseline.
SELECT r.age_group, r.gender,
       ROUND(AVG(r.rating)::numeric, 1) AS post_fix_avg_rating,
       COUNT(*) AS team_count
FROM rankings r
WHERE r.team_id IN (
  SELECT DISTINCT g.home_team_id FROM games g
  JOIN events e ON e.id = g.event_id
  WHERE e.event_tier = 'ga_aspire'
  UNION
  SELECT DISTINCT g.away_team_id FROM games g
  JOIN events e ON e.id = g.event_id
  WHERE e.event_tier = 'ga_aspire'
)
GROUP BY r.age_group, r.gender
ORDER BY r.age_group, r.gender;
```

Compare each `(age_group, gender)` row against the pre-fix baseline. Expected: Girls rows show increased avg_rating. Boys rows show no change. Document the delta for each.

### Section 3: U13G Top-50 Rank Movement Analysis

The DSS demo cohort. Compare post-fix rankings against `reports/pre-april28-u13g-top50.txt`.

```sql
-- Post-fix U13G top-50
SELECT t.name, r.rating, r.rank, t.source_org_id
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13G'
  AND r.rank <= 50
ORDER BY r.rank;
```

For each team in the pre-fix snapshot, record:
- Did rating change? By how much?
- Did rank change? By how many positions?

Document in a table: `team_name | pre_rating | post_rating | delta | pre_rank | post_rank | rank_change`

Record and flag:
- **Football Academy NJ:** Must appear in top-50 post-fix. Rating should be ≥ pre-fix. If it dropped: investigate.
- **Teams that entered the top-50 post-fix:** Were they GA ASPIRE affiliated? This is expected.
- **Teams that dropped out of top-50:** Were they not GA ASPIRE affiliated (expected to stay) or GA ASPIRE affiliated (should have gone up, not down)? If a GA ASPIRE team dropped: anomaly, investigate.

### Section 4: Output Document Format

ELO files results at: `02 - Tiger Tournaments/Projects/Reports/post-april28-rating-shift-analysis-[date].md`

The document must have exactly these sections:

```markdown
## Summary
[One paragraph: X teams affected across N age groups. Girls GA ASPIRE average change: +Y points. 
Boys GA ASPIRE average change: Z points (expected ≈0). U13G top-50 stable/shifted (N teams changed rank). 
Fix is confirmed clean / anomaly detected (specify).]

## Girls GA ASPIRE: Rating Movement
[Table: age_group | pre_avg | post_avg | delta | team_count]

## Boys GA ASPIRE: No-Change Confirmation
[Table: age_group | pre_avg | post_avg | delta — expected delta ≈0 for all rows]

## U13G Top-50: Rank Movement
[Table: top 10 movers only. Full diff attached or referenced.]

## Football Academy NJ
[Pre-fix: rating X, rank N. Post-fix: rating X, rank N. Status: expected / unexpected change.]

## Anomalies
[Any unexpected movement outside the GA ASPIRE cohort. Format: team_name, age_group, pre_rating, 
post_rating, delta, why unexpected. If none: "No anomalies detected."]

## SENTINEL Flags
[Items requiring SENTINEL attention. If clean: "None."]
```

### Section 5: Escalation Thresholds

File a SENTINEL flag in the output document if any of these conditions are met:

1. Boys GA ASPIRE average rating change > 15 points (Option A violation — investigate before Club Rankings or Event Strength sessions proceed)
2. Any non-GA-ASPIRE age group / gender pair shifts > 10 points avg (unexpected propagation)
3. Football Academy NJ falls below rank 30 in U13G post-fix (DSS anchor team vulnerability)
4. More than 5 U13G top-50 teams unexpectedly exited the top-50 (non-GA-ASPIRE teams that dropped)
5. Any verified ECNL team's rating changed by > 20 points (ECNL dedup flag should not affect ratings — if it does, the flag has unexpected side effects)

If none of these: state "No escalations. Fix applied as expected." in the SENTINEL Flags section.

## Definition of Done

- Spec written at `Rankings/Post-April-28 Rating Shift Analysis Spec.md`
- All five sections present
- SQL queries in Sections 2 and 3 are complete and copy-paste ready
- Output document format in Section 4 is fully specified (Presten can tell what the deliverable should look like before ELO writes it)
- Escalation thresholds in Section 5 are explicit and enumerated
- Spec complete before April 28 — ELO runs the analysis on April 29 using this spec as the work order
- Summary in briefing: "Post-April-28 rating shift analysis spec written. ELO runs April 29. Girls/Boys differential check in Section 2. U13G top-50 comparison in Section 3. 5 escalation thresholds defined. Output doc: Reports/post-april28-rating-shift-analysis-[date].md"

## Why This Matters

The 15-minute verification spec confirms April 28 ran. This analysis spec confirms April 28 worked. SENTINEL needs both before authorizing Event Strength Phase 1 (which depends on correct GA ASPIRE calibration being in place). Without this analysis, the gate between April 28 and Event Strength Phase 1 is a system health check, not a data quality check. This spec closes that gap.

## References

- `Rankings/Post-April-28 Verification Spec.md` — the 15-minute health check this extends analytically
- `Rankings/Pre-April-28 Baseline Snapshot Spec.md` — source of before-state (task-2026-04-24, ELO)
- `Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md` — Option A means Boys should not move
- `Rankings/Event Strength Phase 1 — Full Execution Package.md` — downstream task gated on this analysis
- `Product & Planning/DSS Demo Validation Criteria — May 2026.md` — Football Academy NJ status criteria
