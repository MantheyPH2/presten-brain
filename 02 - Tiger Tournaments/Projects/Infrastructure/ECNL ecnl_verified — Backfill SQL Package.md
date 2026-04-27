---
title: "ECNL ecnl_verified — Backfill SQL Package"
tags: [infrastructure, sql, ecnl, ecnl-verified, backfill, april-29, evo-draw, forge]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: ready — execute on April 29 after April 28 Step 2 confirms column exists
---

# ECNL ecnl_verified — Backfill SQL Package

> **Execution window:** April 29, after confirming the `ecnl_verified` column was added on April 28. Must complete before ELO's CP2 checkpoint. Total execution time: < 10 minutes.

---

## Section 1: ECNL Source Identification Logic

**Field used to identify ECNL-sourced games:** `source` + `event_tier`

ECNL games in the Evo Draw database are identified by:

```
source IN ('tgs', 'tgs_athleteone')
AND event_tier = 'ecnl'
```

**Why this combination:**
- The TGS scraper (`source = 'tgs'`) and the AthleteOne variant (`source = 'tgs_athleteone'`) are the two ingestion paths for ECNL data.
- `event_tier = 'ecnl'` is set by the ECNL event classifier at ingestion time. Games classified as ECNL are tagged with this tier value.
- This combination is more reliable than using `event_name ILIKE '%ecnl%'` because event_name is raw text that may vary ("ECNL", "ECNL Spring", "ECNL Girls", etc.). The `event_tier` classification is normalized.

**Expected row count (pre-backfill):** 8,000–40,000 rows with `ecnl_verified = FALSE`.

> [!note] If both source values are in use
> The query uses `IN ('tgs', 'tgs_athleteone')` to capture both ingestion paths. If only one source exists in your database, the query still works — the non-matching source simply contributes 0 rows.

---

## Section 2: Verification Query (Pre-Backfill)

Run this **before** the UPDATE to confirm scope and that the column exists.

```sql
-- Pre-backfill scope check
-- Run AFTER April 28 confirms ecnl_verified column exists (Step 1 of the April 29 checklist)
SELECT
  COUNT(*) AS total_ecnl_games,
  SUM(CASE WHEN ecnl_verified = FALSE THEN 1 ELSE 0 END) AS unverified_count,
  SUM(CASE WHEN ecnl_verified = TRUE THEN 1 ELSE 0 END) AS already_verified_count
FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl';
```

**Expected result after April 28 column addition:**
- `total_ecnl_games` = N (between 8,000 and 40,000)
- `unverified_count` = N (same as total — all FALSE after column addition with FALSE default)
- `already_verified_count` = 0

**Stop conditions:**
- If this query errors with `column "ecnl_verified" does not exist` → the April 28 ALTER TABLE did not run. Flag to SENTINEL immediately. Do not proceed.
- If `total_ecnl_games` = 0 → the source/event_tier criteria are not matching any rows. Verify source values in the DB with: `SELECT DISTINCT source FROM games WHERE event_tier = 'ecnl' LIMIT 10;`. If different source values appear, update the criteria accordingly.
- If `already_verified_count > 0` → the column was pre-populated (unexpected). Investigate before running the UPDATE — do not overwrite existing TRUE values without understanding why they exist.

**Record:** total_ecnl_games count. This is the expected UPDATE row count in Section 3.

---

## Section 3: Backfill SQL

Run this after confirming the pre-backfill count is in range (Section 2).

**Run inside a transaction — do not run as a bare UPDATE.**

```sql
BEGIN;

-- ECNL ecnl_verified Backfill
-- Sets ecnl_verified = TRUE for all games sourced from ECNL ingestion paths
-- Run AFTER April 28 confirms ecnl_verified column exists
-- Run BEFORE ELO CP2 checkpoint on April 29
UPDATE games
SET ecnl_verified = TRUE
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
  AND ecnl_verified = FALSE;

-- Confirm row count before committing
SELECT COUNT(*) AS now_verified
FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
  AND ecnl_verified = TRUE;

-- If now_verified matches pre-backfill total_ecnl_games → COMMIT
-- If now_verified does not match → ROLLBACK and flag to SENTINEL
COMMIT;
```

**Expected UPDATE output:** `UPDATE N` where N matches `total_ecnl_games` from Section 2.

**Commit gate:**
- `now_verified` = `total_ecnl_games` from Section 2 → COMMIT
- `now_verified` ≠ `total_ecnl_games` (unexplained discrepancy) → ROLLBACK, flag to SENTINEL

**If rows affected >> total_ecnl_games:** The `AND ecnl_verified = FALSE` filter may have been omitted. This should not happen if the query is copy-pasted as written, but if it does: ROLLBACK immediately. The criteria would have been correct — this indicates an unexpected data state.

---

## Section 4: Post-Backfill Verification

Run immediately after the COMMIT to confirm full coverage.

```sql
-- Post-backfill confirmation — matches ELO's S4-4 query
SELECT
  event_tier,
  source,
  COUNT(*) AS game_count,
  SUM(CASE WHEN ecnl_verified = TRUE THEN 1 ELSE 0 END) AS verified_count,
  ROUND(100.0 * SUM(CASE WHEN ecnl_verified = TRUE THEN 1 ELSE 0 END) / COUNT(*), 1) AS pct_verified
FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
GROUP BY event_tier, source
ORDER BY game_count DESC;
```

**Expected result:** `pct_verified = 100.0` for all ECNL source/event_tier rows.

**Failure condition:** `pct_verified < 100.0` after the UPDATE committed — some rows were not updated. This likely means a third source value also contains ECNL games. Run: `SELECT DISTINCT source FROM games WHERE event_tier = 'ecnl';` to find any unlisted sources and re-run the backfill for those.

**File this output in the FORGE briefing.** ELO can use this as CP2 input directly — it is the same check as the S4-4 query in the Section 4 Query Package.

---

## Section 5: Execution Instructions for Presten

**Pre-conditions:**
1. April 28 is confirmed complete (April 29 Execution Checklist Pre-Flight Gate ✅)
2. Step 1 of April 29 Execution Checklist confirmed `ecnl_verified` column exists

**Execution sequence:**

1. Open psql session.
2. Run Section 2 verification query. Record `total_ecnl_games` count.
3. Confirm count is between 8,000 and 40,000. If outside this range, stop and contact FORGE before proceeding.
4. Copy-paste Section 3 SQL block (BEGIN through COMMIT) into psql. Execute.
5. Read the `UPDATE N` output. Confirm N matches `total_ecnl_games` from step 2.
6. Read the `now_verified` count from the SELECT inside the transaction. Confirm it matches `total_ecnl_games`.
7. If both match: the transaction auto-COMMIT ran. Confirm with: `SELECT COUNT(*) FROM games WHERE ecnl_verified = TRUE;` — should be N.
8. Run Section 4 post-backfill verification. Confirm `pct_verified = 100.0`.
9. File in FORGE briefing: "ecnl_verified backfill complete: [N] rows. pct_verified = 100.0%. ELO CP2 gate unblocked."

**Total time: < 10 minutes.**

---

## Section 6: Ongoing Population (New Games After April 28)

After this backfill covers existing games, new ECNL games ingested by the pipeline need `ecnl_verified = TRUE` at write time.

**Current status (requires confirmation):**  
The TGS scraper may or may not set `ecnl_verified = TRUE` on insert. This depends on whether the April 28 pipeline code update included write-time population or only the ALTER TABLE column addition.

**How to check:**  
After the May 1 pipeline first run, run:
```sql
SELECT ecnl_verified, COUNT(*) 
FROM games 
WHERE source IN ('tgs', 'tgs_athleteone') 
  AND event_tier = 'ecnl'
  AND created_at >= '2026-05-01'
GROUP BY ecnl_verified;
```

**If new May 1 games show `ecnl_verified = FALSE`:** The scraper is not setting the value at write time. Flag as a follow-up task: update the TGS scraper to set `ecnl_verified = TRUE` at insert for ECNL-classified games. This is required before the June 2026 ECNL migration — any games written after April 28 with `ecnl_verified = FALSE` will be incorrectly classified in the migration baseline.

**If new May 1 games show `ecnl_verified = TRUE`:** The pipeline code was updated to set the value at write time. No follow-up needed — new games are populating correctly.

---

## References

- `Infrastructure/April 28 Execution Log.md` — Step 2 (column addition) must be complete before this package runs
- `Infrastructure/April 29 — FORGE Execution Checklist.md` — Step 2 of that checklist references this package for escalation path
- `Infrastructure/April 29 — Section 4 Source Baseline Query Package.md` — S4-4 is the same check as Section 4 of this package
- `Infrastructure/Step 2b — ecnl_verified Backfill SQL Execution Package.md` — April 28 backfill execution (run on April 28, between Step 2 and Step 3)
- `Infrastructure/ECNL ecnl_verified Column — Population Plan.md` — column design and June 2026 dependency
- `Rankings/ECNL Migration Evidence Campaign.md` — CP2 gate this package unblocks

*FORGE — 2026-04-27*
