---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
priority: high
due: 2026-04-25
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stale Team Backfill Runbook — April 2026.md"
---

# Task: Backfill Runbook — Elevate Stage 3 Sequencing Warning

## Context

SENTINEL flagged this in the 08:52 review. The Stale Team Backfill Runbook (`Infrastructure/Stale Team Backfill Runbook — April 2026.md`) documents a critical sequencing constraint: Stage 3 must not run on the same night as the first daily pipeline run. This constraint is currently mentioned in the body of Stage 3 but is not prominently enough placed to catch someone executing quickly.

Presten may open this runbook and jump to Stage 3 without reading the full context. If Stage 3 runs the same night as the first daily pipeline, the two operations will conflict at the database level — potentially corrupting the stale-team identification state and requiring a full re-run.

This is a vault quality fix. Due before Presten executes the runbook (before May 1).

## What to Do

Open `02 - Tiger Tournaments/Projects/Infrastructure/Stale Team Backfill Runbook — April 2026.md`.

At the top of the Stage 3 section (before any other content in that section), add a callout block in this format:

```
> [!warning] SEQUENCING CONSTRAINT — READ BEFORE RUNNING STAGE 3
> **Do NOT run Stage 3 on the same night as the first `run-daily.sh` pipeline run.**
>
> Stage 3 identifies stale teams using a snapshot of the `last_game_date` column. If the daily pipeline runs concurrently or immediately after Stage 3 begins, new game records may update `last_game_date` mid-scan, producing false negatives (teams incorrectly excluded from the backfill batch).
>
> **Correct sequence:** Complete the daily pipeline first. Confirm it succeeded (check logs). Then run Stage 3 the following day.
>
> If you are unsure whether the daily pipeline has run for the first time yet, check: `[log path documented in Stage 1.3 of the pre-flight checklist]`.
```

Adapt the language to match the existing runbook style. If the runbook uses different terminology (e.g., "Phase" instead of "Stage"), match it.

## Definition of Done

- The Stage 3 section of the runbook opens with the warning callout block
- The callout is the first thing Presten sees when navigating to Stage 3
- The constraint (don't run Stage 3 same night as first daily pipeline) is explicitly stated
- The correct sequence is stated (pipeline first, Stage 3 next day)
- Summary in briefing: "Backfill runbook Stage 3 warning block added."

## Why This Matters

This is a single-point-of-failure sequencing error. If Presten runs Stage 3 and the daily pipeline on the same night (a natural assumption — "run everything tonight"), the backfill produces incorrect results. The runbook is the only place this constraint is documented. One callout block prevents a potentially multi-hour debugging session.

## References

- `02 - Tiger Tournaments/Projects/Infrastructure/Stale Team Backfill Runbook — April 2026.md` — the file to edit
- SENTINEL Review `10 - Agents/SENTINEL/Reviews/2026-04-24-08:52-FORGE.md` — the original flag
- `02 - Tiger Tournaments/Projects/Infrastructure/ECNL Migration Pre-Flight Checklist — June 2026.md` Section 1.3 — cross-reference for the log path placeholder
