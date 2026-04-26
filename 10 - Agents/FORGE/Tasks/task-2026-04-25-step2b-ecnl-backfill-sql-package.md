---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-25
priority: urgent
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Step 2b — ecnl_verified Backfill SQL Execution Package.md"
---

# Task: Step 2b — ecnl_verified Backfill SQL Execution Package

## Context

SENTINEL's 21:23 briefing flags a CRITICAL quality gap: the April 28 Execution Log has no structured Step 2b entry for the `ecnl_verified` backfill UPDATE. The log currently covers Step 1 (GA ASPIRE tier fix), Step 2 (ALTER TABLE), and Step 3 (rankings recompute) — but the UPDATE that sets `ecnl_verified = TRUE` for all existing ECNL games is absent.

Without this step captured and executed on April 28, all historical ECNL games remain `ecnl_verified = FALSE` permanently. The June 2 ECNL migration depends on this column being accurately populated. If Step 2b is missed on April 28, it becomes a June 2 dependency that FORGE will need to track and re-execute — at the worst possible time.

This task closes that gap. FORGE produces the SQL package AND adds a Step 2b block directly to the April 28 Execution Log so Presten cannot miss it.

## What to Produce

### File: `Infrastructure/Step 2b — ecnl_verified Backfill SQL Execution Package.md`

A standalone SQL execution package. Presten copies each query in sequence.

---

### Pre-Flight Count Query

Before running the UPDATE: how many games will be affected?

```sql
-- Run this BEFORE the UPDATE to confirm scope
SELECT COUNT(*) AS games_to_update
FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
  AND (ecnl_verified IS NULL OR ecnl_verified = FALSE);
```

**Expected result:** 8,000–40,000 rows. If result is < 100, something is wrong — flag to SENTINEL before proceeding.

---

### Step 2b UPDATE Statement

```sql
UPDATE games
SET ecnl_verified = TRUE
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl';
```

**Record:** Number of rows affected (UPDATE will print `UPDATE N`). Write this in the Execution Log Step 2b entry.

---

### Post-Execution Verification Query

```sql
-- Run immediately after UPDATE to confirm it applied
SELECT
  ecnl_verified,
  COUNT(*) AS game_count
FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
GROUP BY ecnl_verified
ORDER BY ecnl_verified;
```

**Expected result:** All rows show `ecnl_verified = TRUE`. Any row with `FALSE` or `NULL` means the UPDATE was incomplete — flag to SENTINEL.

---

### Step 2b Block for Execution Log

Produce the following as a formatted block for Presten to **paste into the April 28 Execution Log** between Step 2 and Step 3:

```
### Step 2b: ecnl_verified Backfill UPDATE

| Field | Presten Fills |
|-------|---------------|
| Time started | |
| Pre-flight count query run (games to update) | ✅ / ❌ |
| Pre-flight count result (# rows) | |
| UPDATE statement executed | ✅ / ❌ |
| Rows affected (UPDATE output) | |
| Post-execution verification query run | ✅ / ❌ |
| Verification result (all ecnl_verified = TRUE?) | ✅ / ❌ |
| Any rows remaining FALSE or NULL? | |
| Any errors or unexpected output? | |
| Step complete? | ✅ / ❌ |
```

---

## Deliverable Requirements

1. File written at `Infrastructure/Step 2b — ecnl_verified Backfill SQL Execution Package.md`
2. All three queries present and copy-paste ready
3. Expected result ranges defined for pre-flight count
4. Step 2b log block is formatted identically to the Execution Log's other step blocks
5. FORGE also manually edits `Infrastructure/April 28 Execution Log.md` to insert the Step 2b block between Step 2 and Step 3 — the SQL package alone is not sufficient. The Execution Log must contain Step 2b.

## Why This Matters

This is the only remaining structural gap in April 28 readiness. Every other step is documented and logged. Step 2b is the backfill that makes the entire ECNL dedup feature meaningful on June 2. Missing it on April 28 is not a catastrophic failure, but it creates a fragile undocumented dependency exactly when the June 2 migration is being finalized. Filing this now, before April 28, is a 30-minute FORGE task that eliminates a potential June 2 incident.

## References

- `Infrastructure/April 28 Execution Log.md` — must be edited directly by FORGE to add Step 2b
- `Infrastructure/April 28 Reference Card.md` — original reference for Step 2
- `Infrastructure/ECNL Dedup — ecnl_verified Column Population Plan.md` — context for June 2 dependency
