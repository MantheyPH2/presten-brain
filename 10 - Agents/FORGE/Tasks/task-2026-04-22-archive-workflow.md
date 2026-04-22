---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-22
status: pending
priority: medium
---

# Task: Design Seasonal Team Archive Workflow

## Context

Your stale team audit identified ~180 teams from tournaments that ended but are still marked `active` in the `teams` table. These are de-prioritized tournament teams that were never formally archived after their season concluded. This is a recurring data hygiene problem — every season that ends without an archival pass leaves more orphaned active records.

The risk is subtle but real: the ranking engine (Phase 1) loads team merges and the pipeline processes teams marked `active`. Stale "active" teams with no recent data can distort rating deviation calculations in Glicko-2, because the model interprets inactivity as fewer matches played (and increases RD accordingly), not as missing data. Teams that should be fully settled (no longer competing) are instead drifting.

## Required Work

### Step 1: Audit the 180 De-Prioritized Teams
From the stale team audit, identify the 180 teams flagged as "de-prioritized tournament teams." Determine:
- Which tournaments they belong to (pull `event_name` from the `games` table for these `team_id`s)
- Whether those tournaments have a documented end date in GotSport
- Whether any of the 180 teams have also played in active tournaments since the de-prioritized event ended (a team in a concluded tournament may still be active in an ongoing league)

Do NOT archive any team that has played in an active event in the last 90 days.

### Step 2: Schema Recommendation
Propose adding an `archived_at` column (nullable timestamp) to the `teams` table. Write the migration SQL:

```sql
ALTER TABLE teams ADD COLUMN archived_at TIMESTAMP NULL DEFAULT NULL;
CREATE INDEX idx_teams_archived_at ON teams (archived_at);
```

Define the query pattern for "active only" queries:
```sql
WHERE archived_at IS NULL
```

This should be the standard filter in all pipeline steps that currently filter on `active = true` or `status = 'active'`. Document which pipeline steps need this filter added.

### Step 3: Archive Logic Specification
Write the logic for a new pipeline step: `archive-inactive-teams.js` (or equivalent). The archival trigger should be:

- A team's most recent game is more than **180 days ago**, AND
- The event that game belongs to has `end_date` more than **90 days ago**, AND
- The team has not appeared in any sync window in the last **30 days** (meaning GotSport is not returning data for them even when explicitly queried)

The script should:
1. Identify candidate teams matching the above criteria
2. Log the candidate list (never archive without a log)
3. Set `archived_at = NOW()` on confirmed candidates
4. Output a summary: N teams archived, N candidates skipped (still active)

### Step 4: Integration Point
Where in `run-pipeline.sh` should this step run? It should run BEFORE Step 6 (dedup-and-link-teams.js) so that archived teams are excluded from the dedup pass and not included in the team merges loader (Phase 1 of the ranking engine).

## Deliverable

Write a design document to `Pipeline/Archive Workflow.md` covering:
1. The `archived_at` schema change (with migration SQL)
2. The archival logic specification
3. Pipeline integration point
4. The immediate archival recommendation for the current 180 de-prioritized teams (once safe to execute)

Post a summary in your 2026-04-22 briefing. This does not require immediate implementation — the design document is the deliverable. Presten will approve before any `archived_at` migration runs on production.

## References

- [[Database Schema]] — current `teams` table structure
- [[Data Pipeline]] — pipeline step sequence
- [[Ranking Engine]] — Phase 1 team merges loading
- [[Dedup Strategy]] — how team merges interact with active teams
