---
title: ECNL Rating Continuity Spec — June 2026
aliases: [ECNL Rating Continuity, Rating Orphan Spec]
tags: [rankings, migration, ecnl, rating-continuity, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: ELO Agent
status: ready
task: task-2026-04-24-ecnl-rating-continuity-spec
---

# ECNL Rating Continuity Spec — June 2026

> This spec addresses a structural dependency in the [[June 2 ECNL Migration Verification Spec]]: the June 2 morning checks assume Elo ratings are preserved through the June 1 ECNL platform migration. This document validates that assumption, identifies the failure scenario, and adds Check 6 (Rating Orphan Detection) to the June 2 spec.
>
> **FORGE coordination note:** Section 4 of this document contains an explicit requirement for FORGE's `Infrastructure/ECNL Migration Pre-Flight Checklist — June 2026.md`. The Section 3.2 placeholder in that checklist should reference this spec.

---

## Section 1: Current Rating Storage Model

### Where ratings are stored

From [[Database Schema]] and [[Ranking Engine]]:

**Ratings are stored in the `rankings` table, keyed by `team_id`.** The `rankings` table stores: team ID, Elo rating, rank position, age group, gender, and season. It is the output table written by `compute-rankings.js` after each full recompute.

### How merges interact with the Elo computation

The Ranking Engine processes merges in **Phase 1**, before any game processing occurs. Phase 1 loads all verified (`verified = true`) team merge records from the `team_merges` table. These records map `merged_team_id` → `canonical_team_id`.

In Phase 3 (Elo Computation), games are processed for all teams. Based on the Phase 1 merge map, any game where `home_team_id` or `away_team_id` matches a `merged_team_id` is attributed to the corresponding `canonical_team_id`. **The merge resolution happens at computation time via the in-memory merge map — not by rewriting game records in the database.**

**This is Option B: JOIN at query time (in-memory merge resolution during computation).**

> "Ratings are stored in `rankings.team_id`. When a merge is written to `team_merges`, game records are NOT re-attributed at write time. Instead, the Elo computation loads the verified merge map at startup (Phase 1) and resolves all game attributions to the canonical team during Phase 3. The `ecnl_transition` merge type is treated identically to all other merge types — Phase 1 loads it as long as `verified = true`."

### What `verified` controls

The `verified` flag is the critical gate. Phase 1 loads **only** records where `verified = true` (18,684 of 27,820 total merges as of April 2026). An `ecnl_transition` record with `verified = false` or NULL is **invisible to the ranking engine**. The canonical team gets no game history from the merged team; its rating initializes at the Glicko default (~1000).

---

## Section 2: Failure Scenario Analysis

**Rating continuity determination: PRESERVED — conditionally.**

Continuity is preserved if and only if every `ecnl_transition` merge record written by the June 1 migration script has `verified = true` before the next `compute-rankings.js` run.

**The failure scenario:**

> "The June 1 ECNL migration script writes `ecnl_transition` records to `team_merges`. If any of these records have `verified = false` or NULL, Phase 1 of the ranking engine skips them. On the next pipeline run, the new GotSport canonical ID has no game history — its rating initializes at ~1000. A team that was rated 1450 (Silver) before the migration now appears at 1000 (Red/Blue boundary). This is a silent failure: the rating doesn't error, it just resets."

**The FORGE dedup fix (Patch 1 from June 1 Risk Assessment) specifically addresses this.** The fix ensures that `ecnl-transition-dedup.js` inserts all `ecnl_transition` records with `verified = true`. Check 2B in the June 2 verification spec catches any failure of this fix.

**Residual risk:** Check 2B in the June 2 spec only verifies that verified counts are non-zero. If the dedup script runs partially — inserting some records correctly and failing silently on others — Check 2B passes but some teams are still at risk. **This is the gap that Check 6 (below) closes.**

**Scope of risk:** Not ECNL teams broadly — only teams whose `canonical_team_id` changes in the migration (i.e., teams migrating from TGS IDs to GotSport IDs). Teams already on GotSport are unaffected.

---

## Section 3: Check 6 — Rating Orphan Detection

The following check must be added to `Rankings/June 2 ECNL Migration Verification Spec.md` after Check 2E.

This check scans ALL ECNL-migration-involved teams for the rating reset signature — not just those in the 100-team spot-check list.

### Check 6: Rating Orphan Detection (ECNL Teams) — add to June 2 spec

```sql
-- Check 6: ECNL teams with ecnl_transition merges showing near-initialization ratings
-- Signature of a rating orphan: involved in ecnl_transition AND rating near 1000
-- Run for U13G, U14G; adapt for other age groups if time allows

SELECT
  t.name,
  r.rating::int AS current_rating,
  r.rank AS current_rank,
  r.age_group,
  r.gender,
  COUNT(tm.id) AS ecnl_transition_merges
FROM rankings r
JOIN teams t ON t.id = r.team_id
JOIN team_merges tm ON (tm.canonical_team_id = r.team_id OR tm.merged_team_id = r.team_id)
WHERE tm.merge_type = 'ecnl_transition'
  AND r.age_group IN ('U13', 'U14')
  AND r.gender = 'F'
  AND r.rating BETWEEN 950 AND 1050
GROUP BY t.name, r.rating, r.rank, r.age_group, r.gender
ORDER BY r.rank NULLS LAST;
```

**Adapt column name if needed:** the `team_merges` table may use `method` instead of `merge_type` for the ecnl_transition identifier. Check the actual schema:

```sql
-- Confirm the column name used for merge classification
SELECT column_name FROM information_schema.columns
WHERE table_name = 'team_merges'
  AND column_name IN ('merge_type', 'method', 'type');
```

Use whichever column exists. The filter value is `'ecnl_transition'` (from June 2 spec Check 2B, which uses `method = 'ecnl_transition'`).

**Run for all age groups if time permits:**

```sql
-- Check 6 extended — all age groups
SELECT
  t.name,
  r.rating::int AS current_rating,
  r.rank,
  r.age_group,
  r.gender,
  COUNT(tm.id) AS ecnl_transition_merges
FROM rankings r
JOIN teams t ON t.id = r.team_id
JOIN team_merges tm ON (tm.canonical_team_id = r.team_id OR tm.merged_team_id = r.team_id)
WHERE tm.method = 'ecnl_transition'
  AND r.rating BETWEEN 950 AND 1050
GROUP BY t.name, r.rating, r.rank, r.age_group, r.gender
ORDER BY r.age_group, r.rank NULLS LAST;
```

**Pass criterion:** Returns 0 rows. No ecnl_transition-involved team is near the initialization floor.

**Failure criterion:** Any rows returned. Pull the team name, find its pre-migration rating in the May 31 spot-check list. If the team was rated > 1200 pre-migration and now shows < 1050, the merge record for that specific team is not being loaded correctly (verify = false, or the merge was written pointing to the wrong canonical ID).

**Failure investigation query:**

```sql
-- For a specific team showing a rating reset, check its merge records
SELECT tm.*, t1.name AS canonical_name, t2.name AS merged_name
FROM team_merges tm
JOIN teams t1 ON tm.canonical_team_id = t1.id
JOIN teams t2 ON tm.merged_team_id = t2.id
WHERE tm.method = 'ecnl_transition'
  AND (t1.name ILIKE '%[TEAM NAME]%' OR t2.name ILIKE '%[TEAM NAME]%');
```

Check:
1. Is `verified = true` on this record? If no → dedup fix failed for this team specifically.
2. Is `canonical_team_id` pointing to the correct GotSport ID? If the mapping is inverted (GotSport → TGS instead of TGS → GotSport), the game history is orphaned the other direction.

---

## Section 4: Migration Script Requirement for FORGE

Based on the Section 1 determination (Option B — JOIN at query time):

> **REQUIREMENT for FORGE:** The ECNL migration script and/or `ecnl-transition-dedup.js` must ensure that every `ecnl_transition` record written to `team_merges` has `verified = true` at insert time. This is because the Elo computation resolves merges from an in-memory map built from verified records only — an unverified merge is invisible to `compute-rankings.js` and results in a rating reset for the affected team.
>
> Confirm with FORGE that `ecnl-transition-dedup.js` (Patch 1) explicitly inserts with `verified = true`. The pre-migration confirmation query:
>
> ```sql
> SELECT COUNT(*), verified
> FROM team_merges
> WHERE method = 'ecnl_transition'
> GROUP BY verified;
> ```
>
> Before June 1: this should return 0 rows (no ecnl_transition records exist yet).
> After June 1 migration: this should return exactly one row — `verified = true` with the full count of migrated teams. Any `verified = false` row indicates Patch 1 did not work correctly.
>
> **Do NOT run `compute-rankings.js` until this query confirms all ecnl_transition records are `verified = true`.**

This requirement corresponds to Section 3.2 of FORGE's pre-flight checklist. FORGE should confirm this verification step exists in their checklist before June 1.

---

## Section 5: Summary Assessment

**Rating continuity status: PRESERVED — conditional on dedup patch**

The Elo computation resolves merges at runtime via the in-memory verified merge map (Phase 1). If every `ecnl_transition` record written by the June 1 migration has `verified = true`, all ECNL teams will have their full pre-migration game history attributed correctly to their new canonical ID. Ratings will be continuous.

**The single point of failure is the `verified` flag.** Patch 1 (`ecnl-transition-dedup.js`) addresses this. Check 2B in the June 2 spec verifies it at the aggregate level. Check 6 (this document) verifies it at the team level.

**Residual risk after both checks pass:** Near zero. The only remaining scenario is a correctly-verified merge that maps to the wrong canonical ID (an inverted TGS/GotSport mapping). This would not appear in Check 6 (the team's rating would be reassigned, not reset to 1000) but would appear as an anomaly in Check 2E (top-team stability). The failure investigation query in Section 3 addresses this if found.

---

## References

- [[June 2 ECNL Migration Verification Spec]] — updated with Check 6 above
- [[Database Schema]] — `team_merges` schema, `rankings` table
- [[Ranking Engine]] — Phase 1 (verified merge loading), Phase 3 (Elo computation)
- [[June 1 Risk Assessment — Age Group Transition + ECNL Migration]] — Patch 1 (dedup fix) context
- `Reports/ecnl-dedup-verified-fix-2026-04-24.md` — FORGE's Patch 1 confirmation
