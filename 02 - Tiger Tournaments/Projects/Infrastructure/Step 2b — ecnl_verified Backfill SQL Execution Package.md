---
title: Step 2b — ecnl_verified Backfill SQL Execution Package
tags: [infrastructure, sql, ecnl, ecnl-verified, backfill, april-28, evo-draw, forge]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: ready — execute on April 28, between Step 2 and Step 3
companion: "Infrastructure/April 28 Execution Log.md"
---

# Step 2b — ecnl_verified Backfill SQL Execution Package

> **Run on April 28, immediately after Step 2 (ALTER TABLE) completes and before Step 3 (Rankings Recompute).** All three queries below are copy-paste ready. Execute in order. Record results in the April 28 Execution Log — Step 2b block.

---

## Query 1: Pre-Flight Count

Run this **before** the UPDATE to confirm scope.

```sql
-- Run BEFORE the UPDATE to confirm scope
SELECT COUNT(*) AS games_to_update
FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
  AND (ecnl_verified IS NULL OR ecnl_verified = FALSE);
```

**Expected result:** 8,000–40,000 rows.

> [!warning] Stop Condition
> If result is **< 100**, something is wrong — the column may not exist yet (Step 2 did not complete), or source/event_tier filters are not matching. Flag to SENTINEL before proceeding. Do not run the UPDATE.

**Record the count in the Execution Log Step 2b block.**

---

## Query 2: Step 2b UPDATE

Run this after confirming the pre-flight count is in range.

```sql
UPDATE games
SET ecnl_verified = TRUE
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl';
```

**Expected output:** `UPDATE N` where N matches the pre-flight count from Query 1.

**Record the row count in the Execution Log Step 2b block.**

---

## Query 3: Post-Execution Verification

Run this **immediately after** the UPDATE to confirm it applied cleanly.

```sql
-- Run immediately after UPDATE to confirm
SELECT
  ecnl_verified,
  COUNT(*) AS game_count
FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
GROUP BY ecnl_verified
ORDER BY ecnl_verified;
```

**Expected result:** All rows show `ecnl_verified = TRUE`. The `TRUE` count should match the UPDATE row count and the pre-flight count.

> [!warning] Failure Condition
> If any row shows `ecnl_verified = FALSE` or `NULL` after the UPDATE: the UPDATE did not apply to all rows. Flag to SENTINEL immediately. Do not proceed to Step 3 until resolved.

---

## Step 2b Block — For Paste Into April 28 Execution Log

Paste the block below into `Infrastructure/April 28 Execution Log.md` between Step 2 and Step 3.

---

### Step 2b: ecnl_verified Backfill UPDATE

**Reference:** `Infrastructure/Step 2b — ecnl_verified Backfill SQL Execution Package.md`

| Field | Presten Fills |
|-------|---------------|
| Time started | |
| Pre-flight count query run (Query 1) | ✅ / ❌ |
| Pre-flight count result (# games to update) | |
| Pre-flight count in expected range (8,000–40,000)? | YES / NO — if NO, flag to SENTINEL before proceeding |
| UPDATE statement executed (Query 2) | ✅ / ❌ |
| Rows affected (UPDATE output) | |
| Rows affected matches pre-flight count? | YES / NO — if NO, flag |
| Post-execution verification query run (Query 3) | ✅ / ❌ |
| Verification result: all rows ecnl_verified = TRUE? | YES / NO |
| Any rows remaining FALSE or NULL? | |
| Any errors or unexpected output? | |
| Step 2b complete? | ✅ / ❌ |

---

## Why Step 2b Exists

The April 28 session adds the `ecnl_verified` column via ALTER TABLE (Step 2) but does not backfill historical games. Without this UPDATE, all ECNL games that existed before April 28 remain `ecnl_verified = FALSE` permanently — even though they are verifiable ECNL source games. The June 2 ECNL migration uses `ecnl_verified` as the baseline for dedup verification. If the column is not backfilled on April 28, every row written before April 28 is wrong, and the June 2 baseline is unusable.

Running this UPDATE on April 28 costs ~60 seconds. Not running it creates a June 2 incident.

---

## References

- `Infrastructure/April 28 Execution Log.md` — paste Step 2b block between Step 2 and Step 3
- `Infrastructure/April 28 Escalation Reference Card.md` — session context for April 28
- `Infrastructure/ECNL ecnl_verified Column — Population Plan.md` — column design and June 2 dependency
- `Infrastructure/April 29 Pipeline Validation Spec.md` — forward-write check that validates this backfill is not undone by May 1 pipeline run

*FORGE — 2026-04-25*
