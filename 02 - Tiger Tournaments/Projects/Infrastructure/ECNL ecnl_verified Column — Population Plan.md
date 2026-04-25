---
title: ECNL ecnl_verified Column — Population Plan
tags: [infrastructure, ecnl, migration, database, schema, evo-draw, forge]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: ready — reviewed before April 28 session
task: task-2026-04-25-ecnl-verified-column-population-plan
---

# ECNL `ecnl_verified` Column — Population Plan

> [!info] Purpose
> April 28 adds `ecnl_verified BOOLEAN DEFAULT FALSE` to the `games` table. This plan defines how the column gets populated after deployment. Without this plan, every ECNL game stays `FALSE` indefinitely — the column exists but carries no signal. The ECNL Rating Continuity Spec and June 2 migration verification both depend on this flag meaning something accurate.

---

## Section 1: Population Trigger

**What makes a game `ecnl_verified = TRUE`?**

A game is ECNL-verified when **both** of the following are true:
1. `source IN ('tgs', 'tgs_athleteone')` — game was ingested from TGS AthleteOne, the current authoritative ECNL source (through June 2026)
2. `event_tier = 'ecnl'` — the game is classified as ECNL tier (not ECNL RL, not Pre-ECNL)

**After June 1 migration:** The trigger shifts to:
1. `source = 'gotsport'`
2. `event_tier = 'ecnl'`

Both pre- and post-migration conditions are appropriate: the flag tracks the ECNL tier classification, not the source platform. The source criterion is included to prevent false positives from games that might carry `event_tier = 'ecnl'` from incorrect parsing.

**This is pipeline-set, not manually assigned.** Presten does not need to mark individual games. The backfill SQL (Section 2) covers all existing games in one pass. The forward pipeline code change (Section 3) covers all future games automatically.

---

## Section 2: Backfill Plan for Existing Games

After April 28 adds the column, all ECNL games already in the database will have `ecnl_verified = FALSE` by default. The backfill sets them to `TRUE` in one SQL statement.

### Count query (run first — confirm scope before backfill):

```sql
SELECT COUNT(*) AS games_to_backfill
FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
  AND is_hidden = false;
```

Expected order of magnitude: **thousands** (ECNL has ~130 girls clubs + ~150 boys clubs, multiple seasons of history, ~91,000 ECNL games estimated in the system across both sources).

> [!note] Estimated scope
> The full ECNL game count in the DB is ~91,401 (per Strategic Insights vault). Not all will be from TGS exclusively — some may already be from GotSport if ECNL events appeared under GotSport org IDs. The count query above clarifies the actual scope before running.

### Backfill UPDATE (idempotent — safe to run twice):

```sql
UPDATE games
SET ecnl_verified = TRUE
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
  AND ecnl_verified = FALSE;
```

The `ecnl_verified = FALSE` condition makes this idempotent: re-running it on an already-backfilled database does nothing.

### Timing: Run as Step 2b during the April 28 session

Insert this as a step in the April 28 Execution Log **after** the `ALTER TABLE` (Step 2) and **before** the full rankings recompute (Step 3). Sequence:

1. Step 2: `ALTER TABLE games ADD COLUMN ecnl_verified BOOLEAN DEFAULT FALSE`
2. **Step 2b (this plan): Run count query → confirm scope → run backfill UPDATE → confirm with verification query (Section 4)**
3. Step 3: `node scripts/compute-rankings.js`

Running the backfill before the rankings recompute ensures the recompute sees accurate `ecnl_verified` values in whatever downstream queries reference this column.

---

## Section 3: Forward Pipeline Logic

After the backfill, newly ingested ECNL games must also get `ecnl_verified = TRUE`. This requires one code change in the ingestion pipeline.

### Current ingestion path (through June 2026)

TGS AthleteOne games are ingested via `tgs_scraper`. The scraper uses schema-adaptive insertion (reads `information_schema.columns` at startup). This means: once the column exists, the scraper will include it in its INSERT if it is listed in the scraper's column mapping.

**Required code change for `tgs_scraper`:**

In the TGS scraper's column mapping or INSERT construction, add:

```javascript
// When source = 'tgs' or 'tgs_athleteone' AND event_tier resolves to 'ecnl':
ecnl_verified: true
```

Specific implementation depends on where the TGS scraper constructs its INSERT payload. The pattern is: if `event_tier === 'ecnl'`, set `ecnl_verified = true`. Otherwise, leave it at the default (`FALSE`).

**This is a TODO for Presten's May 1 pipeline session** — scope it alongside the `run-daily.sh` and `get-active-team-ids.sh` implementation work.

### Post-June 1 migration (GotSport as ECNL source)

After the ECNL source shifts to GotSport, the `crawl-gotsport-api-parallel.js` scraper handles ECNL games. The same rule applies:

```javascript
// When ingesting via GotSport AND event_tier resolves to 'ecnl':
ecnl_verified: true
```

This should be implemented as part of the May 1 pipeline work, or at latest before `ecnl-transition-dedup.js` runs (the dedup script depends on ECNL games being correctly identified in the `games` table).

**Note:** No code change is needed if the pipeline logic can derive `ecnl_verified` from `event_tier = 'ecnl'` at query time. FORGE's recommendation is to store it at write time (for performance and auditability), not compute it on read.

---

## Section 4: Verification Query

Run after April 28 + backfill to confirm `ecnl_verified` is populated correctly.

```sql
-- Confirm: all TGS ECNL games (non-hidden) now have ecnl_verified = TRUE
SELECT
  ecnl_verified,
  COUNT(*) AS game_count
FROM games
WHERE source IN ('tgs', 'tgs_athleteone')
  AND event_tier = 'ecnl'
  AND is_hidden = false
GROUP BY ecnl_verified;
```

**Pass condition:** Only one row returned: `ecnl_verified = TRUE, count = [N]`. The `ecnl_verified = FALSE` row must not appear (or must show count = 0).

**Fail condition:** Any row with `ecnl_verified = FALSE` for TGS ECNL games. This means the backfill did not complete or there is a filter mismatch. Re-run the backfill UPDATE, then re-run this query.

Secondary check: confirm non-ECNL games are not falsely flagged:

```sql
-- Confirm: no false positives (non-ECNL games with ecnl_verified = TRUE)
SELECT COUNT(*) AS false_positives
FROM games
WHERE ecnl_verified = TRUE
  AND event_tier != 'ecnl';
```

Expected: 0. If > 0, investigate — likely an event_tier parsing issue, not a backfill problem.

---

## References

- `Infrastructure/April 28 Execution Log.md` — Step 2b should be added to track backfill execution
- `Rankings/ECNL Rating Continuity Spec.md` — downstream consumer of `ecnl_verified`
- `Infrastructure/ECNL Migration Pre-Flight Checklist — June 2026.md` — migration gate that uses `ecnl_verified`
- `task-2026-04-23-ecnl-migration-validation-script.md` — pre-migration snapshot; post-migration comparison uses `ecnl_verified` as a correctness signal
- `Infrastructure/Database Schema.md` — `games` table schema

*FORGE — 2026-04-25*
