---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: medium
due: 2026-05-02
deliverable: "02 - Tiger Tournaments/Projects/Rankings/USARank Comparison — Post-April-28 Results.md"
---

# Task: USARank Comparison — Post-April-28 Results Template

## Context

The USARank Post-April-28 Comparison Execution Package (filed Apr 25) defines how ELO runs the comparison once April 28 fixes are live. The USARank Delta Interpretation Spec (filed Apr 25) defines how to interpret the results. What does not exist: the results document itself — the structured template ELO fills in when the comparison is actually run.

Without a pre-built results template, the comparison produces raw SQL output that ELO narrates in a briefing. That is harder for SENTINEL and Presten to review and harder to compare against a future run. A structured results template converts the one-time comparison into a repeatable benchmark with a consistent format.

This document is designed to be filled once (after April 29), then updated after each subsequent major calibration change. It becomes Evo Draw's standing accuracy benchmark against the primary competitor.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/USARank Comparison — Post-April-28 Results.md`

---

### Section 0 — Comparison Metadata

| Field | Value |
|-------|-------|
| Comparison run date | [FILL: date ELO runs the queries] |
| Evo Draw snapshot date | [FILL: last pipeline run date] |
| USARank data date | [FILL: date of USARank reference data] |
| ELO operator | ELO |
| Comparison scope | Girls U13–U19, GotSport-sourced teams |
| Prior comparison baseline | None (this is the first structured comparison) |

---

### Section 1 — Top-50 Rank Correlation

ELO runs the comparison query from the Execution Package against the top 50 teams in each age group. For each age group:

| Age Group | Kendall's τ or Spearman ρ | Evo Draw median rank | USARank median rank | Delta |
|-----------|--------------------------|----------------------|---------------------|-------|
| Girls U13 | [FILL] | [FILL] | [FILL] | [FILL] |
| Girls U14 | | | | |
| Girls U15 | | | | |
| Girls U16 | | | | |
| Girls U17 | | | | |
| Girls U18/U19 | | | | |

**Interpretation threshold (from Delta Interpretation Spec):**
- τ ≥ 0.80: strong correlation — Evo Draw and USARank substantively agree
- τ 0.60–0.79: moderate correlation — note specific divergences
- τ < 0.60: weak correlation — flag to SENTINEL; investigate cause before DSS

ELO fills the threshold verdict for each age group: **Strong / Moderate / Weak / [FILL]**

---

### Section 2 — Notable Divergences (Top 20)

For each age group with τ < 0.80: list the top 5 teams where Evo Draw and USARank diverge most.

| Age Group | Team Name | Evo Draw Rank | USARank Rank | Delta | Likely cause |
|-----------|-----------|--------------|-------------|-------|-------------|
| Girls U15 | [FILL] | [FILL] | [FILL] | [FILL] | [FILL] |

Likely cause categories:
- **Coverage gap:** USARank has games Evo Draw doesn't (or vice versa)
- **Calibration difference:** same games, different weights
- **Recency difference:** one system weights recent games more heavily
- **Merge issue:** Evo Draw has a team split or merged that USARank treats differently
- **Unknown:** ELO cannot determine from data alone

If Merge issue appears: flag team name to SENTINEL for team_merges audit.

---

### Section 3 — GA ASPIRE Fix Impact

Specific comparison of Girls GA ASPIRE teams before and after April 28 fix. Purpose: confirm the fix improved Evo Draw accuracy vs. USARank for GA-based teams.

ELO queries the comparison specifically for teams with a Georgia zip code or `ga_aspire` classifier, and compares:

| Metric | Pre-April-28 (estimated from Pre-Run Model) | Post-April-28 (actual) | Delta |
|--------|---------------------------------------------|------------------------|-------|
| Girls GA avg rank vs. USARank | [from Pre-Run Model] | [FILL] | [FILL] |
| Girls GA ASPIRE teams in top-50 | [from Pre-Run Model] | [FILL] | [FILL] |
| τ for GA-heavy age groups | [from Pre-Run Model] | [FILL] | [FILL] |

**Verdict:** Did the GA ASPIRE fix measurably improve Evo Draw's correlation with USARank for GA teams? YES / NO / INCONCLUSIVE `[FILL]`

---

### Section 4 — Summary Assessment

One-paragraph summary for SENTINEL and Presten. Structure:

> Overall Evo Draw / USARank correlation: [Strong/Moderate/Weak] across [N] of 6 age groups. Best correlation: [age group] (τ = [X]). Weakest: [age group] (τ = [X]). GA ASPIRE fix: [improved / did not improve / inconclusive] correlation for GA-heavy age groups. [N] teams flagged for merge audit. DSS ranking accuracy claim: [supported / partially supported / not yet supported] by this comparison.

---

### Section 5 — Next Comparison Schedule

| Date | Trigger | Comparison scope |
|------|---------|-----------------|
| May 9 | Pre-DSS readiness | Girls all age groups + Boys if Option A live |
| Post-DSS | Presten request | Full comparison including Boys |

ELO files the May 9 comparison as a new version of this document (`USARank Comparison — May 9 Results.md`) following the same template, enabling direct delta comparison vs. April 29 results.

---

## Definition of Done

- All 5 sections present (Sections 1–5)
- Section 0 metadata fields are defined; actual values left blank for fill-in on comparison run date
- Section 1 table has all 6 age groups pre-populated with row structure; values blank
- Section 2 table structure defined with cause categories
- Section 3 references the Pre-Run Model as the pre-April-28 baseline (not a blank — pull those numbers now from the Pre-Run Model document)
- Section 4 summary paragraph structure written with [FILL] brackets
- Section 5 schedule rows defined

Note: This task is COMPLETE when the template is written with Pre-Run Model values pre-populated in Section 3. Actual comparison values are filled on April 29 (or within 48 hours after Presten provides USARank reference data).

## Why This Matters

SENTINEL's mandate includes comparing Evo Draw against competitors. This document is the primary instrument for that comparison. Without it, the USARank comparison produces SQL output in a briefing that SENTINEL cannot action or reference. With it, the comparison is a tracked benchmark that SENTINEL reviews in 3 minutes and can cite to Presten as evidence of ranking accuracy improvement. On the DSS timeline, a competitor comparison that cannot be quickly reviewed and cited is a missed opportunity.

## References

- `Rankings/USARank Post-April-28 Comparison Execution Package.md` — the queries and method; this document captures their results
- `Rankings/USARank Delta Interpretation Spec.md` — interpretation thresholds used in Section 1 and Section 4
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — pre-fill source for Section 3 baseline values
- `Rankings/USARank Reference Data Sourcing Spec.md` — how ELO obtains USARank data for the comparison
- `Rankings/Team Merges — High Priority Audit List.md` — destination for merge flags from Section 2
