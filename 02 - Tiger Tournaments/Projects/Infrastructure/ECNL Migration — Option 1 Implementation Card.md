---
title: ECNL Migration — Option 1 Implementation Card
tags: [infrastructure, ecnl, migration, forge, option1, no-retag]
created: 2026-04-29
updated: 2026-04-29
author: FORGE
status: ready-to-execute — pending SENTINEL authorization
linked_task: task-2026-04-29-ecnl-option1-forge-implementation-card
trigger: "SENTINEL authorizes Option 1 (expected: 2026-04-30 EOD)"
---

# ECNL Migration — Option 1 Implementation Card

> **One-page action card. Execute only after SENTINEL authorizes Option 1 (No Re-tag).**
> Authorization expected: 2026-04-30 EOD
> ELO decision brief: `Rankings/ECNL Migration Option — Decision Brief.md`
> Full pipeline impact reference: `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md` (Option A section)

---

## What Option 1 Means for FORGE

- **No schema changes.** Option 1 requires no ALTER TABLE statements — no `ecnl_legacy` tier column, no new tables, no column additions to `games` or `teams`. DDL complexity: zero.
- **No game record re-labeling.** FORGE does not run any UPDATE SQL on `home_team_id`, `away_team_id`, or `event_tier` fields. All historical records are untouched.
- **Pipeline ingestion continues unchanged on June 1.** The TGS AthleteOne scraper runs the same code path on migration day as it does today. No scraper code changes required at the migration boundary.
- **FORGE's active work is team_merges and monitoring.** Once ECNL's old→new team ID mapping is received from TGS/ECNL, FORGE inserts `team_merges` rows linking old TGS IDs to new TGS IDs. The dedup and ranking engines already traverse `team_merges` — no query rewrites needed.
- **Config adjustments if source identifiers shift.** If ECNL migrated teams change their TGS AthleteOne org or club identifier post-June 1, FORGE updates affected config entries within 1 session. Monitoring (below) surfaces this within 24 hours of the first post-migration scrape.

---

## Implementation Timeline

| Date | FORGE Action | Trigger |
|------|-------------|---------|
| **2026-04-30 EOD** | Authorization received from SENTINEL/Presten | SENTINEL files decision |
| **2026-05-02** | Request ECNL old→new team ID mapping from TGS/ECNL | Day after authorization |
| **2026-05-26 (gate)** | Mapping received + `team_merges` rows drafted | If delayed past this date, FORGE flags to SENTINEL — June 2 window compresses |
| **2026-05-28** | `team_merges` rows reviewed and ready to insert (3 days before migration) | Mapping received |
| **2026-05-29** | Confirm `ecnl_verified` backfill complete: `SELECT COUNT(*) FROM games WHERE source IN ('tgs','tgs_athleteone') AND ecnl_verified = FALSE` = 0 | Pre-migration gate |
| **2026-06-01** | Pre-migration snapshot: save game count + team_id distribution for all ECNL records | Standard FORGE pre-execution snapshot |
| **2026-06-01** | Insert `team_merges` rows for all old→new ECNL team ID pairs | Must complete before first post-migration TGS scrape |
| **2026-06-01** | Migration day monitoring (see below) | ECNL migration executes |
| **2026-06-02 + 24 hrs** | File ECNL migration ingestion confirmation report | Post-migration first TGS scrape |
| **2026-06-04** | File 3-day post-migration source health check | Pipeline stability confirmation |

---

## What FORGE Monitors on June 1

- **Game count continuity for ECNL teams:** Compare game counts for restructured ECNL teams between the pre-migration snapshot and the first post-migration TGS scrape. Count should increase or hold steady — never drop for active teams.
- **No orphaned team records:** After first post-June-2 TGS scrape, run `SELECT COUNT(*) FROM games WHERE source IN ('tgs','tgs_athleteone') AND home_team_id NOT IN (SELECT team_id FROM teams)` — must return 0. Orphaned records mean new TGS IDs arrived without a corresponding `teams` row; FORGE inserts the missing `teams` row and adds the `team_merges` link.
- **Source identifier integrity:** Confirm TGS AthleteOne scraper resolves ECNL team queries successfully post-migration — no 404 or entity-not-found errors for migrated teams. Any new error code introduced post-June-1 is flagged to SENTINEL same day.
- **ecnl_verified flag continuity:** `SELECT COUNT(*) FROM games WHERE source IN ('tgs','tgs_athleteone') AND ecnl_verified = FALSE AND ingested_at >= '2026-06-01'` must return 0. New ECNL games arriving after migration must still receive `ecnl_verified = TRUE` at write time (FM2 fix, deployed May 1).

---

## If Migration Fails (Option 1 Fallback)

If the first post-June-2 TGS scrape shows ingestion failures for ECNL migrated teams — defined as 0 new games ingested for teams active in the prior 30 days, or a 50%+ drop in ECNL game volume relative to the pre-migration baseline — FORGE immediately pauses ECNL team dedup operations, files a SENTINEL queue item (priority: urgent), and notifies ELO that CP1 and CP2 milestone timing is affected. FORGE does not attempt schema changes or config edits without SENTINEL authorization. Escalation path: FORGE files alert → SENTINEL reviews within 4 hours → Presten and ELO diagnose whether the failure is a team ID mapping gap (FORGE-fixable), a TGS AthleteOne platform issue (out of FORGE scope), or a config entry problem (FORGE-fixable within 1 hour).

---

## Handoff to ELO

When FORGE confirms migration ingestion is stable (June 2 + 24 hrs), FORGE files the ingestion confirmation at `Infrastructure/ECNL Migration — Post-June-2 Ingestion Confirmation.md` and signals to ELO in the June 2 briefing with this language:

> **ECNL Migration Ingestion Confirmed — [Date]:** First post-migration TGS scrape complete. [N] games ingested from restructured ECNL teams. `ecnl_verified` = TRUE confirmed for all new records. `team_merges` rows in place for [N] restructured team pairs. FORGE pipeline is stable. ELO: CP1 and CP2 can proceed.

**What ELO needs from FORGE for CP1 and CP2:**
- Confirmation that the first post-migration TGS scrape ingested games from expected ECNL teams (no silent gaps)
- Confirmation that `ecnl_verified = TRUE` for all new ECNL records post-migration
- The `team_merges` row list used for this migration (so ELO can cross-check against their rating model's team ID references)
- Pre-migration vs. post-migration game count snapshot to verify no historical records were modified

---

## References

- `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md` — full pipeline impact analysis (Option A = implementation basis for Option 1)
- `Infrastructure/ECNL Migration Pre-Flight Checklist — June 2026.md` — comprehensive pre-migration checklist
- `Infrastructure/Step 2b — ecnl_verified Backfill SQL Execution Package.md` — ecnl_verified backfill (must be confirmed complete before migration)
- `Rankings/ECNL Migration Option — Decision Brief.md` — ELO decision document recommending Option 1

*FORGE — 2026-04-29*
