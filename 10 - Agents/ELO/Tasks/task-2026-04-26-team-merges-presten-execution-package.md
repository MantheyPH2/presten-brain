---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: high
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Team Merges Audit — Presten Execution Package.md"
---

# Task: Team Merges Audit — Presten Execution Package

## Context

The team merges verification campaign (`task-2026-04-23-team-merges-verification-campaign.md`) established that ~8,000–20,000 games may be affected by incorrect team merges in the 27,820 `team_merges` entries. The merge audit SQL was written and filed April 23. It has been blocked on psql access ever since — listed as a HIGH-priority blocked task in every briefing.

The reason it remains blocked: there is no Presten execution package. The SQL exists in the task file. Presten cannot open a task file, copy a query, and know what to do with the result. Every other ELO execution task has been converted to a self-contained package. This one has not.

This task closes that gap. Format identically to `Rankings/Boys Option A Spot Check — Presten Execution Package.md`.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Team Merges Audit — Presten Execution Package.md`

---

### Pre-Run Gate

One line: run after April 28 recompute is confirmed complete. Team merges affect ratings — running the audit before the recompute means the baseline shifts. The post-April-28 rating state is the correct baseline for identifying anomalous merges.

**Run window:** April 29 – May 5. Due before May 9 DSS readiness check — if bad merges affect demo teams (Football Academy NJ, PA Classics, PDA), SENTINEL needs time to correct before DSS.

---

### Query 1 — High-Confidence Merge Anomaly Detection

Pull from `task-2026-04-23-team-merges-verification-campaign.md`. This query identifies merged teams where the rating spread across merged components exceeds the threshold for a credible merge.

Expected result columns: `team_id`, `team_name`, `merge_count`, `max_component_rating`, `min_component_rating`, `rating_spread`, `anomaly_flag`.

**PASS:** < 50 rows with `anomaly_flag = TRUE`.
**YELLOW:** 50–200 anomaly rows → file in SENTINEL briefing. ELO reviews top 20 by rating spread.
**FAIL:** > 200 anomaly rows OR any anomaly row includes a top-50 ranked team → escalate to SENTINEL. Stop merge audit and request Presten guidance.

Save output: `02 - Tiger Tournaments/Projects/Reports/team-merges-query1-[date].md`

---

### Query 2 — Demo Team Merge Status Check

Targeted check: confirm the primary demo teams are NOT anomalous merges.

```sql
SELECT t.name, t.id, COUNT(tm.id) AS merge_components, 
       MAX(r.rating) AS current_rating,
       r.rank AS current_rank
FROM teams t
LEFT JOIN team_merges tm ON tm.primary_team_id = t.id
LEFT JOIN rankings r ON r.team_id = t.id
WHERE t.name ILIKE ANY(ARRAY['%football academy%nj%', '%football academy nj%', 
                              '%PA Classics%', '%PDA%', '%NJ Wildcats%',
                              '%FC Stars%'])
  AND r.age_group IN ('U13G', 'U14G', 'U15G', 'U16G')
GROUP BY t.name, t.id, r.rank
ORDER BY r.rank;
```

**PASS:** All named demo teams appear as single records with `merge_components` ≤ 3 (expected: small clubs may have 1–3 name variants merged).
**FLAG:** Any demo team appears as 2 separate records (not merged) OR has `merge_components` > 10 (over-merged).

Save output: `02 - Tiger Tournaments/Projects/Reports/team-merges-query2-[date].md`

---

### Query 3 — Merge Coverage Check

How many teams in the top 500 per age group have at least one merge record?

```sql
SELECT age_group, 
       COUNT(*) AS top500_teams,
       COUNT(CASE WHEN merge_count > 0 THEN 1 END) AS teams_with_merges,
       ROUND(100.0 * COUNT(CASE WHEN merge_count > 0 THEN 1 END) / COUNT(*), 1) AS pct_merged
FROM (
  SELECT r.age_group, r.team_id, COUNT(tm.id) AS merge_count
  FROM rankings r
  LEFT JOIN team_merges tm ON tm.primary_team_id = r.team_id
  WHERE r.rank <= 500
  GROUP BY r.age_group, r.team_id
) sub
GROUP BY age_group
ORDER BY age_group;
```

**Expected:** 20–60% of top-500 teams have at least one merge (name variants are common). If < 5% have merges, the merging system may be under-running. If > 80% have merges, investigate over-merging.

Save output: `02 - Tiger Tournaments/Projects/Reports/team-merges-query3-[date].md`

---

### Decision Table

| Q1 Result | Q2 Result | Q3 Result | Action |
|-----------|-----------|-----------|--------|
| PASS | PASS | Expected | File GREEN in FORGE briefing. No action needed. |
| PASS | FLAG (demo team 2x rows) | Any | Run merge SQL from `task-2026-04-25-team-merge-pre-dss-audit.md` for flagged teams. File in next briefing. |
| YELLOW | Any | Any | ELO reviews top 20 anomalies. File triage in next ELO briefing. SENTINEL decides whether to hold DSS coverage claim. |
| FAIL | Any | Any | Escalate to SENTINEL. ELO does not proceed with club rankings or demo examples until SENTINEL clears. |

---

### ELO Action After Results

Once Presten sends output files:
1. ELO classifies each query per thresholds above
2. ELO files triage summary in next briefing: "Team merges audit: Q1 [result], Q2 [result], Q3 [result]. Anomaly count: [N]. Demo teams clean: YES / NO."
3. If any FLAG: ELO identifies specific teams, cross-checks against demo examples in `Rankings/Rank Bands Demo Examples.md`, and files replacement candidates if needed
4. ELO updates DSS Feature Readiness Tracker note for Rank Bands row: "Demo team merge status: CONFIRMED / FLAGGED"

---

## Quality Standard

- Every query copy-paste ready with no modification needed from Presten
- Query 2 demo team list should include all teams named in `Rankings/Rank Bands Demo Examples.md` — check that file and add any missing names
- Decision table must be complete — no "investigate" actions, only specific next steps
- Estimated run time: 15 minutes in psql for all 3 queries

## Why This Matters

8,000–20,000 games affected by bad merges is the largest unchecked data quality risk in Evo Draw. It has been deferred because it required psql access. Converting it to an execution package removes that friction — Presten can run it in the next available psql session without any further ELO involvement. If a demo team (Football Academy NJ, PA Classics) has a merge anomaly, ELO needs to know before May 9. Discovering it on May 14 (pre-demo validation day) leaves no time to fix.

## References

- `task-2026-04-23-team-merges-verification-campaign.md` — source SQL (pull queries from this file)
- `task-2026-04-25-team-merge-pre-dss-audit.md` — targeted demo team merge SQL (referenced in Q2 decision table)
- `Rankings/Rank Bands Demo Examples.md` — demo team names for Query 2
- `Rankings/Club Rankings — Reference Clubs.md` — additional demo clubs to include in Query 2
