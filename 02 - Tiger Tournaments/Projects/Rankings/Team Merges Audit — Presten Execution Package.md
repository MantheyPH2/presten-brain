---
title: Team Merges Audit — Presten Execution Package
type: execution-package
author: ELO
date: 2026-04-26
status: ready-for-execution
due: 2026-05-05
related: "[[Merge Audit Triage Framework — May 2026]]", "[[Pre-DSS Team Merge Audit — U13G]]", "[[Rank Bands — Demo Examples Document]]"
tags: [team-merges, audit, execution-package, dss, evo-draw]
---

# Team Merges Audit — Presten Execution Package

> **Purpose:** Self-contained psql execution package for the team merges anomaly audit. Three queries, ~15 minutes. Run each, save outputs to the specified files, send files to ELO. ELO classifies results and files a triage summary in the next briefing.
>
> **Format follows:** `Rankings/Boys Option A Spot Check — Presten Execution Package.md`

---

## Pre-Run Gate

**Confirm before running:** April 28 GA ASPIRE fix applied AND recompute confirmed complete (per `Infrastructure/April 28 Execution Log.md`). Team merges affect ratings — the post-April-28 state is the correct baseline for identifying anomalous merges. Running before the recompute means the anomaly detection compares against a pre-fix rating state.

**Run window:** April 29 – May 5. Must complete before May 9 DSS readiness check. If bad merges affect demo teams (Football Academy NJ, PA Classics, PDA), ELO and FORGE need time to correct before DSS.

---

## Query 1 — High-Confidence Merge Anomaly Detection

**What this finds:** Merged teams where the rating spread across components is implausibly large. A credible merge (same team, different name/source) should have components within ~200 points of each other — a team doesn't change 300+ Elo points by being renamed. Large spreads signal a probable wrong merge.

**Copy-paste ready:**

```sql
-- Query 1: Merge anomaly detection by rating spread
-- Identifies merged teams where component ratings diverge beyond plausible range
SELECT
  t.id                                         AS team_id,
  t.name                                       AS team_name,
  COUNT(tm.id)                                 AS merge_count,
  ROUND(MAX(r_merged.rating)::numeric, 1)      AS max_component_rating,
  ROUND(MIN(r_merged.rating)::numeric, 1)      AS min_component_rating,
  ROUND((MAX(r_merged.rating) - MIN(r_merged.rating))::numeric, 1) AS rating_spread,
  CASE
    WHEN (MAX(r_merged.rating) - MIN(r_merged.rating)) > 300 THEN TRUE
    ELSE FALSE
  END                                          AS anomaly_flag
FROM teams t
JOIN team_merges tm ON tm.primary_team_id = t.id
JOIN rankings r_merged ON r_merged.team_id = tm.merged_team_id
WHERE r_merged.rank IS NOT NULL
GROUP BY t.id, t.name
HAVING COUNT(tm.id) >= 2
ORDER BY rating_spread DESC
LIMIT 500;
```

**Save output to:** `02 - Tiger Tournaments/Projects/Reports/team-merges-query1-[date].md`
_(Replace [date] with today's date, e.g. 2026-04-29)_

**Result thresholds:**

| Count of `anomaly_flag = TRUE` | Action |
|-------------------------------|--------|
| **< 50** | **PASS** — file GREEN in ELO briefing. No blocking action. |
| **50–200** | **YELLOW** — send output to ELO. ELO reviews top 20 by rating_spread and files triage in next briefing. |
| **> 200** | **FAIL** — escalate to SENTINEL. Stop merge audit and await ELO guidance before proceeding. |
| **Any row with anomaly_flag = TRUE AND team rank ≤ 50** | **FAIL regardless of total count** — a top-50 team with a bad merge affects visible rankings. Escalate immediately. |

---

## Query 2 — Demo Team Merge Status Check

**What this finds:** Whether the primary DSS demo teams appear as single, cleanly-merged records. A demo team appearing as two separate rows means the merge hasn't run — the Rank Bands demo may show a fragmented record. A team with `merge_components > 10` may be over-merged (other teams' games incorrectly attributed).

**Copy-paste ready:**

```sql
-- Query 2: Demo team merge status check
-- Confirm primary demo teams are not anomalous merges
SELECT
  t.name                         AS team_name,
  t.id                           AS team_id,
  COUNT(tm.id)                   AS merge_components,
  ROUND(MAX(r.rating)::numeric, 1) AS current_rating,
  r.rank                         AS current_rank,
  r.age_group
FROM teams t
LEFT JOIN team_merges tm ON tm.primary_team_id = t.id
LEFT JOIN rankings r ON r.team_id = t.id
WHERE t.name ILIKE ANY(ARRAY[
  '%football academy%nj%',
  '%football academy nj%',
  '%PA Classics%',
  '%Pennsylvania Classics%',
  '%PDA%',
  '%premier development academy%',
  '%NJ Wildcats%',
  '%NJ Rush%',
  '%FC Stars%',
  '%Cedar Stars%',
  '%Match Fit%',
  '%Eclipse Select%',
  '%Michigan Hawks%',
  '%Real Colorado%',
  '%FC Dallas%',
  '%Sporting Blue Valley%',
  '%Utah Avalanche%'
])
  AND r.age_group IN ('U13G', 'U14G', 'U15G', 'U16G', 'U17G', 'U13', 'U14', 'U15', 'U16', 'U17')
GROUP BY t.name, t.id, r.rank, r.age_group
ORDER BY r.rank NULLS LAST;
```

**Save output to:** `02 - Tiger Tournaments/Projects/Reports/team-merges-query2-[date].md`

**Result thresholds:**

| Condition | Action |
|-----------|--------|
| All named demo teams appear as single records with `merge_components` ≤ 3 | **PASS** |
| A demo team appears as 2 separate rows (not merged) | **FLAG** — run targeted merge SQL from `task-2026-04-25-team-merge-pre-dss-audit.md` for that team. Report to ELO immediately. |
| Any demo team has `merge_components` > 10 | **FLAG** — over-merged. Send to ELO. ELO investigates before Demo Spec is finalized. |
| Football Academy NJ or PA Classics absent entirely | **ESCALATE** — primary demo teams must be ranked. Check if team exists in DB with a different name variant. |

---

## Query 3 — Merge Coverage Check

**What this finds:** What fraction of top-500 teams in each age group have at least one merge record. This is a sanity check on the merging system's completeness. Too low = merging system is under-running; too high = possible over-merging.

**Copy-paste ready:**

```sql
-- Query 3: Merge coverage check — top 500 teams per age group
SELECT
  age_group,
  COUNT(*)                                                          AS top500_teams,
  COUNT(CASE WHEN merge_count > 0 THEN 1 END)                      AS teams_with_merges,
  ROUND(
    100.0 * COUNT(CASE WHEN merge_count > 0 THEN 1 END) / COUNT(*),
    1
  )                                                                 AS pct_merged
FROM (
  SELECT
    r.age_group,
    r.team_id,
    COUNT(tm.id) AS merge_count
  FROM rankings r
  LEFT JOIN team_merges tm ON tm.primary_team_id = r.team_id
  WHERE r.rank <= 500
  GROUP BY r.age_group, r.team_id
) sub
GROUP BY age_group
ORDER BY age_group;
```

**Save output to:** `02 - Tiger Tournaments/Projects/Reports/team-merges-query3-[date].md`

**Result thresholds:**

| `pct_merged` per age group | Interpretation |
|---------------------------|----------------|
| 20–60% | **Expected** — name variants are common; this range is healthy. |
| < 5% | **Under-running signal** — merging system may have stopped or missed a batch. File in ELO briefing for investigation. |
| > 80% | **Over-merging signal** — investigate whether distinct teams are being collapsed. Flag to ELO. |

---

## Decision Table

| Q1 Result | Q2 Result | Q3 Result | Action |
|-----------|-----------|-----------|--------|
| PASS | PASS | Expected (20–60%) | File GREEN in ELO briefing. No action required before DSS. |
| PASS | FLAG (demo team 2× rows) | Any | Run targeted merge SQL from `task-2026-04-25-team-merge-pre-dss-audit.md` for the flagged team. Confirm merge applied. File resolution in next ELO briefing. |
| PASS | PASS | Under-running (< 5%) | ELO investigates merge pipeline in next session. Not a DSS blocker if Q2 passes. |
| YELLOW | Any | Any | Send Q1 output to ELO. ELO reviews top 20 by rating_spread. Files triage in next briefing. SENTINEL reviews triage before May 9. |
| FAIL | Any | Any | Escalate to SENTINEL. ELO does not finalize Club Rankings or demo examples until SENTINEL clears. |
| Any | FLAG (Football Academy NJ or PA Classics absent) | Any | Escalate to SENTINEL immediately. Primary demo team missing from DB is a pre-DSS blocker. |

---

## ELO Action After Results

Once Presten sends the three output files, ELO will:

1. Classify each query per the thresholds above.
2. File a triage summary in the next briefing: "Team merges audit: Q1 [result], Q2 [result], Q3 [result]. Anomaly count: [N]. Demo teams clean: YES / NO."
3. If any FLAG: identify specific teams, cross-check against `Rankings/Rank Bands — Demo Examples Document.md`, and file replacement candidates if needed.
4. Update the DSS Feature Readiness Tracker note for the Rank Bands row: "Demo team merge status: CONFIRMED / FLAGGED."

---

## References

- `task-2026-04-23-team-merges-verification-campaign.md` — source SQL methodology and risk classification framework
- `task-2026-04-25-team-merge-pre-dss-audit.md` — targeted demo team merge SQL (referenced in Q2 decision table)
- `Rankings/Rank Bands — Demo Examples Document.md` — primary demo teams (Football Academy NJ, PDA, FC Stars, Cedar Stars, PA Classics, NJ Wildcats)
- `Rankings/Club Rankings Reference Spot-Check Set.md` — additional demo clubs included in Query 2
- `Rankings/Merge Audit Triage Framework — May 2026.md` — ELO's triage methodology after results received

*ELO — 2026-04-26*
