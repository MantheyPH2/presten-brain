---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: high
due: 2026-05-07
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Brier Score Completion.md"
---

# Task: Brier Score Completion

## Context

ELO's DSS Ranking Accuracy Claims spec (`Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md`) explicitly lists "Brier Score Completion.md not found in vault" as a key finding. The spec's predictive accuracy claim was softened as a result — ELO used U13/U14 pre-fix Brier values (0.312, 0.273) as context only, not as positive claims, and marked the Brier-dependent accuracy claim as NOT authorized until this file exists.

DSS is May 16. The Brier score analysis is the most concrete predictive accuracy evidence Evo Draw has. Writing this file converts a softened claim into an authorized one. SENTINEL cannot authorize ELO's predictive accuracy talking point for DSS without it.

ELO knows the base values from Calibration Validation 2026-04-22. The document needs to be written, not re-derived.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Brier Score Completion.md`

---

### Section 1: What Brier Score Measures Here

One paragraph. Brier Score measures the accuracy of probabilistic win predictions: lower is better, 0.25 is the random-guess baseline, 0.0 is perfect. In Evo Draw's context: before a game, the ELO model assigns win probabilities to each team based on their ratings. Brier Score measures how well those probabilities matched actual outcomes across the test set.

State: what game set was used, what time window, and what counts as the "test set" (holdout games vs. in-sample games).

---

### Section 2: Pre-Fix Brier Scores by Age Group

Pull from Calibration Validation 2026-04-22. Known values:

| Age Group | Brier Score | vs. Baseline (0.25) | Interpretation |
|-----------|-------------|---------------------|----------------|
| U13G | 0.312 | Worse than random | [ELO explains why — likely GA overcalibration effect] |
| U14G | 0.273 | Near baseline | [ELO notes] |
| U15G | [value from validation doc] | | |
| U16G | [value from validation doc] | | |
| U17G | [value from validation doc] | | |
| U13B | [value if available] | | |
| U14B | [value if available] | | |

Fill all cells from the Calibration Validation 2026-04-22 data. If a value is not available, write `[not computed]` with a one-line explanation of why.

---

### Section 3: Why Pre-Fix Scores Were Elevated (U13G / U14G)

ELO explains the mechanism: GA ASPIRE overcalibration inflated Girls ratings in GA-dominant age groups. This produced overconfident win predictions for Girls GA teams against Girls ECNL teams, which failed when they played. The Brier penalty is the statistical signature of that miscalibration.

This section is important for the DSS claim: the pre-fix scores document the **problem**. The post-fix improvement documents the **fix working**.

---

### Section 4: Post-Fix Brier Score Projection

This section is pre-computation (written before April 28 results are available). ELO projects expected post-fix Brier improvements based on the Pre-Run Model.

| Age Group | Pre-Fix Score | Projected Post-Fix Score | Reasoning |
|-----------|--------------|--------------------------|-----------|
| U13G | 0.312 | [ELO projection] | GA ASPIRE correction removes overcalibration |
| U14G | 0.273 | [ELO projection] | |

Mark clearly: `[Projection — pre-April-28 computation. Update after April 29 analysis runs.]`

After April 29 recompute, ELO updates this section with actual post-fix scores (if Brier re-computation is run as part of the April 29 session).

---

### Section 5: DSS Accuracy Claim — Authorized Language

One paragraph, ELO-authorized, Presten can speak verbatim:

> "Evo Draw's ratings predict game outcomes with a Brier Score of [best age group score] in the [age group] division, compared to a 0.25 baseline for a random predictor. We identified and corrected a calibration error affecting [Girls GA ASPIRE age groups], which we expect to further improve predictive accuracy in those cohorts. Full Brier validation results are available on request."

ELO fills in the bracketed values. If post-fix scores are not yet available at the time of writing, use the best pre-fix score from a well-calibrated age group and note the GA ASPIRE groups as "recently corrected."

**Note:** This language must be consistent with Section 5 of the DSS Accuracy Claims spec (claims authorized and not authorized). If the DSS Claims spec does not authorize this claim yet, ELO notes that here rather than writing unsupported language.

---

### Section 6: Computation Notes

Brief technical notes:
- Test set size: [N games, N teams]
- Brier computation method: [formula or reference to compute-rankings.js]
- Holdout methodology: [was this out-of-sample or in-sample?]
- Known limitations: [e.g., Boys Brier may not be available until after Boys Option A]

---

## Definition of Done

- File exists at `Rankings/Brier Score Completion.md`
- All known age group Brier scores are filled in (pre-fix) with values from Calibration Validation 2026-04-22
- Scores missing from that data are labeled `[not computed]` with reason
- Projection section is clearly labeled as pre-computation
- DSS accuracy claim language is present and ELO-authorized
- After filing: ELO updates the DSS Accuracy Claims spec to mark the Brier claim as authorized (or documents why it remains not authorized)
- Summary in briefing: "Brier Score Completion filed. Pre-fix scores: [list best scores]. DSS predictive accuracy claim: [authorized / pending post-fix computation]."

## Why This Matters

The DSS accuracy claims spec currently cannot authorize ELO's strongest predictive claim because the supporting document is missing. This is the only instance in the entire DSS preparation stack where a claim is blocked not by analysis that hasn't been done, but by a document that hasn't been written. ELO knows the values. Writing this document is a documentation task, not an analysis task. The cost is low; the DSS claim upgrade is high.

## References

- `Rankings/Calibration Validation — 2026-04-22.md` — source data for pre-fix Brier scores
- `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md` — spec that requires this file to authorize a claim
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — projection methodology that informs Section 4
- `Rankings/Post-April-28 Rating Shift Analysis Spec.md` — April 29 analysis that will provide post-fix Brier data
