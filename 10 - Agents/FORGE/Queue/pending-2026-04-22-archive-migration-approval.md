---
type: agent-queue
agent: FORGE
category: question
priority: medium
date: 2026-04-22
status: pending
answer: ""
---

# Question: Authorization to Run `archived_at` Schema Migration

## Context

As part of today's archive workflow design task (SENTINEL task 2), I designed a schema migration that adds an `archived_at` column to the `teams` table. The full design document is at `Pipeline/Archive Workflow.md`.

The migration is:
```sql
ALTER TABLE teams ADD COLUMN archived_at TIMESTAMP NULL DEFAULT NULL;
CREATE INDEX idx_teams_archived_at ON teams (archived_at);
```

This is a purely additive change — it adds a nullable column with no default on existing rows. All existing teams retain `archived_at = NULL`, which is semantically equivalent to "active" (the new query pattern is `WHERE archived_at IS NULL`). No data is changed, renamed, or deleted by the migration itself.

## What Happens After the Migration

Once the schema is in place, the archive script (`archive-inactive-teams.js`) can run in `--dry-run` mode to generate a candidate list. The dry run outputs which teams qualify for archival without executing any `UPDATE` statements. You can review the list before anything actually changes.

Estimated output from the dry run: 155–165 of the ~180 de-prioritized tournament teams identified in the 2026-04-21 stale audit. The remaining 15–25 would be excluded because they also played in an active event in the last 90 days.

## The Question

**Do I have authorization to:**

1. Run the `ALTER TABLE` migration on the production `youth_soccer` database?
2. Run the archive script in `--dry-run` mode and share the candidate list with you?
3. (Pending your review of the dry run output) Execute the actual archival `UPDATE` for confirmed candidates?

I am asking for explicit confirmation before touching the schema or running any UPDATEs against the `teams` table on production.

## Why This Matters Now

The 180 de-prioritized teams are causing two problems:
- They load into pipeline Steps 6 and 7 as active teams, adding unnecessary processing overhead
- Their Glicko-2 RD continues to increase in the rankings table, signaling "uncertain rating" for teams that have permanently stopped competing

The fix is low-risk (additive schema change, reversible archival via `UPDATE teams SET archived_at = NULL WHERE team_id = ANY(...)` if needed) and addresses a data quality issue that recurs every season without a formal archival process.

## References

- `Pipeline/Archive Workflow.md` — full design doc including migration SQL, archival logic, and safety guards
- `Reports/stale-teams-2026-04-21.csv` — source data for the 180 de-prioritized teams
- [[Database Schema]] — current `teams` table structure
