---
title: Post-April-28 Rating Shift Analysis Spec
type: analysis-spec
author: ELO
date: 2026-04-24
status: ready — execute April 29
related: "[[Pre-April-28 Baseline Snapshot Spec]]", "[[Boys GA ASPIRE Calibration Decision — April 2026]]"
tags: [calibration, ga-aspire, post-fix, analysis, april-28, evo-draw]
---

# Post-April-28 Rating Shift Analysis Spec

> **Purpose:** Confirm the April 28 GA ASPIRE fix moved the right teams in the right direction by the right amount. This spec is ELO's analytical layer — it supplements the 15-minute system health check in `Rankings/Post-April-28 Verification Spec.md`.
>
> **Execute on April 29** after Presten or FORGE confirms the recompute completed. Baseline source: `reports/pre-april28-*.txt` files from `Rankings/Pre-April-28 Baseline Snapshot Spec.md`.

---

## Section 1: Expected Shift Profile

State the expected outcome before running any queries.

### Girls GA ASPIRE Teams (events moved from `ga` tier to `ga_aspire` tier)

- **Expected direction:** Rating increases.
- **Expected magnitude:** 15–40 points upward (based on the calibration delta between `ga` and `ga_aspire` in the League Hierarchy).
- **Expected rank shift:** Teams near a rank band boundary (e.g., near Silver ≥1350 threshold) may cross into a higher band. Up to 5 U13G top-50 teams may enter the top-30 or top-20 from lower positions.
- **Cohorts most affected:** Girls age groups with GA ASPIRE game coverage — primarily U13G–U17G. U13G is the DSS demo cohort and the primary audit target.

### Boys GA ASPIRE Teams (Option A: both `ga` and `ga_aspire` at cal=100)

- **Expected direction:** No material change.
- **Tolerance:** ±5 points due to floating-point recompute variance.
- **If Boys move > 15 points on average:** Unexpected. Escalate to SENTINEL before any Club Rankings or Event Strength sessions proceed. Option A may be incorrect or the Boys calibration map has an inconsistency.

### All Other Teams (no GA ASPIRE involvement)

- **Expected direction:** No change.
- **Tolerance:** ±10 points average per cohort (indirect effect via opponent rating shifts).
- **If a non-GA-ASPIRE cohort shifts > 10 points avg:** Indirect effect via opponent ratings. Investigate and document — do not suppress.

---

## Section 2: Girls vs Boys Differential Verification

The most important single check. If Option A is correct, Girls and Boys GA ASPIRE teams should diverge post-fix: Girls go up, Boys stay flat.

```sql
-- Run after April 28 recompute. Compare to reports/pre-april28-ga-aspire-teams.txt baseline.
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

Compare each `(age_group, gender)` row against the pre-fix baseline from `reports/pre-april28-ga-aspire-teams.txt`.

**Expected results table:**

| Cohort | Pre-fix avg | Post-fix avg | Expected delta | Status |
|--------|------------|-------------|---------------|--------|
| U13G (Girls) | [from baseline] | [query result] | +15 to +40 pts | ✅/⚠️/🚨 |
| U14G (Girls) | [from baseline] | [query result] | +15 to +40 pts | ✅/⚠️/🚨 |
| U13B (Boys) | [from baseline] | [query result] | 0 ± 5 pts | ✅/⚠️/🚨 |
| U14B (Boys) | [from baseline] | [query result] | 0 ± 5 pts | ✅/⚠️/🚨 |

Fill in pre-fix values from baseline file. Fill in post-fix values after running the query. Mark ✅ if within expected range, ⚠️ if borderline, 🚨 if threshold exceeded.

---

## Section 3: U13G Top-50 Rank Movement Analysis

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

### Key Teams to Flag

**Football Academy NJ:**
- Must appear in top-50 post-fix.
- Rating should be ≥ pre-fix (if in GA ASPIRE events, rating goes up; if not, unchanged).
- If it dropped: investigate. This is the DSS demo anchor — a drop here is a demo risk.

**Teams that entered the top-50 post-fix:**
- Were they GA ASPIRE affiliated? If yes, this is expected and healthy.
- If no GA ASPIRE affiliation: investigate the source of the improvement.

**Teams that dropped out of top-50:**
- Were they not GA ASPIRE affiliated? Expected to stay stable — investigate if they dropped.
- Were they GA ASPIRE affiliated? Should have gone up, not down — this is an anomaly if it occurs.

---

## Section 4: Output Document Format

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

---

## Section 5: Escalation Thresholds

File a SENTINEL flag in the output document if **any** of these conditions are met:

1. **Boys GA ASPIRE average rating change > 15 points** — Option A violation. Investigate before Club Rankings or Event Strength Phase 1 sessions proceed. May indicate the Boys calibration map has an inconsistency.

2. **Any non-GA-ASPIRE age group / gender pair shifts > 10 points avg** — Unexpected propagation through opponent ratings. Investigate source before proceeding.

3. **Football Academy NJ falls below rank 30 in U13G post-fix** — DSS anchor team vulnerability. Presten needs to know before the Rank Bands implementation session.

4. **More than 5 U13G top-50 teams unexpectedly exited the top-50** — Non-GA-ASPIRE teams that dropped out suggest the fix had wider effects than modeled. Investigate.

5. **Any verified ECNL team's rating changed by > 20 points** — The ECNL dedup flag should not affect ratings. If it does, the flag has unexpected side effects. Flag FORGE.

If none of these: state "No escalations. Fix applied as expected." in the SENTINEL Flags section.

---

## References

- `Rankings/Post-April-28 Verification Spec.md` — the 15-minute health check this supplements
- `Rankings/Pre-April-28 Baseline Snapshot Spec.md` — source of before-state (must exist before April 28)
- `Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md` — Option A: Boys should not move
- `Rankings/Event Strength Phase 1 — Full Execution Package.md` — downstream task gated on this analysis
- `Rankings/DSS Demo Validation Criteria — May 2026.md` — Football Academy NJ status criteria
