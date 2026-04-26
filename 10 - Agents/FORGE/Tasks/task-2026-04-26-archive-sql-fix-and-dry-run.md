---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Archive Inactive Teams — Dry-Run Results.md"
---

# Task: Fix Archive Query GROUP BY Bug and Run First Dry-Run

## Context

The `archived_at` schema migration ran successfully on 2026-04-22. The `archive-inactive-teams.js` script has been designed and specified. The migration is done; the first production-adjacent run has not happened.

SENTINEL's 2026-04-22 review flagged a SQL correctness issue: the archive candidate query includes `g.event_name` in both the `SELECT` and `GROUP BY` clauses. This means a team with games across multiple events produces multiple rows — one per event — instead of one row per team. During a dry-run this will produce an inflated candidate count and may surface the same team_id multiple times. During an actual UPDATE it would not cause duplicates (UPDATE is idempotent by team_id), but the dry-run output will be misleading.

This bug must be fixed before the dry-run runs. The fix is a one-line change.

## What to Produce

### Step 1: Fix the SQL

In `archive-inactive-teams.js` (or wherever the candidate query lives), remove `g.event_name` from the `GROUP BY` clause and from the `SELECT`. The `last_event_name` output field is not needed for the candidate decision logic — only `team_id`, `last_game_date`, and `game_count` are needed for the dry-run output and the subsequent UPDATE.

If `last_event_name` is useful for human review during the dry-run (to understand what a team last played), pull it as a subquery scalar instead:

```sql
-- Instead of GROUP BY ..., g.event_name
-- Use:
(SELECT g2.event_name FROM games g2 WHERE g2.team_id = t.id ORDER BY g2.game_date DESC LIMIT 1) AS last_event_name
```

Document the fix with one comment: what was wrong and what was changed. No other refactoring.

### Step 2: Run the Dry-Run

Execute `archive-inactive-teams.js --dry-run` against the current production database. The dry-run should:
- Query the 180 de-prioritized teams against the candidate criteria (last game date < cutoff, game count < threshold — use criteria from the original archive design doc)
- Return a list of team_ids, team names, last game dates, and game counts
- NOT execute any UPDATE

### Step 3: File the Dry-Run Results

File: `02 - Tiger Tournaments/Projects/Infrastructure/Archive Inactive Teams — Dry-Run Results.md`

Structure:

```
## Run Date
[date and time]

## SQL Fix Applied
[one line: what was changed and why]

## Dry-Run Configuration
- Total de-prioritized teams queried: 180
- Archive cutoff date used: [date]
- Game count threshold used: [N]

## Results Summary
- Candidate count (meets archive criteria): [N]
- Not candidate (still has recent games): [N]
- Review required (edge cases): [N]

## Candidate List
[Table: team_id | team_name | last_game_date | game_count | last_event_name]

## Edge Cases
[Any teams where the criteria are ambiguous — e.g., exactly at the cutoff date]

## SENTINEL Disposition
[ ] Dry-run reviewed
[ ] Candidate list approved for production UPDATE
[ ] Production UPDATE authorized: YES / HOLD
If HOLD, reason: ___
```

---

## Quality Standard

- The GROUP BY fix must be applied and documented before the dry-run runs. Do not run the dry-run against the unfixed query.
- The candidate count must be a single number (one row per team_id), not a count of event-team pairs.
- The SENTINEL Disposition section must be blank — SENTINEL fills it after review.
- If the dry-run produces 0 candidates or >170 candidates, flag this in the Results Summary as unexpected and do not proceed to production UPDATE without SENTINEL sign-off.

## Why This Matters

The 180 de-prioritized teams are currently loading through pipeline Steps 6–8 on every run, contributing to Glicko-2 RD inflation and wasting pipeline capacity. The archival pass removes them from active processing without deleting their data. This has been ready to run since April 22 — the only blocker was the SQL bug and the absence of someone running the command. With the backoff and DLQ now in place, the pipeline infrastructure is stable enough that an archival pass won't disrupt anything.

May 1 the daily pipeline goes live. It would be better to have the archive pass complete before May 1 so the first daily run operates on a clean active-team set.

## References

- `Infrastructure/Archive Workflow.md` — design doc and original candidate query
- SENTINEL Review 2026-04-22-FORGE.md — SQL bug originally flagged here
- `Infrastructure/GotSport API Scraper.md` — confirms DLQ and backoff are now in place
