---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: open
priority: medium
due: 2026-05-07
deliverable: "02 - Tiger Tournaments/Projects/Rankings/DSS Demo — Ranking Accuracy Evidence Package.md"
---

# Task: DSS Demo — Ranking Accuracy Evidence Package

## Context

The DSS demo is May 9. Presten will make accuracy claims about Evo Draw's rankings to potential stakeholders. ELO has produced a DSS competitive accuracy narrative, talking points, and a demo script. What does not exist: a consolidated, SENTINEL-reviewed evidence package that backs those accuracy claims with specific numbers from ELO's own validation work.

If someone at DSS asks "how accurate are your rankings?" Presten needs to be able to say a specific, defensible number — not a vague claim. That number must come from ELO's Brier score analysis, USARank comparison, and calibration validation work. Those results exist in multiple separate documents across the vault. ELO needs to compile them into one authoritative evidence package that SENTINEL reviews before DSS.

This also serves as ELO's quality certification: by producing this document, ELO is asserting that the stated accuracy claims are supported by the data.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/DSS Demo — Ranking Accuracy Evidence Package.md`

---

### Section 1: Accuracy Claims Summary (DSS-Ready Language)

State each accuracy claim in plain language that Presten can use at DSS. Maximum 5 claims. Each claim must be backed by a specific evidence entry in Section 2.

| # | Claim (Plain Language) | Metric | Evidence Reference |
|---|----------------------|--------|-------------------|
| 1 | | | Section 2.X |
| 2 | | | Section 2.X |
| 3 | | | Section 2.X |
| 4 | | | Section 2.X |
| 5 | | | Section 2.X |

Examples of well-formed claims:
- "Evo Draw correctly predicts match outcomes at X% accuracy" (backed by Brier score)
- "Evo Draw rankings align with [competitor] rankings for top-N teams at X% agreement rate"
- "After calibration, rating inflation is within Y points across the [cohort] pool"

Claims ELO cannot make: claims about data FORGE has not yet ingested, claims about age groups where calibration is flagged as incomplete.

---

### Section 2: Evidence Entries

**2.1 — Brier Score Results**

| Cohort (age/gender) | Sample Size | Brier Score | Interpretation | Data Source |
|---------------------|-------------|-------------|----------------|-------------|
| | | | | |

Brier score reference: 0.25 = chance-level prediction; 0.20 = modest skill; ≤ 0.15 = strong predictive power.

Status: _[complete / incomplete — reason]_
Source document: `task-2026-04-25-brier-score-completion.md`

**2.2 — USARank Comparison**

| Age Group | Gender | Evo Draw Rank Agreement (top 50) | Avg Rank Delta | Data Date |
|-----------|--------|----------------------------------|---------------|-----------|
| | | | | |

Source document: `task-2026-04-25-usarank-delta-interpretation-spec.md` and `task-2026-04-26-post-april28-usarank-comparison-execution.md`

**2.3 — Calibration Improvement Evidence**

Before/after the girls calibration work:

| Metric | Pre-Calibration | Post-April-28 | Improvement |
|--------|----------------|---------------|-------------|
| GA ASPIRE avg rating error | | | |
| Girls U13 median rank delta vs. USARank | | | |
| Boys Option A: rating std dev change | | _[pending April 29]_ | |

**2.4 — Coverage-Adjusted Accuracy**

Accuracy claims are only valid for leagues with sufficient game coverage. ELO documents which cohorts are coverage-limited:

| Cohort | Coverage Status | Accuracy Claim Valid? |
|--------|----------------|----------------------|
| Girls ECNL | Good | YES |
| Boys GA | Improving | CONDITIONAL — note in demo |
| _[other]_ | | |

---

### Section 3: Known Limitations (Do Not Claim)

ELO documents what Evo Draw CANNOT accurately claim at DSS May 9, and why:

| Limitation | Reason | Timeline to Resolve |
|------------|--------|---------------------|
| | | |

This section protects Presten from overstating accuracy in the demo. If challenged on a limitation, Presten can reference the timeline.

---

### Section 4: SENTINEL Review Checklist

Before SENTINEL approves this evidence package:

- [ ] All evidence entries in Section 2 reference a real, filed document (not a placeholder)
- [ ] No accuracy claim is stronger than the weakest supporting data point
- [ ] Known limitations include all calibration-incomplete cohorts
- [ ] Section 1 claims have been reviewed by ELO for defensibility under technical questioning
- [ ] Boys data: claims are conditional on April 29 Option A results (note if Option A pending)

SENTINEL approval status: _PENDING_
SENTINEL review date: _[fill when reviewed]_

---

## Definition of Done

- Document filed at deliverable path by May 7 (gives SENTINEL 2 days to review before May 9)
- All 5 claims in Section 1 are filled with specific numbers
- Section 2 evidence entries reference existing documents (not placeholders)
- Section 3 limitations are honest and complete
- SENTINEL has reviewed and approved before May 9
- ELO briefing May 7 states: "DSS accuracy evidence package ready for SENTINEL review"

## Why This Matters

Evo Draw's credibility at DSS depends on accuracy claims being defensible. Vague claims ("our rankings are accurate") invite skepticism. Specific claims ("Brier score of 0.18 on a 3,200-game test set") invite engagement and demonstrate rigor. This document also protects against SENTINEL authorizing the demo with claims ELO hasn't actually verified — every claim in Section 1 must pass Section 4 before SENTINEL signs off.

## References

- `task-2026-04-25-brier-score-completion.md` — Brier score analysis
- `task-2026-04-25-usarank-delta-interpretation-spec.md` — USARank comparison methodology
- `task-2026-04-26-post-april28-usarank-comparison-execution.md` — comparison execution
- `task-2026-04-27-dss-competitive-accuracy-narrative.md` — existing narrative (input to Section 1)
- `task-2026-04-27-dss-demo-talking-points-rankings.md` — talking points (complement this doc)
- `task-2026-04-28-pre-may9-calibration-state-dashboard.md` — calibration state input
