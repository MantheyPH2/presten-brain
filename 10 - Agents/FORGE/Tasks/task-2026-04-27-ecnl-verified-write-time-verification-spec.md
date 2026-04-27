---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: high
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/ecnl_verified — Pipeline Write-Time Verification Spec.md"
---

# Task: ecnl_verified — Pipeline Write-Time Verification Spec

## Context

FORGE's April 27 03:06 session filed the `ecnl_verified` Backfill SQL Package, which correctly sets the column for all existing ECNL games. FORGE's own Section 6 of that document flagged a critical follow-on risk: **new ECNL games ingested by the daily pipeline after May 1 must also receive `ecnl_verified = TRUE` at write time.** If they do not, the column is only accurate for historical games and silently grows stale as new games arrive.

SENTINEL flagged this as an untracked gap in the 03:06 briefing and committed to assigning a formal task this cycle.

This matters because ELO's ECNL migration evidence campaign (CP1–CP7) depends on `ecnl_verified` accurately representing the full ECNL game set — including games played in May and June 2026 leading up to the migration. If May–June games are not flagged, CP2 and subsequent checkpoints will undercount ECNL-verified games and may produce a false-FAIL on a checkpoint. The migration is June 2026. The verification window opens May 1.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/ecnl_verified — Pipeline Write-Time Verification Spec.md`

---

### Section 1: Write Path Identification

Identify where in the pipeline code new ECNL games are written to the database. FORGE should check:

1. The scraper(s) responsible for ECNL game ingestion (which source tags games as `source = 'ecnl'`)
2. The database write function (INSERT or UPSERT) for ECNL games
3. Whether `ecnl_verified` is included in the write path

State one of three findings per write path:

- **Currently sets ecnl_verified = TRUE** — no change needed. Confirm with example code reference.
- **Write path exists but ecnl_verified is not included** — identify the specific file and function to modify.
- **Write path is unclear or uses a shared ingestion layer** — describe what FORGE found and the investigation step needed.

---

### Section 2: Required Code Change (if applicable)

If `ecnl_verified` is not currently set at write time, specify exactly what change is needed:

- File path and function name to modify
- The specific line or block to add (write the actual code or pseudocode)
- Whether the change requires a schema migration or is purely application-layer
- Whether the change is additive-only (safe to deploy) or modifies existing behavior

If no code change is needed (ecnl_verified is already being set), state this explicitly and close the task.

---

### Section 3: Verification Query

Write a SQL query FORGE can run 3–5 days after May 1 to verify that new ECNL games (ingested since May 1) have ecnl_verified populated:

```sql
-- Example structure — FORGE writes the correct version
SELECT
  COUNT(*) AS ecnl_games_since_may1,
  SUM(CASE WHEN ecnl_verified IS TRUE THEN 1 ELSE 0 END) AS verified_count,
  ROUND(100.0 * SUM(CASE WHEN ecnl_verified IS TRUE THEN 1 ELSE 0 END) / COUNT(*), 1) AS pct_verified
FROM games
WHERE source = 'ecnl'
  AND ingested_at >= '2026-05-01';
```

If the schema uses a different timestamp column than `ingested_at`, use the correct column. If no ingested_at equivalent exists, note this and propose an alternative (e.g., game_date >= May 1 as a proxy).

**Expected result:** 100% of post-May-1 ECNL games should have ecnl_verified = TRUE. If < 100%, the code change was not deployed or did not cover all ingestion paths.

---

### Section 4: Deployment Timeline Recommendation

Based on findings:

- If code change is needed: state the recommended deploy window (before May 5 if possible; absolutely before May 16 DSS)
- If no change needed: confirm the spec is complete and mark the gap as resolved
- If investigation is incomplete: state what FORGE still needs to do and the expected completion date

---

## Quality Standard

- Section 1 must reference specific code files, not describe them in the abstract. If FORGE cannot locate the ECNL write path, state this explicitly with the files checked.
- Section 2 must provide an actionable code change, not a description of what the change should accomplish.
- The verification query must be copy-paste ready.

## Why This Matters

The ecnl_verified backfill is a one-time operation. Without write-time population, every new ECNL game after May 1 silently has `ecnl_verified = NULL` or `FALSE`. ELO's CP2 checkpoint (scheduled April 29) and subsequent checkpoints query the ecnl_verified column to build migration evidence. A growing un-verified tail in the data means ELO's evidence is incomplete and the migration option selection may be made on stale data. Catching this now — before May 1 — means the fix can go in with the pipeline deployment, not after.

## Definition of Done

- File written at the deliverable path
- Section 1 identifies the ECNL write path (or states why it cannot be identified)
- Section 2 provides a code change spec or explicit confirmation that no change is needed
- Section 3 has a copy-paste ready verification query
- Section 4 gives a deployment timeline recommendation
- FORGE briefing note: "ecnl_verified write-time spec complete. Write path: [found/not found]. Code change: [needed/not needed]. Verification query: ready. Deploy by: [date]."

## References

- `Infrastructure/ecnl_verified — Backfill SQL Package.md` — the backfill this task extends to ongoing ingestion
- `task-2026-04-26-ecnl-migration-evidence-collection.md` — ELO's CP2 checkpoint depends on ecnl_verified accuracy
- `Infrastructure/April 29 — FORGE Execution Checklist.md` — Step 6: ecnl_verified backfill runs April 29; write-time verification is the follow-on step
