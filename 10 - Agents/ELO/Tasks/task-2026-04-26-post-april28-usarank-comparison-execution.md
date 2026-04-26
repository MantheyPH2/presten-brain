---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-05-03
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Post-April-28 USA Rank Comparison — Execution Package.md"
---

# Task: Post-April-28 USA Rank Comparison — Execution Package

## Context

Two documents address USA Rank comparison:
1. `Rankings/USA Rank Comparison — Scope and Methodology.md` (filed April 23) — long-term methodology for ongoing comparison
2. `Infrastructure/USA Rank Reference Data Sourcing Spec.md` (filed April 26) — how Presten collects the USA Rank snapshot

Neither document specifies the **first comparison** — the specific execution for comparing Evo Draw's corrected (post-April-28 recompute) ratings against USA Rank. This comparison has a distinct purpose: measuring whether the GA ASPIRE fix improved Evo Draw's accuracy on Girls ratings, specifically. It is a point-in-time diagnostic, not the ongoing comparison framework.

The first post-April-28 comparison is important enough to warrant its own execution package for four reasons:

1. **It validates the fix.** If Evo Draw Girls rankings moved in the correct direction AND moved closer to USA Rank, that is direct evidence the fix worked. If Evo Draw moved further from USA Rank on Girls, that is a flag requiring investigation.
2. **It produces the first real DSS accuracy claim.** SENTINEL needs a specific comparison result to tell Presten "Evo Draw is now X% more accurate than before the fix on Girls ratings." Narrative claims without a specific comparison date and methodology are not credible.
3. **It is time-sensitive.** The comparison is most meaningful immediately post-fix — before new games dilute the signal. If ELO doesn't execute the comparison by May 3, the April 28 fix's effect is already mixed with 5 days of new game data.
4. **It differs from the ongoing comparison methodology.** The ongoing methodology (from April 23) is designed for repeated use. The first post-fix comparison may require setup steps (exporting the right rating snapshot, pulling the right USA Rank cohort) that don't need to be repeated each time.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Post-April-28 USA Rank Comparison — Execution Package.md`

---

### Section 1: Comparison Purpose and Scope

One paragraph: what this comparison measures, why April 28 is the reference point, and what constitutes a meaningful result.

Scope:
- **Evo Draw cohort:** Girls teams in USA Rank's tracked leagues — ELO specifies which leagues and age groups
- **USA Rank cohort:** The snapshot collected per `Infrastructure/USA Rank Reference Data Sourcing Spec.md`
- **Comparison metric:** Top-N ranking overlap (N = 20 per age group is the baseline from April 23 methodology) and/or average rating correlation for teams appearing in both systems
- **Primary question:** Did the April 28 GA ASPIRE fix reduce the rating gap between Evo Draw and USA Rank for Girls GA-heavy age groups?

---

### Section 2: Evo Draw Data Export

Query to export post-April-28 Girls ratings for the comparison cohort:

```sql
-- ELO writes the actual query
-- Must be run AFTER April 28 recompute is confirmed (Step 3 of execution log ✅)
-- Returns: team_id, team_name, age_group, gender, rating, league_tier, rank_within_age_group
-- Filter: gender = 'F', age_group IN (...), rated_game_count >= [minimum threshold]
-- Output: saved as Rankings/Exports/evo-draw-girls-export-2026-04-28.csv or equivalent
```

ELO specifies the minimum game threshold for inclusion in the comparison (teams with < N games are excluded from both Evo Draw and USA Rank cohorts for consistency).

---

### Section 3: USA Rank Data Integration

Reference: `Infrastructure/USA Rank Reference Data Sourcing Spec.md`

Describe how Presten's USA Rank data collection integrates with the Evo Draw export:

- Format Presten should collect USA Rank data in (CSV? table? specific columns needed)
- How to align team names between systems (name matching strategy — reference `Rankings/Team Name Matching.md` if applicable)
- Minimum USA Rank data requirements before comparison can run (e.g., "Presten must collect Girls U15–U17 top 50 for at least 3 age groups")
- If USA Rank data is not available by May 3: note the contingency — what partial comparison can ELO run with available data?

---

### Section 4: Comparison Methodology (First-Run Specific)

Three specific analyses for this first comparison:

**Analysis A — Top-20 Overlap Rate**
For each Girls age group: what % of Evo Draw's top 20 appear in USA Rank's top 20?

```
| Age Group | Evo Draw Top 20 | USA Rank Top 20 | Overlap Count | Overlap % |
| U13G | | | | |
...
```

Comparison baseline: what was this overlap rate before April 28? If a pre-April-28 comparison was run (from the April 23 methodology task), use it. If not, ELO notes this is the first data point.

**Analysis B — Girls GA Rating Delta vs. USA Rank**
For Girls GA teams specifically: what is the average rating difference between Evo Draw and USA Rank?

Expected directional result: GA ASPIRE fix should reduce the Evo Draw-vs-USA-Rank gap for Girls GA teams (if they were overcalibrated, correcting them should bring them closer to USA Rank's positioning).

**Analysis C — ECNL/MLS NEXT Girls Rating Comparison**
For Girls ECNL and MLS NEXT teams: did the fix affect Evo Draw's accuracy for the top-tier teams? (It should not — these teams were not GA ASPIRE overcalibrated.)

This is a control group check: if ECNL/MLS NEXT accuracy changed significantly, the fix had unintended scope.

---

### Section 5: Result Interpretation Framework

| Finding | Interpretation | SENTINEL Action |
|---------|---------------|-----------------|
| Top-20 overlap improved ≥ 5% vs. pre-fix | Fix meaningfully improved accuracy | DSS claim: "Rankings now [X]% more accurate for Girls" |
| Top-20 overlap unchanged (< 2% change) | Fix had minimal accuracy impact (may be game volume dilution) | Note in DSS narrative; explain mechanism |
| Girls GA rating delta reduced | Fix closed the USA Rank gap for GA teams | Direct evidence fix worked as intended |
| Girls GA rating delta increased | Fix moved Evo Draw AWAY from USA Rank | Flag to SENTINEL: possible overcorrection |
| ECNL/MLS NEXT accuracy degraded > 5% | Fix had unintended scope | SENTINEL holds Event Strength Phase 1 |

---

### Section 6: Filing Protocol

When comparison is complete:
- File results in `Rankings/USA Rank Comparison Results — April 2026.md` (new file)
- Update DSS Feature Readiness Tracker: USA Rank comparison row → ✅ with one-line result summary
- File summary in next briefing: "Post-April-28 USA Rank comparison complete. Girls GA overlap: [result]. GA rating delta: [direction]. Control group (ECNL/MLS NEXT): [result]. Overall: [improved / unchanged / degraded]."

---

## Quality Standard

- Analysis A must have pre-fix and post-fix numbers side by side, not just post-fix alone
- Analysis B must be Girls GA-specific — do not average across all Girls age groups
- Analysis C control check must be included — this is the primary guard against unintended scope
- If USA Rank data is incomplete (Presten collected fewer age groups than specified), ELO files a partial comparison and notes which age groups are missing

## Why This Matters

SENTINEL cannot make a confident DSS accuracy claim without a specific post-fix comparison result. "Rankings are more accurate" is a marketing claim. "Girls ranking top-20 overlap with USA Rank improved from 35% to 55% after the April 28 fix" is evidence. This execution package gives ELO the methodology to produce that specific, defensible number before the DSS demo on May 16.

## References

- `Rankings/USA Rank Comparison — Scope and Methodology.md`
- `Infrastructure/USA Rank Reference Data Sourcing Spec.md`
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md`
- `Rankings/April 29 Decision Tree — Verdict Filing Document.md` (G2 results feed Analysis B baseline)
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md`
