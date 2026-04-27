---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/ECNL ecnl_verified — Backfill SQL Package.md"
---

# Task: ECNL `ecnl_verified` Backfill SQL Package

## Context

April 28 adds the `ecnl_verified` column to the `games` table with a default of `FALSE`. This means after April 28, every existing ECNL game in the database will have `ecnl_verified = FALSE` — including games that ARE correctly sourced from the ECNL scraper and should be marked `TRUE`. The column addition does not populate it.

ELO's ECNL migration evidence campaign depends on `ecnl_verified`. Specifically, CP2 is a check of the `ecnl_verified` population rate — if the column exists but is entirely `FALSE`, CP2 will fail regardless of whether ECNL sourcing is working correctly. This is a false failure caused by missing backfill, not by a real data problem.

FORGE needs to write the SQL to populate `ecnl_verified = TRUE` for games that should be verified — before April 29 so ELO's CP2 isn't blocked by an empty column. The exact logic for "which games should have ecnl_verified = TRUE" needs to come from FORGE's understanding of how ECNL games are sourced (what field identifies ECNL-sourced games: `source_id`, `league_id`, `event_tier`, or a combination).

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/ECNL ecnl_verified — Backfill SQL Package.md`

---

### Section 1: ECNL Source Identification Logic

Before writing the backfill SQL, FORGE must document the source identification logic:

- Which field(s) in the `games` table (or related tables) identify a game as ECNL-sourced?
- Is ECNL data ingested via a distinct scraper with a `source_id` value, or is it identified by `event_tier`, `league_id`, `org_id`, or some combination?
- Is there a `sources` table or equivalent that can be JOIN'd to find ECNL-origin games?

Document the answer. If multiple approaches are viable (e.g., `event_tier IN ('ecnl', 'ecnl_regional_league')` OR `source_id = 3`), state which is most reliable and why.

---

### Section 2: Verification Query (Pre-Backfill)

Before running the backfill, confirm the scope:

```sql
-- Pre-backfill scope check
-- Replace [ECNL identification criteria] with the logic from Section 1
SELECT
  COUNT(*) AS total_ecnl_games,
  SUM(CASE WHEN ecnl_verified = FALSE THEN 1 ELSE 0 END) AS unverified_count,
  SUM(CASE WHEN ecnl_verified = TRUE THEN 1 ELSE 0 END) AS already_verified_count
FROM games
WHERE [ECNL identification criteria];
```

Expected result after April 28 column addition: `unverified_count` = `total_ecnl_games`, `already_verified_count` = 0.
If `already_verified_count > 0`: the column was pre-populated somehow — investigate before running backfill.

---

### Section 3: Backfill SQL

```sql
-- ECNL ecnl_verified Backfill
-- Sets ecnl_verified = TRUE for all games meeting ECNL source criteria
-- Run AFTER April 28 confirms ecnl_verified column exists
-- Run BEFORE ELO CP2 checkpoint on April 29

UPDATE games
SET ecnl_verified = TRUE
WHERE [ECNL identification criteria]
  AND ecnl_verified = FALSE;

-- Expected: N rows affected (where N = total_ecnl_games from verification query)
-- If 0 rows affected: re-run verification query — column may not exist or criteria may be wrong
-- If rows affected >> expected: criteria is too broad — stop, flag to SENTINEL
```

State clearly: this UPDATE should be run in a transaction (`BEGIN; UPDATE ...; SELECT COUNT(*) FROM games WHERE ecnl_verified = TRUE; COMMIT;`). If the post-UPDATE count is unexpected, `ROLLBACK`.

---

### Section 4: Post-Backfill Verification

After the UPDATE, confirm the result:

```sql
-- Post-backfill confirmation
SELECT
  event_tier,
  COUNT(*) AS game_count,
  SUM(CASE WHEN ecnl_verified = TRUE THEN 1 ELSE 0 END) AS verified_count,
  ROUND(100.0 * SUM(CASE WHEN ecnl_verified = TRUE THEN 1 ELSE 0 END) / COUNT(*), 1) AS pct_verified
FROM games
WHERE [ECNL identification criteria]
GROUP BY event_tier
ORDER BY game_count DESC;
```

Expected: `pct_verified = 100.0` for all ECNL event tiers.

This output is identical to ELO's S4-4 query result — if FORGE files this post-backfill verification, ELO can use it as CP2 input directly.

---

### Section 5: Execution Instructions for Presten

A plain-language execution block:

1. Confirm April 28 is complete (ecnl_verified column added — check via `\d games`)
2. Run Section 2 verification query — record total ECNL game count
3. Run Section 3 backfill in a transaction (BEGIN / UPDATE / post-count check / COMMIT)
4. Confirm: rows affected = count from step 2. If mismatch: ROLLBACK, flag to SENTINEL.
5. Run Section 4 post-backfill verification — confirm 100% verified
6. File output in briefing. Note: "ecnl_verified backfill complete: [N] rows. ELO CP2 gate unblocked."

Total time: < 10 minutes.

---

### Section 6: Ongoing Population (New Games)

After the backfill covers existing games, new ECNL games ingested by the pipeline need `ecnl_verified = TRUE` at ingestion time. FORGE should confirm:

- Does the ECNL scraper set `ecnl_verified = TRUE` on insert, or does it rely on a post-insert update?
- If the scraper does not set `ecnl_verified` on insert, FORGE notes this as a follow-up task: update the ECNL ingestion logic to set `ecnl_verified = TRUE` at write time. This is not required before April 29 (backfill covers existing games) but is required before the June ECNL migration.

State the answer and flag if a follow-up task is needed.

---

## Quality Standard

- Section 1 must commit to a specific identification criterion — "probably `event_tier`" is not acceptable. If FORGE is uncertain, run a discovery query and document the result.
- The backfill SQL must be wrapped in a transaction. No bare UPDATE without a rollback option.
- The post-backfill verification must produce an expected output that FORGE states in advance — so Presten can assess correctness without needing to interpret raw numbers.

## Why This Matters

ELO's CP2 gate (ECNL migration evidence campaign) checks `ecnl_verified` population rate. If FORGE does not file this backfill package, CP2 fails on April 29 with `pct_verified = 0%` — which is a false failure that delays the entire ECNL migration evidence campaign and potentially delays the June 1 go/no-go. Writing and executing this package before April 29 is a 10-minute task that prevents a multi-day downstream delay.

## References

- `task-2026-04-24-ecnl-dedup-verified-flag.md` — original task specifying the `ecnl_verified` column design
- `Infrastructure/April 28 Execution Log.md` — Step 2 (column addition) must be complete before this package runs
- `Rankings/ECNL Migration Evidence Campaign.md` — CP2 gate this package unblocks
- `Infrastructure/April 29 — Section 4 Source Baseline Query Package.md` — S4-4 is the same check as Section 4 of this package
