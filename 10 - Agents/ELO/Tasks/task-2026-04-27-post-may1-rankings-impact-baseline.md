---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: open
priority: medium
due: 2026-05-04
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Post-May-1 Rankings Impact — Pipeline Baseline.md"
---

# Task: Post-May-1 Rankings Impact — Pipeline Baseline

## Context

May 1 is the daily pipeline launch date. For the first time, Evo Draw will ingest games on an automated daily cadence rather than through manual imports or one-off pipeline runs.

This is a significant product milestone. The first week of daily pipeline data will produce a measurable change in team ratings — new games flow in, ELO updates run, rankings shift. SENTINEL needs to know: how much did the rankings move during the first week of live pipeline operation, for which teams, and is that movement in the expected direction?

Without this baseline, SENTINEL cannot:
1. Confirm the pipeline is producing correct ratings updates (not just game ingestion)
2. Quantify the "freshness improvement" for DSS: "since launching daily pipeline, X teams have had their ratings updated, rankings have moved by an average of Y points"
3. Distinguish normal pipeline-driven rating change from anomalies caused by data quality issues

ELO's May 4 deliverable: a snapshot comparison of ratings before May 1 vs. after the first 3 daily pipeline runs (May 1, 2, 3), with a summary of what moved and why.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Post-May-1 Rankings Impact — Pipeline Baseline.md`

Complete this document on May 4 (after 3 full daily runs: May 1, 2, 3).

---

### Pre-Checklist: Required Inputs

Before running any analysis, confirm:

```
Pre-May-1 ratings baseline snapshot location: ___________
Date of baseline snapshot: ___________
May 1 pipeline run confirmed complete (from FORGE validation checklist): YES / NO
May 2 pipeline run confirmed complete: YES / NO
May 3 pipeline run confirmed complete: YES / NO
```

If any pipeline run did not complete cleanly, note this and describe the impact on the analysis (partial game data may distort rating changes).

---

### Section 1: Coverage Change Summary

How many teams have ratings in the DB before vs. after the first 3 runs?

| Metric | Pre-May-1 | Post-May-3 | Delta |
|--------|-----------|-----------|-------|
| Total teams with ratings | | | |
| Total games in DB | | | |
| Teams with ratings updated (new games) | N/A | | — |
| New teams added (first ratings) | N/A | | — |

---

### Section 2: Rating Movement Summary

For all teams that had at least 1 new game ingested across May 1–3:

| Metric | Value |
|--------|-------|
| Teams with rating change | |
| Average absolute rating change (pts) | |
| Max rating increase (pts) | |
| Max rating decrease (pts) | |
| Teams with rating change > 20 pts | |
| Teams with rating change > 50 pts | |

---

### Section 3: Rating Movement by Age Group and Gender

Break down Section 2 by age group and gender:

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

Age groups with 0 teams updated may indicate org-ID coverage gaps for that age group — note this for SENTINEL.

---

### Section 4: Outlier Review

For every team with a rating change > 50 points (in either direction) during May 1–3:

| Team ID | Team Name | Age/Gender | Pre-May-1 Rating | Post-May-3 Rating | Change | Game Count (New) | Likely Cause |
|---------|-----------|-----------|-----------------|------------------|--------|-----------------|-------------|
| | | | | | | | |

"Likely Cause" options: Many new games ingested (catch-up effect), strong opponent results, calibration-sensitive tier (ECNL/MLS NEXT), data quality issue (check game records).

Any outlier that appears to be a data quality issue must be flagged in the following ELO briefing.

---

### Section 5: Pipeline Freshness Narrative

One paragraph (5–8 sentences) summarizing what the first 3 days of daily pipeline operation produced. This paragraph should be usable directly in a DSS presentation or Presten briefing:

> "Since launching the daily pipeline on May 1, Evo Draw has ingested [N] new games, updated ratings for [N] teams across [N] age groups and [N] genders. The average rating change across updated teams was [N] points — consistent with [description of what's expected for a fresh batch of recent-season games]. The teams with the largest rating improvements were [brief description, e.g., 'U16 Girls teams with strong recent ECNL results']. No anomalies were detected in the first 3 runs."

Adjust the final sentence if anomalies were detected — state what they were and what ELO did.

---

### Section 6: First-Week Verdict

One of three verdicts:

**NOMINAL** — Rating movements are within expected ranges for new game ingestion. No outliers indicating data quality issues. Pipeline is producing correct ELO updates. Recommend continuing standard monitoring per the Post-Fix Calibration Stability Monitoring Spec.

**MONITORING FLAG** — 1–2 anomalies detected (outlier teams, coverage gaps for specific age groups). ELO is tracking. No calibration change required. Flag in next briefing.

**ESCALATION** — Systematic anomaly detected (e.g., entire age group showing 0 updates, multiple teams with implausible rating swings, game records appearing corrupted). Flag to SENTINEL immediately. Do not wait for May 4.

---

## Quality Standard

- The pre-checklist must confirm all 3 pipeline runs completed before the analysis begins. Do not analyze partial data.
- Section 3 must include all age groups. A row showing 0 updated teams is not a gap in the document — it is a finding.
- The pipeline freshness narrative (Section 5) must be written in plain language. It will be used as-is in reporting.
- If any May 1–3 pipeline run did not complete (FORGE validation shows RED or YELLOW), note the specific impact on this analysis and adjust the verdict accordingly.

## Why This Matters

SENTINEL needs two things from the May 1 pipeline launch: (1) confirmation it ran, and (2) confirmation it's producing correct rating updates. FORGE's first-run validation checklist covers (1). This document covers (2). Without it, SENTINEL can tell Presten "the pipeline ran" but not "the pipeline is making rankings better." The pipeline freshness narrative in Section 5 is also the clearest data-driven statement Evo Draw has for DSS: not "we have a daily pipeline" (architectural claim) but "since May 1, N teams have fresher, more accurate ratings" (outcome claim). That is a meaningfully stronger competitive statement.

## References

- `Infrastructure/May 1 First-Run Validation Checklist.md` — FORGE's run confirmation (prerequisite for this analysis)
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — stability monitoring framework this feeds into
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — baseline context for expected rating ranges
- `Rankings/Pre-April-28 Baseline Snapshot.md` — pre-pipeline ratings reference
