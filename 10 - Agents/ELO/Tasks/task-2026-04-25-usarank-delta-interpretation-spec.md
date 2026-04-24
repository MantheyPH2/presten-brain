---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-24
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Rankings/USA Rank Delta Interpretation Spec — April 2026.md"
---

# Task: USA Rank Delta Interpretation Spec

## Context

FORGE completed `Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` containing 16 SQL queries that compare Evo Draw rankings against USA Rank. This audit is blocked pending Presten's psql access — FORGE cannot run the queries. When Presten runs them, the output will be raw numbers: Evo Draw rating vs. USA Rank rating for overlapping teams, sorted by delta.

What does not exist: a guide that tells ELO (and Presten) how to interpret the delta output. A 50-point rating difference between Evo Draw and USA Rank for the same team could mean:
1. A calibration error in Evo Draw
2. Different game coverage (Evo Draw sees fewer games for that team)
3. Different methodology (USA Rank weighs goals, Evo Draw does not)
4. An expected structural difference (USA Rank is not the ground truth)

ELO is the calibration owner. ELO needs to define — before the queries run — what constitutes a "concerning" delta vs. an "expected" delta, and what ELO will do in each case.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Rankings/USA Rank Delta Interpretation Spec — April 2026.md`

---

### Section 1: Methodology Differences (Expected Structural Deltas)

Document the known methodological differences between Evo Draw and USA Rank that will produce systematic rating gaps. For each difference:
- **What:** [Difference]
- **Direction:** Evo Draw will tend to rate [higher/lower] than USA Rank for teams affected by this
- **Magnitude estimate:** [rough range, e.g., ±10–20 pts]

Known differences to document (ELO should verify and expand):
- Goal differential: USA Rank likely includes GD in rating; Evo Draw's ELO is result-only (W/D/L)
- Game coverage: FORGE's Source Gap Inventory shows known gaps. Teams in leagues with no Evo Draw source will have ratings based on fewer games, possibly diverging from USA Rank.
- Age group mapping: If USA Rank uses different age cutoffs (birth year vs. grade level), the team populations may not perfectly overlap.

### Section 2: Delta Thresholds

Define the thresholds ELO will use to classify a team's delta:

| Delta Category | Range | Interpretation | Action |
|----------------|-------|----------------|--------|
| Expected | [0–N pts] | Within normal methodology variance | No action |
| Mild discrepancy | [N–M pts] | Worth noting; check game coverage | Flag for source review |
| Concerning | [M–P pts] | Possible calibration error | ELO investigation required |
| Critical | [> P pts] | Strong signal of systematic miscalibration | Escalate to Presten before DSS |

Set the thresholds based on ELO's understanding of the calibration system's normal variance. If uncertain, use the Girls GA ASPIRE overcalibration case as a reference point (40 pts was severe enough to require a fix).

### Section 3: Investigation Protocol

When ELO flags a team as "Concerning" or "Critical":

1. **Coverage check first:** Run FORGE's Source Gap Inventory — is this team in a league with a known gap? If yes, delta is likely coverage-driven, not calibration-driven.
2. **Opponent quality check:** Does this team's Evo Draw rating reflect the quality of opponents they played? Check their top 5 rated opponents in Evo Draw vs. USA Rank.
3. **Calibration check:** If not coverage-driven: does the team's age_group/gender/league have a known calibration issue? Cross-reference with League Hierarchy calibration values.
4. **Resolution:** Note whether the delta requires: (a) a calibration value change, (b) a source addition, or (c) no action (expected).

### Section 4: Comparison Scope Limitations

State what USA Rank does and does not cover:
- Does USA Rank cover Boys? Girls? Both?
- Does USA Rank cover all the same leagues Evo Draw covers, or only a subset?
- Is USA Rank considered the authoritative ground truth, or a peer comparison?

If ELO believes Evo Draw's methodology is superior in any dimension, state it explicitly with reasoning. SENTINEL will use this to frame the Presten-facing comparison.

### Section 5: Output Format for Post-Run Review

After Presten runs FORGE's 16 queries, the output will need to be filed somewhere. Define:
- Where Presten should paste the raw query output: `Reports/usarank-accuracy-audit-results-[date].md`
- What ELO will do with the output once filed: "ELO will read the output within one session and apply this spec to classify each delta. Results will be summarized in ELO's next briefing."

---

## Definition of Done

- Spec written at `Rankings/USA Rank Delta Interpretation Spec — April 2026.md`
- All five sections present
- Delta thresholds are explicit numbers (not "large" or "small")
- Methodology differences section cites at least 2 known structural differences
- Investigation protocol is a numbered list ELO can follow when reviewing results
- Summary in briefing: "USA Rank delta interpretation spec written. Thresholds: expected <[N] pts, concerning >[M] pts, critical >[P] pts."

## Why This Matters

When Presten runs the 16 audit queries, the numbers will appear without context. Without this spec, ELO will be reverse-engineering interpretation criteria after seeing the output — reactive rather than calibrated. Writing the spec first forces ELO to commit to what "good" looks like before seeing the results, which is better science and faster decision-making.

## References

- `02 - Tiger Tournaments/Projects/Rankings/USA Rank Ranking Accuracy Audit — April 2026.md` — the 16 queries ELO will interpret
- `02 - Tiger Tournaments/Projects/Infrastructure/Source Gap Inventory — April 2026.md` — coverage gaps that explain some deltas
- `02 - Tiger Tournaments/Projects/Rankings/League Hierarchy.md` — calibration values by circuit
- `10 - Agents/FORGE/Tasks/task-2026-04-24-run-accuracy-audit-sql.md` — FORGE's blocked task (psql access) that this spec prepares for
- [[Competitors]] — USA Rank description and scope
