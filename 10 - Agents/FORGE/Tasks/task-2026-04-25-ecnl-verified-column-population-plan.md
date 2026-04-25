---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/ECNL ecnl_verified Column — Population Plan.md"
---

# Task: ECNL `ecnl_verified` Column — Population Plan

## Context

The April 28 session adds the `ecnl_verified` column to the `games` table with `DEFAULT FALSE`. This is the correct schema step. What does not follow from that step alone: any mechanism to set `ecnl_verified = TRUE` for actual ECNL games.

If no pipeline logic is updated to populate this column, every ECNL game will remain `ecnl_verified = FALSE` indefinitely. The column exists in the schema but carries no signal. The ECNL migration spec, the dedup strategy, and ELO's rating continuity spec all depend on `ecnl_verified` meaning something.

This task: before April 28 runs, FORGE writes the plan for how this column gets populated post-deployment.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/ECNL ecnl_verified Column — Population Plan.md`

---

### Section 1: Population Trigger

What sets `ecnl_verified = TRUE` for a game?

Define the exact condition:
- Is it set during crawl ingestion (when the game is first inserted)?
- Is it set during a post-crawl update step?
- Is it set manually by Presten after validation, or automatically by pipeline logic?

State the criterion: what makes a game "ecnl_verified"? Examples:
- Source is an ECNL-registered org_id AND event_tier = 'ecnl'
- Source passes a specific ECNL org validation check
- Manual flag from Presten's ECNL migration verification session

Pick the one that is correct for Evo Draw's architecture. If FORGE is uncertain, file a Queue item — but include a proposed answer.

---

### Section 2: Backfill Plan for Existing Games

After April 28 adds the column with DEFAULT FALSE, how many existing ECNL games need to be backfilled to `TRUE`?

- Query to count games that should have `ecnl_verified = TRUE` but currently would be `FALSE` (i.e., ECNL games already in the database before April 28)
- SQL for the backfill UPDATE statement
- Estimated row count (rough order of magnitude — hundreds? thousands?)
- Whether the backfill should run during the April 28 session or as a separate step

If running during April 28: add this step to the April 28 Execution Log as Step 2b (after the ALTER TABLE, before rankings recompute). If running separately: specify when and why the delay is acceptable.

---

### Section 3: Forward Pipeline Logic

After the backfill, how does new game ingestion handle `ecnl_verified`?

- What code change in the pipeline sets this field for newly ingested ECNL games?
- If no code change is needed (e.g., `ecnl_verified` is derived from existing fields), show the derivation logic
- If a code change is needed: write the specific change as a TODO for Presten's May 1 pipeline session

---

### Section 4: Verification Query

One query Presten runs after April 28 + backfill to confirm `ecnl_verified` is populated correctly:

```sql
-- Should return: count of ecnl_verified = TRUE games, count of ecnl_verified = FALSE ECNL games (should be 0 after backfill)
```

Pass condition: `ecnl_verified = FALSE` count for ECNL-tagged games = 0.

---

## Quality Standard

- The population trigger must be a specific, implementable rule — not "set it when appropriate"
- The backfill SQL must be safe to run (idempotent — running it twice should not cause errors or double-sets)
- If FORGE determines the column is purely manual (Presten sets it case-by-case), say so explicitly with reasoning — that is a valid answer, not a gap

## Why This Matters

April 28 adds the column. Without this plan, that's where the ECNL verification feature stops — a schema stub with no population logic. ELO's ECNL rating continuity spec (`Rankings/ECNL Rating Continuity Spec.md`) and the June 2 migration verification spec both depend on `ecnl_verified` being an accurate signal. If it's always FALSE, those specs are testing nothing.

## References

- `Infrastructure/April 28 Execution Log.md` — April 28 steps; backfill may need to be inserted as Step 2b
- `task-2026-04-23-ecnl-dedup-verified-flag.md` — original spec for this column (check if population logic was defined there)
- `Rankings/ECNL Rating Continuity Spec.md` — downstream dependency
- `task-2026-04-24-june2-ecnl-migration-verification-spec.md` — ELO verification that depends on this flag
