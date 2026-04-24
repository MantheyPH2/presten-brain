---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Rank Bands Threshold Validation — 2026-04-24.md"
priority: high
due: 2026-04-27
---

# Task: Rank Bands Threshold Validation — Confirm Gold/Silver/Bronze Cutoffs Before Implementation

## Context

The Rank Bands implementation plan is written. Presten has a 5.5-hour session to implement it. Before that session runs, the Gold/Silver/Bronze rating thresholds must be validated — not assumed.

Your implementation plan specifies threshold values, but those values were designed before the current ratings distribution was known precisely. The question this task answers: **given the actual ratings distribution in the DB today, do the planned thresholds produce a sensible number of teams per band?** If they produce 3 Gold teams out of 10,000 ranked, the thresholds are wrong. If they produce 4,000 Gold teams, the bands are meaningless.

This is ELO's responsibility because ELO defines what "sensible" looks like for the calibration system.

You cannot run the queries yourself — but you can write them precisely and define the expected output ranges, so Presten can run them in 15 minutes and either confirm the thresholds or get the data needed to adjust them.

## What You Need to Do

### Step 1: State the Planned Thresholds

From your Rank Bands implementation plan, document the current proposed thresholds:

| Band | Rating Threshold | Expected % of Teams |
|------|-----------------|---------------------|
| Gold | ≥ [X] | ~5–10% |
| Silver | [Y]–[X-1] | ~20–30% |
| Bronze | [Z]–[Y-1] | ~30–40% |
| Unranked | < [Z] | remainder |

Fill in your planned values. If the plan has different thresholds per age group, document all of them.

### Step 2: Write the Threshold Validation Query

Write the query to check how many teams fall in each band for a representative age group (U13G — the DSS demo age group):

```sql
-- Threshold validation for U13G — adjust thresholds to match your plan
SELECT
  CASE
    WHEN rating >= [GOLD_THRESHOLD] THEN 'Gold'
    WHEN rating >= [SILVER_THRESHOLD] THEN 'Silver'
    WHEN rating >= [BRONZE_THRESHOLD] THEN 'Bronze'
    ELSE 'Unranked'
  END AS band,
  COUNT(*) AS team_count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 1) AS pct_of_total,
  MIN(rating) AS band_min,
  MAX(rating) AS band_max,
  ROUND(AVG(rating), 0) AS band_avg_rating
FROM rankings
WHERE age_group = 'U13' AND gender = 'F' AND rank IS NOT NULL
GROUP BY band
ORDER BY band_max DESC NULLS LAST;
```

Replace `[GOLD_THRESHOLD]`, `[SILVER_THRESHOLD]`, `[BRONZE_THRESHOLD]` with your planned values.

### Step 3: Write the Distribution Query

Write a second query that gives Presten the raw ratings distribution — this is needed to make threshold adjustments if the validation query shows bad band proportions:

```sql
-- Ratings distribution for U13G (percentile breakpoints)
SELECT
  PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY rating) AS p90_rating,
  PERCENTILE_CONT(0.70) WITHIN GROUP (ORDER BY rating) AS p70_rating,
  PERCENTILE_CONT(0.40) WITHIN GROUP (ORDER BY rating) AS p40_rating,
  PERCENTILE_CONT(0.10) WITHIN GROUP (ORDER BY rating) AS p10_rating,
  COUNT(*) AS total_ranked_teams
FROM rankings
WHERE age_group = 'U13' AND gender = 'F' AND rank IS NOT NULL;
```

Explain what the output means: if Gold = ≥p90 and Silver = p70–p90 and Bronze = p40–p70, the bands are roughly 10% / 20% / 30% / 40%. Adjust this guidance based on whatever band proportions you believe are correct.

### Step 4: Write the Decision Rule

Define the acceptance criteria for each threshold. Presten should be able to run the validation query and immediately know whether to proceed with the planned thresholds or adjust them.

Write explicit pass/fail criteria:

```
GOLD threshold passes if: between 5% and 15% of U13G ranked teams qualify
GOLD threshold fails if: fewer than 3% (too exclusive) or more than 20% (too inclusive)

SILVER threshold passes if: between 15% and 35% of U13G ranked teams qualify
SILVER threshold fails if: fewer than 10% or more than 40%

BRONZE threshold passes if: between 25% and 45% of U13G ranked teams qualify
BRONZE threshold fails if: fewer than 15% or more than 50%
```

Adjust these ranges based on your calibration analysis. The percentages should reflect what's meaningful for the Evo Draw audience — clubs at DSS will ask "is my team Gold?" and the answer should feel earned, not automatic.

### Step 5: Write Adjustment Instructions

If the thresholds fail the decision rule, what should Presten do?

Write a clear algorithm:
1. Run the distribution query (Step 3) to see the percentile breakpoints
2. Set Gold threshold to the p[X] percentile value (you decide what X should be based on the target % of Gold teams)
3. Set Silver threshold to the p[Y] percentile value
4. Set Bronze threshold to the p[Z] percentile value
5. Re-run the validation query with the adjusted thresholds
6. Iterate until all three bands pass

This makes threshold adjustment a 5-minute data exercise, not a judgment call made under pressure.

### Step 6: Cross-Check Against the DSS Demo Teams

Write a query to verify that the DSS anchor teams land in appropriate bands at the current thresholds. Football Academy NJ should be Gold. A mid-table team should be Silver or Bronze.

```sql
-- Confirm demo team band assignments
SELECT t.name, r.rating, r.rank,
  CASE
    WHEN r.rating >= [GOLD_THRESHOLD] THEN 'Gold'
    WHEN r.rating >= [SILVER_THRESHOLD] THEN 'Silver'
    WHEN r.rating >= [BRONZE_THRESHOLD] THEN 'Bronze'
    ELSE 'Unranked'
  END AS band
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND (t.name ILIKE '%football academy%'
    OR t.name ILIKE '%wall soccer%'
    OR t.name ILIKE '%FC Stars%')
ORDER BY r.rank;
```

The expected results: Football Academy NJ = Gold. Wall SC = Gold or high Silver. If these don't match expectations, flag the threshold values for review before implementation.

## Deliverable

Write a threshold validation spec to:
`02 - Tiger Tournaments/Projects/Rankings/Rank Bands Threshold Validation — 2026-04-24.md`

The document must:
- State the current planned thresholds (from the implementation plan)
- Include Step 2 and Step 3 queries with your planned values pre-filled (not placeholders)
- Include the pass/fail decision rules with explicit percentages
- Include the adjustment algorithm
- Include the demo-team spot check query

Presten should be able to run every query in this document before starting the 5.5-hour implementation session and confirm in 15 minutes whether the thresholds are production-ready.

## Definition of Done

- Planned thresholds are stated explicitly (no references to "the plan" — copy the values here)
- All queries have actual threshold values filled in (not `[GOLD_THRESHOLD]`)
- Pass/fail decision rule is explicit and numeric
- Adjustment algorithm gives Presten a clear path if thresholds fail
- Demo-team spot check query is present
- Summary in your briefing: planned thresholds confirmed, validation queries ready

## Why This Matters

The Rank Bands implementation session is ~5.5 hours of Presten's time. If it runs with wrong thresholds, the output is a Gold/Silver/Bronze assignment that doesn't reflect real competition quality. That fails the demo: if Football Academy NJ is Silver and a mediocre team is Gold, the system looks broken. This 15-minute pre-flight eliminates that risk at zero cost.

## References

- `02 - Tiger Tournaments/Projects/Rankings/Rank Bands — Implementation Plan.md` — source of current threshold values
- `02 - Tiger Tournaments/Projects/Rankings/Rank Bands Design.md` — design rationale
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Demo Spec — May 2026.md` — demo team requirements (Football Academy NJ as Gold anchor)
