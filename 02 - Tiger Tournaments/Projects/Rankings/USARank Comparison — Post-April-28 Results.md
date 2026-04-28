---
title: USARank Comparison — Post-April-28 Results
type: benchmark-results
author: ELO
date: 2026-04-27
status: template — ELO fills after running comparison queries (target: April 29 or within 48 hours of Presten providing USARank reference data)
related: "[[USA Rank Post-April-28 Comparison — Execution Package]]", "[[USA Rank Delta Interpretation Spec — April 2026]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]"
tags: [usarank, comparison, benchmark, accuracy, evo-draw]
---

# USARank Comparison — Post-April-28 Results

> **Template — ELO fills on April 29 (or within 48 hours after Presten provides USARank reference data). Do not pre-fill result fields.**
>
> This is Evo Draw's standing accuracy benchmark against USARank. When filled, it becomes the primary competitive comparison reference — cited at DSS, referenced in future calibration reviews, and updated with each major calibration change.

---

## Section 0 — Comparison Metadata

| Field | Value |
|-------|-------|
| Comparison run date | [FILL: date ELO runs the queries] |
| Evo Draw snapshot date | [FILL: last pipeline run date before comparison] |
| USARank data date | [FILL: date of USARank reference data provided by Presten] |
| ELO operator | ELO |
| Comparison scope | Girls U13–U19, GotSport-sourced teams with ≥ 5 games in both systems |
| Prior comparison baseline | None — this is the first structured post-fix comparison |
| Pre-run model reference | `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` |

---

## Section 1 — Top-50 Rank Correlation

ELO runs the comparison query from `Rankings/USA Rank Post-April-28 Comparison — Execution Package.md` against the top 50 teams in each age group. Fill after executing comparison queries.

| Age Group | Kendall's τ or Spearman ρ | Evo Draw median rank | USARank median rank | Delta | Correlation Verdict |
|-----------|--------------------------|----------------------|---------------------|-------|---------------------|
| Girls U13 | [FILL] | [FILL] | [FILL] | [FILL] | [FILL] |
| Girls U14 | [FILL] | [FILL] | [FILL] | [FILL] | [FILL] |
| Girls U15 | [FILL] | [FILL] | [FILL] | [FILL] | [FILL] |
| Girls U16 | [FILL] | [FILL] | [FILL] | [FILL] | [FILL] |
| Girls U17 | [FILL] | [FILL] | [FILL] | [FILL] | [FILL] |
| Girls U18/U19 | [FILL] | [FILL] | [FILL] | [FILL] | [FILL] |

**Interpretation thresholds (from `Rankings/USA Rank Delta Interpretation Spec — April 2026.md`):**
- τ ≥ 0.80: **Strong** — Evo Draw and USARank substantively agree. Evo Draw's accuracy claim is supported.
- τ 0.60–0.79: **Moderate** — note specific divergences. Calibration differences are real but not alarming.
- τ < 0.60: **Weak** — flag to SENTINEL. Investigate cause before DSS claim is made in demo.

**Overall correlation summary:** [FILL after running queries — "Strong across N of 6 age groups / Moderate in U[X]G due to [cause] / Weak in U[X]G — flagged for investigation"]

---

## Section 2 — Notable Divergences (Top 20)

For each age group where correlation is τ < 0.80: list the top 5 teams where Evo Draw and USARank diverge most. Fill after running comparison queries.

| Age Group | Team Name | Evo Draw Rank | USARank Rank | Delta | Likely Cause |
|-----------|-----------|--------------|-------------|-------|-------------|
| [FILL] | [FILL] | [FILL] | [FILL] | [FILL] | [FILL] |

**Likely cause categories:**
- **Coverage gap:** USARank has games Evo Draw doesn't (or vice versa) — most common in regions with poor GotSport adoption
- **Calibration difference:** Same games, different weights — check if divergent team has high GA ASPIRE exposure (pre-fix vs. post-fix artifact)
- **Recency difference:** One system weights recent games more heavily — expect if team had a strong recent result not yet in the other system's data
- **Merge issue:** Evo Draw has a team split or merged that USARank treats differently — flag team name to SENTINEL for team_merges audit
- **Unknown:** ELO cannot determine from data alone — requires further investigation

If **Merge issue** appears in any row: flag team name and ID in ELO briefing for team_merges high-priority audit list.

---

## Section 3 — GA ASPIRE Fix Impact

The April 28 fix reclassified Girls GA ASPIRE events from cal=140 to cal=100, expected to raise Girls GA ASPIRE team ratings by +20 to +40 pts (highly exposed teams) and +5 to +15 pts (average across cohort). Per the Pre-Run Model, this should improve Evo Draw's correlation with USARank for GA-based Girls teams.

**Pre-April-28 baseline values** (from `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` §3):

| Metric | Pre-April-28 (Pre-Run Model estimate) | Post-April-28 (actual — FILL) | Delta (FILL) |
|--------|---------------------------------------|-------------------------------|--------------|
| Girls GA ASPIRE avg rating direction vs. USARank | Expected: GA ASPIRE teams suppressed below USARank by ~20–40 pts | [FILL] | [FILL] |
| Girls GA ASPIRE teams in top-50 U14G (most affected age group) | Expected: underrepresented vs. USARank pre-fix | [FILL — count of GA ASPIRE teams in Evo Draw top-50 vs. USARank top-50] | [FILL] |
| Kendall's τ for U14G (highest GA ASPIRE exposure) | Pre-run estimate: τ 0.65–0.75 (GA suppression creates systematic divergence) | [FILL] | [FILL] |
| Boys GA avg rating vs. USARank | Expected: no change — Boys GA = cal=100 before and after fix | [FILL — should confirm Boys unchanged] | [FILL — should be ≤ 5 pts] |

**GA ASPIRE fix verdict:** [FILL after comparison — "YES, the fix measurably improved Evo Draw's correlation with USARank for GA-heavy age groups" / "NO — correlation unchanged despite expected improvement" / "INCONCLUSIVE — USARank reference data does not include enough GA-based teams to measure"]

**If INCONCLUSIVE:** Note what USARank data would be needed to make a definitive assessment (e.g., specific age group, specific state coverage).

---

## Section 4 — Summary Assessment

[FILL after running all comparison queries — write one paragraph using structure below]

> Overall Evo Draw / USARank correlation: [Strong/Moderate/Weak] across [N] of 6 age groups. Best correlation: [age group] (τ = [X]). Weakest: [age group] (τ = [X]) — likely cause: [coverage gap / calibration difference / unknown]. GA ASPIRE fix: [improved / did not improve / inconclusive] correlation for GA-heavy age groups. [N] teams flagged for merge audit. DSS ranking accuracy claim: [supported / partially supported / not yet supported] by this comparison.

---

## Section 5 — Next Comparison Schedule

| Date | Trigger | Comparison Scope | Document |
|------|---------|-----------------|----------|
| May 9 | Pre-DSS readiness assessment | Girls all age groups + Boys if Option A implementation complete | `Rankings/USARank Comparison — May 9 Results.md` (new document, same template) |
| Post-DSS (date TBD) | Presten request or quarterly review | Full comparison including Boys all age groups | `Rankings/USARank Comparison — Post-DSS Full Results.md` |

**Template reuse:** File the May 9 comparison as `USARank Comparison — May 9 Results.md` following this same structure. Section 3 of the May 9 version will compare the May 9 actual values against this document's April 29 actuals — enabling direct improvement measurement.

---

## Filing Note

When ELO fills this document (April 29 or within 48 hours of receiving USARank reference data), ELO includes in the next briefing:

```
USARank Comparison filed (April 29 actuals).
Overall correlation: [Strong/Moderate/Weak] across [N] of 6 Girls age groups.
Best correlation: [age group] (τ = [X]).
GA ASPIRE fix impact on USARank correlation: [improved / inconclusive / no change].
Merge flags: [N] teams flagged for audit.
DSS accuracy claim status: [supported / partially supported].
```

---

## References

- `Rankings/USA Rank Post-April-28 Comparison — Execution Package.md` — comparison queries and method
- `Rankings/USA Rank Delta Interpretation Spec — April 2026.md` — τ thresholds used in Section 1
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — pre-fix baseline for Section 3
- `Rankings/Team Merges — High-Priority Audit List.md` — destination for merge flags from Section 2
- `Rankings/USA Rank Reference Data — Sourcing Spec.md` — how Presten obtains USARank reference data
- `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md` — DSS claims this comparison validates

*ELO — 2026-04-27 (template)*
