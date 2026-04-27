---
title: Post-May-1 Rankings Impact — Pipeline Baseline
type: pipeline-baseline
author: ELO
date: 2026-04-27
status: template — complete on May 4 after May 1, 2, 3 pipeline runs
related: "[[Post-Fix Calibration Stability Monitoring Spec]]", "[[Post-April-28 Expected Rating Shift — Pre-Run Model]]"
tags: [pipeline, rankings, baseline, may-1, evo-draw]
---

# Post-May-1 Rankings Impact — Pipeline Baseline

> **[Template — ELO completes this document on May 4, after three full daily pipeline runs (May 1, 2, 3). Do not fill result fields before all three runs are confirmed complete by FORGE.]**

---

## Pre-Checklist: Required Inputs

Complete before running any analysis:

```
Pre-May-1 ratings baseline snapshot location: Rankings/Pre-April-28 Baseline Snapshot.md (or updated snapshot)
Date of baseline snapshot: ___________
May 1 pipeline run confirmed complete (from FORGE validation checklist): YES / NO
May 2 pipeline run confirmed complete: YES / NO
May 3 pipeline run confirmed complete: YES / NO
```

**If any pipeline run did not complete cleanly:** Note the specific run and its impact on this analysis. Partial game data may distort rating changes. State which age groups or regions are affected and adjust the Section 6 verdict accordingly.

---

## Section 1: Coverage Change Summary

| Metric | Pre-May-1 | Post-May-3 | Delta |
|--------|-----------|-----------|-------|
| Total teams with ratings | | | |
| Total games in DB | | | |
| Teams with ratings updated (new games) | N/A | | — |
| New teams added (first ratings) | N/A | | — |

**Note:** Age groups with 0 updated teams may indicate org-ID coverage gaps — flag these for SENTINEL in the next briefing.

---

## Section 2: Rating Movement Summary

For all teams with ≥ 1 new game ingested across May 1–3:

| Metric | Value |
|--------|-------|
| Teams with rating change | |
| Average absolute rating change (pts) | |
| Max rating increase (pts) | |
| Max rating decrease (pts) | |
| Teams with rating change > 20 pts | |
| Teams with rating change > 50 pts | |

---

## Section 3: Rating Movement by Age Group and Gender

| Age Group | Gender | Teams Updated | Avg Rating Change (pts) | Any Outliers (> 50 pts)? |
|-----------|--------|--------------|------------------------|--------------------------|
| U13 | Girls | | | |
| U13 | Boys | | | |
| U14 | Girls | | | |
| U14 | Boys | | | |
| U15 | Girls | | | |
| U15 | Boys | | | |
| U16 | Girls | | | |
| U16 | Boys | | | |
| U17 | Girls | | | |
| U17 | Boys | | | |
| U18/U19 | Girls | | | |
| U18/U19 | Boys | | | |

Age groups showing 0 updated teams are findings, not gaps in the document. Flag for SENTINEL with interpretation (coverage gap vs. low game volume for that age group in this period).

---

## Section 4: Outlier Review

For every team with a rating change > 50 points (either direction) during May 1–3:

| Team ID | Team Name | Age/Gender | Pre-May-1 Rating | Post-May-3 Rating | Change | Game Count (New) | Likely Cause |
|---------|-----------|-----------|-----------------|------------------|--------|-----------------|-------------|
| | | | | | | | |

**Likely Cause options:** Many new games ingested (catch-up effect) / Strong opponent results / Calibration-sensitive tier (ECNL/MLS NEXT) / Data quality issue (check game records)

Any outlier assessed as a data quality issue must be flagged in the ELO briefing filed May 4.

---

## Section 5: Pipeline Freshness Narrative

*(5–8 sentences, plain language. Usable directly in DSS presentation or Presten briefing. ELO fills May 4.)*

> "Since launching the daily pipeline on May 1, Evo Draw has ingested [N] new games, updated ratings for [N] teams across [N] age groups and [N] genders. The average rating change across updated teams was [N] points — consistent with [description of expected range for a fresh batch of recent-season games]. The teams with the largest rating improvements were [brief description, e.g., 'U16 Girls teams with strong recent ECNL results']. No anomalies were detected in the first 3 runs."

[Adjust final sentence if anomalies were detected — state what they were and what ELO did about them.]

---

## Section 6: First-Week Verdict

ELO selects one verdict and completes the associated statement:

**NOMINAL**
> Rating movements are within expected ranges for new game ingestion. No outliers indicating data quality issues. Pipeline is producing correct ELO updates. Recommend continuing standard monitoring per the Post-Fix Calibration Stability Monitoring Spec.

**MONITORING FLAG**
> 1–2 anomalies detected: [describe]. ELO is tracking. No calibration change required. Flagged in this briefing.

**ESCALATION**
> Systematic anomaly detected: [describe]. Do not wait for May 4 — flag to SENTINEL immediately. [This branch should have been filed before May 4 if condition occurred.]

**ELO verdict:** [NOMINAL / MONITORING FLAG / ESCALATION — ELO fills May 4]

---

## References

- `Infrastructure/May 1 First-Run Validation Checklist.md` — FORGE's run confirmation (prerequisite for this analysis)
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — stability monitoring framework this feeds into
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — baseline context for expected rating ranges
- `Rankings/Pre-April-28 Baseline Snapshot.md` — pre-pipeline ratings reference

*ELO — 2026-04-27 (template pre-filed)*
