---
title: Pre-April-28 Baseline Snapshot Spec
type: snapshot-spec
author: ELO
date: 2026-04-24
status: ready
related: "[[Post-April-28 Verification Spec]]", "[[Post-April-28 Rating Shift Analysis Spec]]"
tags: [calibration, ga-aspire, baseline, april-28, evo-draw]
---

# Pre-April-28 Baseline Snapshot Spec

> **Purpose:** Capture the before-state required to validate that the April 28 GA ASPIRE fix and ECNL dedup fix moved the right teams in the right direction by the right amount. Without this snapshot, post-fix verification can only confirm the fix ran — not that it worked correctly.
>
> **Critical timing: Run these queries BEFORE the April 28 session begins.** Recommended: April 26 or 27.

---

## Section 1: U13G Top-50 Snapshot

The primary DSS demo cohort. Capture the full top-50 before any fix runs.

```sql
-- Save output as: reports/pre-april28-u13g-top50.txt
SELECT t.name, r.rating, r.rank, t.source_org_id
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13G'
  AND r.rank <= 50
ORDER BY r.rank;
```

Save this output verbatim (copy output from psql terminal to file). After April 28 recompute, ELO will compare line by line.

**Expected post-fix:** GA ASPIRE-affiliated teams in the U13G top-50 will have higher ratings; rank order may shift at boundaries. Football Academy NJ should remain in or move up in the top-50.

---

## Section 2: GA ASPIRE Teams Snapshot

Capture the teams most affected by the fix — all teams with games in events currently tagged `ga` that should be `ga_aspire`, plus teams already correctly tagged.

```sql
-- Save output as: reports/pre-april28-ga-aspire-teams.txt
SELECT t.name, r.rating, r.rank, r.age_group, r.gender,
       e.event_tier AS current_tier,
       COUNT(g.id) AS games_in_tier
FROM rankings r
JOIN teams t ON t.id = r.team_id
JOIN games g ON (g.home_team_id = t.id OR g.away_team_id = t.id)
JOIN events e ON e.id = g.event_id
WHERE e.event_tier IN ('ga', 'ga_aspire')
GROUP BY t.name, r.rating, r.rank, r.age_group, r.gender, e.event_tier
ORDER BY r.age_group, r.gender, r.rank;
```

**Expected magnitude of rating shift for Girls teams moving from `ga` to `ga_aspire` calibration:** 15–40 points upward. Boys GA ASPIRE teams should not move (Option A: both tiers at cal=100 for Boys — see `Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md`).

---

## Section 3: Overall Distribution Snapshot

Broad baseline to confirm the fix only moves GA ASPIRE-affected cohorts, not the full ranking population.

```sql
-- Save output as: reports/pre-april28-distribution.txt
SELECT age_group, gender,
       COUNT(*) as team_count,
       ROUND(AVG(rating)::numeric, 1) as avg_rating,
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating) as median_rating,
       MAX(rating) as max_rating,
       MIN(rating) as min_rating
FROM rankings
WHERE rank IS NOT NULL
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

After April 28, ELO runs this again. Any age group / gender pair that shifted significantly and is NOT GA ASPIRE-heavy is an anomaly — investigate before filing the post-fix report.

**Acceptable shift:** ≤5 points average change for non-GA ASPIRE cohorts. GA ASPIRE-heavy cohorts (primarily U13G–U17G) may shift more due to the fix.

---

## Section 4: Execution Instructions for Presten

**Run these queries BEFORE April 28.** Recommended timing: April 26 or 27.

1. Open psql connection to `youth_soccer` database
2. Run Section 1 query — save output to `reports/pre-april28-u13g-top50.txt`
3. Run Section 2 query — save output to `reports/pre-april28-ga-aspire-teams.txt`
4. Run Section 3 query — save output to `reports/pre-april28-distribution.txt`
5. Record timestamp: __________________________
6. **Do not run April 28 changes until all three files are saved.**

Estimated time: 5 minutes.

If Presten cannot run before April 28, ELO will note in the post-fix analysis that no baseline was available and the magnitude analysis cannot be confirmed — only direction can be inferred from verification spec checks.

---

## Section 5: Post-Fix Comparison Methodology

After April 28 recompute completes, ELO runs the following comparison per `Rankings/Post-April-28 Rating Shift Analysis Spec.md`:

1. **U13G top-50:** Compare each row in `pre-april28-u13g-top50.txt` against a fresh Section 1 query. Record teams that changed rank by > 3 positions.

2. **GA ASPIRE team deltas:** For each team in `pre-april28-ga-aspire-teams.txt` whose `current_tier` was `ga` (incorrectly) — confirm post-fix rating increased. For teams correctly tagged `ga_aspire` already — confirm no change.

3. **Distribution:** Re-run Section 3. Compare per-cohort averages. Any non-GA-ASPIRE cohort that shifted > 5 points avg: investigate.

4. **Football Academy NJ:** Must appear in post-fix top-50, rated ≥ pre-fix (if it was in a GA ASPIRE event, rating goes up; if not, unchanged). This is the DSS demo anchor.

ELO documents findings in `Reports/post-april28-rating-shift-analysis-[date].md` per `Rankings/Post-April-28 Rating Shift Analysis Spec.md`.

---

## References

- `Rankings/Post-April-28 Rating Shift Analysis Spec.md` — the April 29 analysis this baseline feeds
- `Product & Planning/April 28 Escalation Reference Card.md` — the session this spec precedes
- `Rankings/League Hierarchy.md` — calibration values for expected rating shift magnitude
- `Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md` — Boys GA ASPIRE = 100, no net impact
