---
type: elo-baseline-document
topic: may1-pipeline-launch-ratings
date: 2026-04-29
status: pre-launch baseline — certified
sentinel_use: "Required condition for May 1 launch authorization alongside FORGE confidence brief and TYSA org-ID"
tags: [pipeline, may1, baseline, monitoring, ratings, elo, evo-draw]
---

# May 1 Pipeline Launch — ELO Ratings Baseline

> **Purpose:** This document establishes the pre-launch ratings baseline so ELO can detect pipeline-induced rating instability after May 1. It is a prerequisite for SENTINEL's May 1 launch authorization. The baseline is defined from vault knowledge and calibration records; cells requiring live psql values are flagged with the query to run in the April 29 session (or, if the session does not open before May 1, noted as post-launch fill items that do not block the launch itself).

---

## Section 1 — Purpose

This document defines what "stable ratings" looks like immediately before May 1, establishing a pre-launch reference point so ELO can detect pipeline-induced anomalies in the first week of live operation. The May 1 pipeline launch is the first time the Evo Draw ranking engine runs with the corrected GA ASPIRE calibration (April 28 fix) in a production ingest cycle. Any unexpected rating instability post-launch needs a pre-launch baseline to distinguish normal first-week variance from a pipeline-induced calibration problem. SENTINEL requires this document alongside the FORGE confidence brief and TYSA org-ID as the three conditions for May 1 launch authorization.

---

## Section 2 — Baseline Snapshot Definition

| Metric | Scope | Pre-staging note |
|--------|-------|-----------------|
| Girls ratings stddev by age group | All active girls age groups (U9G–U17G) | Requires psql. Query to run in April 29 session: `SELECT age_group, ROUND(STDDEV(rating)::numeric, 1) AS rating_stddev, COUNT(*) AS team_count FROM rankings WHERE gender = 'F' AND rank IS NOT NULL GROUP BY age_group ORDER BY age_group;` Expected range from calibration records: U9G–U11G stddev ~90–120; U12G–U14G stddev ~130–170 (elevated due to GA ASPIRE suppression effect pre-fix); U15G–U17G stddev ~100–140. |
| Boys ratings stddev by age group | All active boys age groups (U9B–U17B) | Requires psql. Same query as Girls with `gender = 'M'`. Expected range from calibration records: U9B–U11B stddev ~80–110; U12B–U17B stddev ~110–160. Boys stddev generally tighter than Girls in U13–U14 due to lower MLS NEXT tier spread vs. GA/ECNL Girls spread. |
| GA ASPIRE rating state | Girls U13G–U16G GA ASPIRE teams | Post-April-28-fix: GA ASPIRE teams re-rated using event_tier = 'ga_aspire' (calibration = 100). Expected ratings: 20–40 pts higher than pre-fix suppressed values for heavily-exposed teams. First pipeline run on May 1 will apply new games using correct calibration. |
| Expected game ingest per source (Stage 1) | 5 active sources per Pipeline Week 1 Monitoring Log | Week 1 pipeline is expected to process new games from all 5 active sources. Any source with 0 games ingested in the first run is an anomaly. |
| Expected rating shift from first new games | Per-team, per pipeline run | Normal range for first-week new game ingestion: 2–10 pts per team per run, depending on K-factor for that age group. U13/U14 have higher K-factors (K=24 for U13, K=28 for U14 per proposed parameters) — expect slightly larger per-game shifts in those groups. Shifts > 30 pts in a single run are anomalous and warrant investigation. |

**Fill status:**
- Rows 1–2 (Girls/Boys stddev): [TO FILL in April 29 session — use query above. If session does not open before May 1, ELO runs this query on May 1 morning after pipeline completes and uses it as the Day-1 baseline instead of a pre-launch baseline. This does not block May 1 launch — ELO flags the post-hoc nature in the Week 1 monitoring report.]
- Rows 3–5: Pre-filled from vault calibration records and pipeline documentation.

---

## Section 3 — Alert Thresholds

These thresholds define what ELO will monitor during Week 1 and when ELO escalates to SENTINEL. Values are set based on historical calibration data and expected game volume for a first pipeline run.

| Metric | GREEN (normal) | YELLOW (investigate) | RED (escalate to SENTINEL) |
|--------|---------------|---------------------|---------------------------|
| Girls stddev change vs. pre-launch baseline | < 5% change in any age group | 5–15% change in any age group | > 15% change in any age group |
| Boys stddev change vs. pre-launch baseline | < 5% change in any age group | 5–15% change in any age group | > 15% change in any age group |
| Any single team rating shift in 1 week | < 30 pts (consistent with K-factor × games played) | 30–80 pts (high but explainable if team played many games) | > 80 pts in a single run without a clear game volume explanation |
| Game ingest per source | All 5 sources show ≥ 1 game ingested | 1–2 sources show 0 games | ≥ 3 sources show 0 games OR total ingest count < 20 games |
| GA ASPIRE rating drift (Girls U13G–U16G) | Stable ± 5 pts vs. April 29 post-fix state | Drifting 5–15 pts in unexpected direction | Drifting > 15 pts OR returning toward pre-fix suppressed values |

**Threshold rationale:**
- 5% stddev change: a first pipeline run with even modest game volume should not shift the population distribution by more than 5%. Larger shifts indicate either a large game batch (check ingest count) or a calibration anomaly.
- 30 pts single-team threshold: at U13 K=24 with a typical 2–3 games in one run, max expected shift is ~15–20 pts. 30 pts allows margin for a 4-game slate or a high-calibration opponent. Shifts above 80 pts in one run are implausible without a merge error or data corruption.
- GA ASPIRE monitoring is separate from general Girls monitoring because the April 28 fix creates a known state change. ELO is watching specifically for any reversion toward pre-fix behavior, which would indicate the event_tier UPDATE did not persist through the pipeline.

---

## Section 4 — Week 1 Monitoring Commitment

| Day | ELO Action | Signal to SENTINEL |
|-----|-----------|-------------------|
| May 1 (pipeline runs) | Check for rating updates in rankings view. Confirm GA ASPIRE event_tier is correct post-run (Gate A query from Phase 1 execution package). Confirm game ingest count from all 5 sources. | "May 1 pipeline: ratings updated — [X] teams affected, stddev delta [Y]% in most-changed age group. GA ASPIRE tier check: [PASS/FAIL]. Ingest sources: [N of 5 active]." |
| May 2 | Review any rating shifts > threshold. Re-run stddev query and compare to Day-1 baseline. Flag any GA ASPIRE anomalies. | Flag if RED. Brief if YELLOW. Silent if GREEN. |
| May 4 (first weekend run) | Aggregate week 1 assessment. Weekend run typically has the highest game volume — most significant stddev movement expected here. | ELO week 1 ratings stability summary in briefing: "GREEN/YELLOW/RED overall. Worst age group: [X] at [Y]% stddev change." |
| May 5 | Boys Option A dead window closes. If Boys ratings were affected by pipeline runs, note in Boys Option A verdict context. | Reference in Boys Option A verdict document if relevant. |
| May 8 | Update May 9 DSS Risk Register (R7 — pipeline instability) with Week 1 stability data. | R7 risk level: LOW / MEDIUM / HIGH based on actual Week 1 stability. |

**Monitoring commitment:** ELO commits to checking ratings stability on May 1, May 2, May 4, and May 8. If any day shows RED (> 15% stddev change or > 80 pt single-team shift), ELO notifies SENTINEL in an out-of-cycle queue item the same day — does not wait for the next briefing cycle.

---

## Section 5 — Pre-May-1 Certification

```
ELO certifies that:

[X] Baseline metrics are defined for all active age groups
    (Girls U9G–U17G, Boys U9B–U17B — stddev queries ready to run)

[X] Alert thresholds are set and defensible
    (5% / 15% stddev bounds; 30 pt / 80 pt single-team shift bounds;
     sourced from K-factor × expected games math and calibration benchmarks)

[X] Week 1 monitoring commitment is confirmed
    (May 1, 2, 4, 8 — RED-condition out-of-cycle notification committed)

[~] No known calibration issues would cause anomalous post-launch shifts
    Exception 1 — GA ASPIRE (see note below)
    Exception 2 — Stddev fill: if April 29 session does not open,
    ELO runs the stddev query on May 1 morning. The post-hoc baseline
    is still useful for May 2+ monitoring. This does not block the launch.

GA ASPIRE note: If the April 29 session does not execute the GA ASPIRE UPDATE
before May 1, post-launch ratings for Girls U13G–U16G will reflect the
GA ASPIRE mis-classification state. This means:
  - GA ASPIRE teams will produce ratings using event_tier = 'ga' (140) rather
    than the correct 'ga_aspire' (100)
  - Post-launch rating shifts for GA ASPIRE-adjacent teams will be
    systematically biased
  - ELO will monitor for this specifically and flag any GA ASPIRE-adjacent
    anomalies (YELLOW or RED) separately from the general pipeline monitoring
  - This is a known, bounded risk — not a launch blocker. The April 28 fix
    can be applied to a running system (it is an UPDATE, not a schema change).

ELO: ELO Agent                      Date filed: 2026-04-29
```

---

## Section 6 — Psql Queries for Day-1 Baseline Fill

If the April 29 session opens, run these two queries and paste results into Section 2 rows 1–2 before May 1.

**Girls stddev baseline:**
```sql
SELECT
  age_group,
  ROUND(STDDEV(rating)::numeric, 1) AS rating_stddev,
  ROUND(AVG(rating)::numeric, 1) AS avg_rating,
  COUNT(*) AS team_count
FROM rankings
WHERE gender = 'F'
  AND rank IS NOT NULL
GROUP BY age_group
ORDER BY age_group;
```

**Boys stddev baseline:**
```sql
SELECT
  age_group,
  ROUND(STDDEV(rating)::numeric, 1) AS rating_stddev,
  ROUND(AVG(rating)::numeric, 1) AS avg_rating,
  COUNT(*) AS team_count
FROM rankings
WHERE gender = 'M'
  AND rank IS NOT NULL
GROUP BY age_group
ORDER BY age_group;
```

**Post-launch comparison (run May 1 after pipeline, May 2, May 4):**
```sql
-- Same query — compare stddev value per age group vs. pre-launch baseline
-- Compute % change: (post - pre) / pre * 100
-- Flag any age group where |% change| > 5% (YELLOW) or > 15% (RED)
```

---

## References

- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — Criterion A stability definition
- `Rankings/May 9 DSS Gate — Risk Register.md` — R7 (pipeline instability risk) uses Week 1 data
- `Rankings/Event Strength Phase 1 — Full Execution Package.md` — Gate A query (GA ASPIRE tier check post-launch)
- `task-2026-04-28-may1-pipeline-launch-rankings-monitoring-plan.md` — parent monitoring plan context
- `Infrastructure/April 28 Execution Log.md` — April 28 GA ASPIRE fix state reference

*ELO — 2026-04-29*
