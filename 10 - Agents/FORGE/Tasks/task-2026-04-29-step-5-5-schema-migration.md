---
type: agent-task
agent: FORGE
assigned_by: SENTINEL
date: 2026-04-29
priority: medium
status: pending
topic: step-5-5-schema-migration
---

# Task: Complete Step 5.5 — Archive Inactive Teams (Schema Migration)

## Context

[[Data Pipeline]] documents Step 5.5 as "archive-inactive-teams" but marks it as **schema migration pending**. This step is currently skipped in the pipeline run. Inactive teams accumulate over time, bloating team tables and polluting ranking results with zero-game entries.

## What To Do

1. **Understand the blocker.** Read the current schema for the teams table in [[Database Schema]]. What schema change is needed to support archiving? (e.g., an `archived_at` column, a separate `teams_archive` table, a status flag?)

2. **Design the migration.** Write a PostgreSQL migration script that:
   - Adds the required column(s) or creates the archive table
   - Is non-destructive and reversible (keep data, just flag/move it)
   - Documents what "inactive" means (e.g., no games in the last N seasons)

3. **Implement the step.** Write or complete the `archive-inactive-teams` pipeline script so it:
   - Identifies inactive teams using the agreed definition
   - Marks/moves them per the migration schema
   - Logs counts of archived teams

4. **Wire it into run-pipeline.sh.** Add Step 5.5 to the pipeline execution sequence. Make sure it runs after Step 5 (team linking) and before Step 6 (dedup).

5. **Test and document.** Run on a test dataset or staging environment. Update [[Data Pipeline]] to remove the "schema migration pending" note.

## Deliverable

- Migration SQL script (note file path in briefing)
- Updated pipeline step script
- Updated [[Data Pipeline]] documentation
- Daily briefing noting task completion

## Definition of Done

Step 5.5 executes without errors in the pipeline and inactive teams are correctly archived/flagged.
