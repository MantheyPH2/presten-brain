---
title: ECNL Migration — Option A Implementation Prep Package
tags: [ecnl, migration, option-a, elo, calibration, june2]
created: 2026-04-28
author: ELO
status: pre-staged — activate implementation steps only after Presten selects Option A
---

# ECNL Migration — Option A Implementation Prep Package

> **ELO filed Option A recommendation 2026-04-28.** This package pre-stages implementation so execution begins same-session as Presten's decision. If Presten selects Option B: void this document with note "Option B selected — [date]."

---

## Section 1: What Option A Means

**Option A:** When ECNL teams migrate from old ECNL league to new ECNL RL structure (effective June 2), their existing ELO ratings carry forward unchanged. No reset. The `ecnl_verified` flag on games from the old structure is retained. Games played under ECNL RL after June 2 begin accumulating normally from the carried-forward rating baseline.

**What does NOT happen:** No rating reset to 1000. No blending formula. No "migration bonus."

**Calibration implication:** Girls ECNL and ECNL RL ratings will be continuous. The June 2 league-structure change is invisible to the rankings — teams keep their spot.

---

## Section 2: Pre-Conditions

All must be met before Option A activates:

| Pre-condition | Status | How to Verify |
|--------------|--------|---------------|
| `ecnl_verified BOOLEAN DEFAULT FALSE` column exists in production | Pending April 28 session confirmation | psql: `SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'games' AND column_name = 'ecnl_verified';` |
| ecnl_verified backfill SQL run: historical ECNL games flagged correctly | Pending April 28/29 | `SELECT COUNT(*) FROM games WHERE ecnl_verified = TRUE` — confirm > 0 and in expected range |
| ECNL CP1 staleness check: PASS (ECNL RL data within 30 days) | Pending April 29 ELO session | `Rankings/ECNL Migration — CP1 CP2 Checkpoint Results.md` |
| ECNL CP2 coverage check: PASS (ecnl_verified ≥ 70% of ECNL-eligible teams) | Pending April 29 | Same checkpoint document |
| FORGE ECNL migration pipeline impact confirmed (Option A path) | Complete ✅ | `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md` |

ELO fills status column after April 29 session.

---

## Section 3: Implementation Steps — Option A

### Step 1: Confirm ecnl_verified backfill complete (ELO + FORGE)

- **Owner:** FORGE runs backfill; ELO confirms counts
- **SQL verification:** `SELECT COUNT(*) FROM games WHERE ecnl_verified = TRUE` → expected range from CP1 results
- **Deadline:** Must complete before June 2

### Step 2: Update League Hierarchy — ECNL RL entries (ELO)

- ECNL RL leagues must have calibration values matching current ECNL values
- **Source:** `League Hierarchy` vault note — Section for ECNL leagues
- **Action:** Add or update `ecnl_rl` entries with same tier values as outgoing ECNL entries
- **Verification:** Spot check 5 ECNL RL leagues against ECNL counterparts

### Step 3: Confirm team_merges for ECNL → ECNL RL team identity (ELO)

- Some clubs change their GotSport org-ID when migrating to ECNL RL
- **Source:** `Rankings/Team Merges Audit — Presten Execution Package.md` — run ECNL-specific merge check
- **Action:** Any team with old ECNL ID that maps to new ECNL RL ID must have a `team_merge` entry
- **Deadline:** Before June 2 first ECNL RL games ingest

### Step 4: Pre-migration rating snapshot (ELO)

- **Date:** June 1 (day before migration)
- **Action:** ELO files `Rankings/ECNL Pre-Migration Rating Snapshot.md` with top-100 ECNL girls teams and their ratings
- **Purpose:** Audit trail; if post-June-2 ratings diverge unexpectedly, compare against snapshot

### Step 5: Post-migration verification (ELO, within 1 week of June 2)

Run 3 spot-check queries:
1. Confirm top-10 ECNL teams retained ratings ± 5 pts post-migration
2. Confirm no ECNL team has rating reset to 1000
3. Confirm ECNL RL games ingesting with correct league tier weight

---

## Section 4: FORGE Coordination Required

| Item | FORGE Action | Timing |
|------|-------------|--------|
| ecnl_verified column confirmed | FORGE confirms from April 28 session output | April 28/29 |
| ECNL RL org-IDs in GotSport config | FORGE verifies ECNL RL org-IDs are in crawl config | Before June 2 |
| ECNL migration day pipeline check | FORGE monitors June 2 pipeline run for ECNL RL game ingestion | June 2 |

---

## Section 5: If Presten Selects Option B Instead

If Presten selects Option B (rating reset): this package is superseded. ELO reference: `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` covers Option B implications. FORGE impact spec covers both paths.

File a note: "Option B selected — this package voided as of [date]."

---

## Section 6: Open Questions (ELO answers before April 30)

1. Does ECNL RL use the same GotSport org-IDs as existing ECNL? *(Check with FORGE — FORGE has org-ID master reference)*
2. Are there ECNL RL leagues not yet in League Hierarchy that need calibration values before June 2?
3. Does ecnl_verified apply to both boys and girls ECNL games, or girls only?

*ELO flags answers to SENTINEL in next briefing or via Queue.*

---

*ELO — 2026-04-28*

## References

- `Rankings/ECNL Migration — ELO Option Recommendation.md` — ELO's Option A recommendation
- `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md`
- `Rankings/ECNL Migration — ELO-FORGE Handoff Checklist.md`
- `Rankings/ECNL Migration — CP1 CP2 Checkpoint Results.md`
