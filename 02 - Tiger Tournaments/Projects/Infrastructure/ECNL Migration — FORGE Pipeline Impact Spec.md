---
title: "ECNL Migration — FORGE Pipeline Impact Spec"
tags: [infrastructure, ecnl, migration, pipeline, forge, sentinel, elo]
created: 2026-04-28
updated: 2026-04-28
author: FORGE
status: pre-staged — awaiting ELO option decision
task: task-2026-04-28-ecnl-migration-pipeline-impact-spec
due: 2026-05-05
active_option: TBD — update to A | B | C when ELO files decision
---

# ECNL Migration — FORGE Pipeline Impact Spec

> [!warning] Pre-Staged Document
> This spec is pre-staged for all three migration options. When ELO files their decision (expected by April 30), FORGE will mark the chosen option **ACTIVE** and update `status` to `in-progress`. Do not execute any section until the SENTINEL Authorization Gate (Section 6) is fully satisfied.

**Filed by:** FORGE — 2026-04-28  
**ELO decision expected:** April 30  
**SENTINEL authorization required before:** any production schema change  
**Migration target date:** June 2, 2026  
**DSS checkpoint:** May 9 readiness check; May 16 demo  

---

## Section 1: Migration Context

### What Is Changing

ECNL is restructuring its team identity model effective June 2, 2026. Post-restructure, ECNL team entities on TGS AthleteOne (Evo Draw's ECNL data source) will carry new identifiers. The game data pipeline must bridge old and new identities without losing historical game attribution or disrupting ongoing ingestion.

### What FORGE Owns

| Area | FORGE Responsibility |
|------|---------------------|
| Schema | All DDL changes to `games`, `teams`, `team_merges` tables; new tables if required by option |
| Ingestion | TGS scraper updates for post-June-2 team ID handling; `ecnl_verified` flag continuity |
| Dedup | `team_merges` rows for old→new ECNL team identity; dedup query correctness after migration |
| `ecnl_verified` flag | Confirm backfill complete before migration; verify flag correctness post-migration |
| Post-migration validation | File validation report before SENTINEL signs off on DSS readiness |

### What ELO Owns

| Area | ELO Responsibility |
|------|-------------------|
| Rating continuity | Pre-migration rating snapshot; team_id continuity spec; post-migration rating verification |
| `team_merges` context | Confirming which merges affect ECNL team entities; flagging any merge rows at risk |
| Option selection | ELO files decision by April 30; all three options analyzed in `task-2026-04-26-ecnl-migration-option-comparison-matrix.md` |
| Post-migration calibration | Running verification query set (per June 2026 ECNL Migration Verification Spec) |

### Decision Authority

- **ELO selects option** — files decision in `task-2026-04-26-ecnl-migration-option-selection-decision-gate.md`
- **FORGE implements** — pipeline impact per the chosen option section below
- **SENTINEL authorizes** — all production schema changes require explicit SENTINEL sign-off (Section 6)
- **Presten approves** — final authorization before any schema change executes in production

---

## Section 2: Option A — Soft Merge (History Preserved)

> **Status: STANDBY** — becomes ACTIVE when ELO selects Option A

### Schema Changes Required

**DDL complexity: LOW**

```sql
-- Add reference column to teams table (stores pre-restructure identity for audit)
ALTER TABLE teams ADD COLUMN ecnl_pre_restructure_team_id TEXT;

-- No changes to games table required
-- No new tables required
-- ecnl_verified column already exists (added April 28)
```

New team_merges rows required (no DDL — data insert only):
```sql
-- One row per restructured ECNL team: links old TGS team ID to new TGS team ID
-- FORGE inserts these rows once the old→new mapping is received from ECNL/TGS
INSERT INTO team_merges (team_id, merged_team_id, verified, source, notes)
VALUES ('[new_tgs_id]', '[old_tgs_id]', true, 'ecnl_restructure_2026', 'June 2 restructure mapping');
```

**Expected number of new team_merges rows:** One per restructured ECNL team entity. FORGE will determine count from ECNL's published restructure map when available.

---

### Ingestion Path Changes

- Pre-June-2: TGS scraper (`tgs_scraper` / `tgs_athleteone`) continues ingesting with existing team IDs. No change.
- Post-June-2: TGS AthleteOne returns new team entity IDs. New games arrive with new `home_team_id` / `away_team_id` values.
- **FORGE action:** Before June 2, insert team_merges rows linking old TGS IDs to new TGS IDs. The dedup and ranking engines already JOIN through `team_merges` — no code change required once merge rows are in place.
- The TGS scraper itself requires no code changes under Option A. New IDs are accepted as-is; team_merges handles identity continuity transparently.

---

### Dedup Rule Changes

- **No changes to dedup logic.** The dedup engine uses `team_merges` to resolve cross-platform identity. Adding new merge rows (old→new ECNL IDs) is sufficient.
- FORGE must verify: no existing `team_merges` rows create ambiguity when old→new ECNL merge rows are added (e.g., a GotSport team already merged to the old ECNL ID must be transitively linked to the new ID).
- Query to confirm before executing: `SELECT COUNT(*) FROM team_merges WHERE team_id IN ([old_ecnl_id_list]) OR merged_team_id IN ([old_ecnl_id_list])` — review all existing rows touching old ECNL IDs.

---

### ecnl_verified Flag Behavior

- **Historical games (pre-June-2):** Already backfilled via Step 2b. `ecnl_verified = TRUE` for all games where `source IN ('tgs', 'tgs_athleteone') AND event_tier = 'ecnl'`.
- **New games (post-June-2):** TGS scraper sets `ecnl_verified = TRUE` at write time (FM2 fix, deployed May 1). No change to flag logic required.
- **Flag continuity across migration:** Under Option A, game records are not rewritten — only team_merges rows are added. `ecnl_verified` values on existing rows are untouched.

---

### Implementation Steps

1. Receive ECNL old→new team ID mapping from TGS/ECNL (FORGE requests this immediately on ELO decision)
2. Confirm `ecnl_verified` backfill complete: `SELECT COUNT(*) FROM games WHERE source IN ('tgs', 'tgs_athleteone') AND ecnl_verified = FALSE` must return 0
3. ELO: Run pre-migration rating snapshot (FORGE confirms ELO has done this before proceeding)
4. FORGE: Update `teams` table — set `ecnl_pre_restructure_team_id` for all restructuring ECNL teams
5. FORGE: Insert `team_merges` rows for all old→new ECNL team ID pairs (one per team)
6. Verify merge correctness: sample query confirms new games for restructured teams link to historical records
7. First post-June-2 TGS scrape: validate new games are attributed to the correct team identity
8. ELO: Run post-migration verification query set
9. FORGE: File post-migration validation report (row counts, ecnl_verified counts, ingestion test result)

**Time estimate: 2–3 days from authorization to first validated run.**
- Day 1: Receive ID mapping → update teams table → insert team_merges rows
- Day 2: First post-June-2 scrape + FORGE validation
- Day 3: ELO verification + FORGE report filed

---

### Risk

- **ID mapping completeness:** If TGS/ECNL does not publish a clean old→new team ID map, FORGE cannot create complete team_merges rows before June 2. Teams without a merge row appear as new entities in rankings. **Mitigation:** FORGE requests the mapping the day ELO files the decision. If mapping is delayed past May 26, FORGE flags to SENTINEL.
- **Transitive merge conflicts:** Existing cross-source merges (ECNL team linked to GotSport team via old ECNL ID) may need transitivity audit. **Mitigation:** Run merge audit query (Step above) before inserting new rows.
- **Lowest-risk option** of the three. No game record rewriting, no DLL changes beyond one column add, no disruption to active ingestion.

---

## Section 3: Option B — Hard Entity Replacement

> **Status: STANDBY** — becomes ACTIVE when ELO selects Option B

> [!warning] CRITICAL Dependency
> FORGE cannot begin Step 4 (game record UPDATE) until ELO has confirmed rating continuity pre-migration snapshot is complete AND ELO's team entity model has been updated. Executing the DB UPDATE before ELO clears this gate will break rating lookups for ECNL teams.

### Schema Changes Required

**DDL complexity: MEDIUM**

```sql
-- No new tables required
-- No new columns required
-- Changes are data updates, not schema changes

-- Step 4: Update all historical ECNL game records (run in transaction)
BEGIN;
UPDATE games
SET home_team_id = [new_tgs_id]
WHERE home_team_id = [old_tgs_id]
  AND source IN ('tgs', 'tgs_athleteone');

UPDATE games
SET away_team_id = [new_tgs_id]
WHERE away_team_id = [old_tgs_id]
  AND source IN ('tgs', 'tgs_athleteone');
-- Repeat for each old→new team ID pair
COMMIT;

-- Step 5: Update team_merges
UPDATE team_merges
SET team_id = [new_tgs_id]
WHERE team_id = [old_tgs_id];

UPDATE team_merges
SET merged_team_id = [new_tgs_id]
WHERE merged_team_id = [old_tgs_id];
```

**Teams table:** Mark old ECNL team records as retired post-replacement:
```sql
UPDATE teams SET is_active = FALSE WHERE team_id IN ([old_ecnl_id_list]);
```

---

### Ingestion Path Changes

- All historical ECNL game records: `home_team_id` and `away_team_id` updated to new entity IDs. After replacement, the DB contains only new IDs for all ECNL games.
- Forward ingestion (post-June-2): TGS scraper ingests new team IDs natively — no scraper changes required.
- **FORGE must execute the UPDATE in a single transaction** with an explicit ROLLBACK plan. Size estimate: all ECNL game rows touching restructured team IDs (FORGE will query count pre-execution).

---

### Dedup Rule Changes

- **All `team_merges` rows referencing old ECNL team IDs must be updated** (both `team_id` and `merged_team_id` columns).
- Cross-source merges at risk: any row linking an old ECNL team ID to a GotSport team ID. If FORGE updates the `team_merges.team_id` column but not the corresponding GotSport-side merge, the cross-source link is broken.
- Pre-execution audit required: `SELECT * FROM team_merges WHERE team_id IN ([old_list]) OR merged_team_id IN ([old_list])` — review all rows, confirm update script covers both sides.

---

### ecnl_verified Flag Behavior

- Historical games: `ecnl_verified = TRUE` on all rows where source IN ('tgs', 'tgs_athleteone'). The UPDATE in Step 4 touches `home_team_id` / `away_team_id` columns only — `ecnl_verified` is not modified.
- Post-update verification: `SELECT COUNT(*) FROM games WHERE source IN ('tgs', 'tgs_athleteone') AND ecnl_verified = FALSE` must return 0. If the UPDATE inadvertently cleared any rows, re-run the Step 2b backfill SQL package.
- New games (post-June-2): `ecnl_verified = TRUE` set by TGS scraper at write time (FM2 fix).

---

### Implementation Steps

1. ELO: Run rating continuity pre-migration snapshot AND confirm team entity model updated (FORGE waits for ELO gate — do not proceed without this)
2. FORGE: Pre-migration snapshot: `SELECT home_team_id, COUNT(*) FROM games WHERE source IN ('tgs','tgs_athleteone') GROUP BY home_team_id` (captures distribution before rewrite)
3. FORGE: Confirm `ecnl_verified` backfill complete
4. FORGE: Execute UPDATE statements for all old→new ECNL team ID pairs (home_team_id, away_team_id) — run in transaction, confirm row counts before COMMIT
5. FORGE: Update `team_merges` for all rows touching old ECNL team IDs
6. FORGE: Mark old ECNL team records inactive in `teams` table
7. Verify `ecnl_verified` count unchanged post-UPDATE
8. ELO: Run post-migration verification query set
9. Recompute rankings (`node scripts/compute-rankings.js`)
10. FORGE: File post-migration validation report

**Time estimate: 4–5 days from authorization to first validated run.**
- Day 1: ELO gate confirmed + FORGE pre-migration snapshot
- Day 2: Execute UPDATE transactions + team_merges update + ecnl_verified verification
- Day 3: ELO verification
- Day 4: Rankings recompute + validation
- Day 5: FORGE report filed

---

### Risk

- **Critical execution sequencing:** ELO must complete rating continuity work before FORGE executes Step 4. If out of order, rating lookups fail until ELO catches up. This is a non-negotiable gate.
- **UPDATE volume:** Depending on total ECNL game count, this may be thousands of UPDATE statements. FORGE will test with a single team pair first, confirm output, then execute full batch.
- **Cross-source merge invalidation:** Highest dedup risk of all options. FORGE must audit all team_merges touching old ECNL IDs before execution.
- **Rollback complexity:** Once committed, reversing requires re-running the UPDATE in the opposite direction. FORGE will retain the old→new mapping file to enable rollback if needed.
- **Irreversible if executed without ELO gate:** Highest-consequence failure mode of any option. SENTINEL must confirm ELO gate is cleared before authorizing FORGE execution.

---

## Section 4: Option C — Parallel Key with Lookup Table

> **Status: STANDBY** — becomes ACTIVE when ELO selects Option C

### Schema Changes Required

**DDL complexity: HIGH**

**New table: `ecnl_team_key_map`**

```sql
CREATE TABLE ecnl_team_key_map (
    ecnl_old_id     TEXT        NOT NULL,
    ecnl_new_id     TEXT        NOT NULL,
    team_name       TEXT,
    effective_date  DATE        NOT NULL DEFAULT '2026-06-02',
    notes           TEXT,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (ecnl_old_id)
);

-- Index for reverse lookup (new_id → old_id)
CREATE INDEX idx_ecnl_key_map_new ON ecnl_team_key_map (ecnl_new_id);
```

**New column on `games` table:**

```sql
ALTER TABLE games ADD COLUMN ecnl_canonical_id TEXT;

-- Backfill: set ecnl_canonical_id = home_team_id for all pre-June-2 ECNL games
-- (canonical = old ID for historical games; canonical = new ID for post-June-2 games)
UPDATE games
SET ecnl_canonical_id = home_team_id
WHERE source IN ('tgs', 'tgs_athleteone')
  AND match_date < '2026-06-02'
  AND ecnl_canonical_id IS NULL;
```

---

### Ingestion Path Changes

- Historical games: `ecnl_canonical_id` = old TGS team ID (backfilled per above). `home_team_id` / `away_team_id` unchanged.
- Post-June-2 games: TGS scraper must write new team ID to `home_team_id` / `away_team_id` AND write the new ID to `ecnl_canonical_id`.
- TGS scraper code change required:
  ```javascript
  // In TGS scraper INSERT payload, add:
  ecnl_canonical_id: (source in ['tgs', 'tgs_athleteone']) ? home_team_id : null,
  ```
- Ranking and dedup queries must JOIN `ecnl_team_key_map` when spanning historical and current records:
  ```sql
  -- Example: find all games for a team across old/new identity
  SELECT g.* FROM games g
  JOIN ecnl_team_key_map m ON g.ecnl_canonical_id IN (m.ecnl_old_id, m.ecnl_new_id)
  WHERE m.ecnl_old_id = '[search_team_id]' OR m.ecnl_new_id = '[search_team_id]';
  ```

---

### Dedup Rule Changes

- `team_merges` rows: No changes required to existing rows under Option C. Historical merges reference old ECNL IDs; those IDs remain in `home_team_id` / `away_team_id` for historical games.
- Cross-team queries must use canonical ID resolution: `COALESCE(ecnl_canonical_id, home_team_id)` as the join key for ECNL teams spanning the restructure date.
- New dedup check required: Confirm `ecnl_team_key_map` has no ambiguous mappings (one old_id → two new_ids). Query: `SELECT ecnl_old_id, COUNT(DISTINCT ecnl_new_id) FROM ecnl_team_key_map GROUP BY ecnl_old_id HAVING COUNT(DISTINCT ecnl_new_id) > 1` must return 0 rows.

---

### ecnl_verified Flag Behavior

- Historical games: `ecnl_verified = TRUE` for all ECNL historical games (backfill complete). Adding `ecnl_canonical_id` column does not affect the flag.
- Post-June-2 games: `ecnl_verified = TRUE` set by TGS scraper at write time. No change.
- Lookup table has no interaction with `ecnl_verified` flag logic.

---

### Implementation Steps

1. Finalize `ecnl_team_key_map` schema — ELO must confirm that the canonical_id key structure matches their rating model (ELO's team identity key must align with FORGE's canonical_id for rating lookups to work)
2. Execute DDL: `CREATE TABLE ecnl_team_key_map` + `CREATE INDEX` + `ALTER TABLE games ADD COLUMN ecnl_canonical_id`
3. Obtain old→new team ID mapping from TGS/ECNL
4. Populate `ecnl_team_key_map`: `INSERT` all known old→new pairs
5. Backfill `games.ecnl_canonical_id` for all pre-June-2 ECNL games (UPDATE per the DDL section above)
6. TGS scraper code update: write `ecnl_canonical_id` for post-June-2 games
7. Update ranking and dedup query logic to handle canonical_id resolution (FORGE audits all queries that reference ECNL team IDs)
8. QA: Run a sample team history query spanning pre- and post-June-2 games; confirm correct attribution
9. First post-June-2 TGS scrape: validate `ecnl_canonical_id` is set correctly for new games
10. ELO: Run post-migration verification query set
11. FORGE: File post-migration validation report

**Time estimate: 6–8 days from authorization to first validated run.** (Highest of all three options)
- Days 1–2: DDL execution + ID mapping population + backfill
- Days 3–4: Scraper update + query logic audit + QA
- Day 5: First post-June-2 scrape + FORGE validation
- Day 6: ELO verification
- Days 7–8: FORGE report + buffer for anomaly resolution

---

### Risk

- **Query complexity:** Every existing query that references ECNL team IDs must be audited and potentially updated to handle the dual-key architecture. Missing any query produces incorrect attribution for ECNL teams post-June-2.
- **ELO alignment required:** If ELO does not adopt `ecnl_canonical_id` as their team identity key for post-June-2 ratings, rating lookups for ECNL teams will reference different keys than FORGE's game records — ratings and games become disconnected. This must be confirmed before Step 2.
- **Ongoing maintenance burden:** The lookup table must be maintained for as long as historical games (referenced by old IDs) coexist with new games (referenced by new IDs). If team identities change again in a future ECNL restructure, the table must be extended.
- **Highest implementation complexity.** Recommended only if ELO determines that Options A and B both have unacceptable rating continuity risk.

---

## Section 5: Cross-Option Shared Requirements

Regardless of option chosen, FORGE must complete the following before migration execution:

- [ ] **ecnl_verified backfill confirmed complete** — `SELECT COUNT(*) FROM games WHERE source IN ('tgs', 'tgs_athleteone') AND ecnl_verified = FALSE` returns 0
- [ ] **Pre-migration snapshot filed** — FORGE runs and saves game count and team_id distribution for all ECNL records before any schema change
- [ ] **ELO pre-migration rating snapshot confirmed** — ELO confirms this is complete before FORGE proceeds with any option
- [ ] **ECNL old→new team ID mapping received** — from TGS/ECNL; FORGE cannot build merge rows or lookup table without this
- [ ] **FORGE pipeline impact section for chosen option reviewed by SENTINEL** — full spec review, not just the summary
- [ ] **Post-migration validation query set ready** — from ELO's June 2026 Migration Verification Spec

---

## Section 6: SENTINEL Authorization Gate

**No production schema change executes under any option until all 4 boxes are checked:**

- [ ] ELO decision filed and acknowledged (option selected, FORGE notified)
- [ ] FORGE pipeline impact spec for chosen option reviewed by SENTINEL (this document, chosen option section only)
- [ ] ELO rating continuity pre-migration snapshot confirmed complete
- [ ] **Presten authorization given** — explicit approval before any DDL or UPDATE statement runs in production

**FORGE does not touch the production schema until all 4 boxes are checked.**

If ELO files decision on April 30: FORGE expects SENTINEL review by May 2. That leaves 31 days to implementation before June 2 — sufficient for any option's time estimate.

If ELO decision is delayed past May 9 (DSS readiness check): FORGE flags to SENTINEL. A May 9+ decision compresses Option C's 6–8 day window to < 2 weeks before June 2 — feasible but no longer comfortable.

---

## Section 7: Post-Migration Validation

FORGE files a post-migration validation report within 24 hours of the first successful post-June-2 TGS scrape. Report covers:

| Check | Pass Criteria |
|-------|--------------|
| Row counts — affected teams before/after | Counts match expected (no unexpected deletions or duplicates) |
| `ecnl_verified` flag counts | Count of TRUE rows unchanged or increased (backfill + new games); 0 FALSE rows for ECNL source games |
| Team identity continuity | Historical games for restructured teams correctly link to new team identity via chosen mechanism (team_merges / entity replacement / canonical_id) |
| Ingestion test | One ECNL game ingested post-June-2 shows correct team_id assignment and `ecnl_verified = TRUE` |
| Ranking recompute | Rankings recomputed cleanly after migration; no structural anomalies in top-100 ECNL team positions |
| ELO verification | ELO post-migration verification query set returns expected results |

Any anomaly blocks SENTINEL from signing the post-migration validation until resolved.

---

## References

- `task-2026-04-26-ecnl-migration-option-comparison-matrix.md` — ELO's option analysis (scores each option against 6 criteria)
- `task-2026-04-26-ecnl-migration-evidence-collection.md` — decision evidence
- `task-2026-04-26-ecnl-migration-option-selection-decision-gate.md` — ELO decision gate (FORGE watches for update)
- `task-2026-04-26-ecnl-verified-ingestion-path-audit.md` — current ECNL ingestion path (FORGE audit)
- `task-2026-04-24-june2-ecnl-migration-verification-spec.md` — ELO verification query set
- `task-2026-04-24-ecnl-rating-continuity-spec.md` — ELO continuity requirement FORGE must coordinate with
- `Infrastructure/Database Schema.md` — games, teams, team_merges table structure
- `Infrastructure/Step 2b — ecnl_verified Backfill SQL Execution Package.md` — ecnl_verified backfill SQL

*FORGE — 2026-04-28*
