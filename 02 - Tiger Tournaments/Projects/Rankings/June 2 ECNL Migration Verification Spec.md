---
title: June 2 ECNL Migration Verification Spec
aliases: [ECNL Migration Day-After Checks, June 2 Verification]
tags: [rankings, migration, ecnl, age-groups, verification, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: ELO Agent
status: ready
task: task-2026-04-24-june2-ecnl-migration-verification-spec
run_on: 2026-06-02 (morning)
---

# June 2 ECNL Migration Verification Spec

> Two simultaneous changes on June 1: age group transition (all teams age up) and ECNL platform migration (TGS → GotSport). This spec is the 45-minute morning checklist for June 2. Section 1 covers what to save on May 31. Section 2 is the June 2 morning verification sequence.
>
> **Risk A** = age group transition. **Risk B** = ECNL platform migration (TGS → GotSport team ID change).

---

## Section 1: Pre-Migration Baseline (Run on May 31)

**Before any migration scripts execute on June 1**, run this snapshot query and save the output. This is the comparison baseline for June 2.

```sql
-- SAVE THIS OUTPUT — required for June 2 comparison
-- Run on May 31 BEFORE any migration scripts execute
-- Save to: Reports/pre-migration-snapshot-2026-05-31.txt

SELECT
  (SELECT COUNT(*) FROM games) AS total_games,
  (SELECT COUNT(*) FROM teams) AS total_teams,
  (SELECT COUNT(*) FROM rankings) AS total_rankings,
  (SELECT COUNT(*) FROM rankings WHERE age_group = 'U13' AND gender = 'F') AS u13g_ranked_teams,
  (SELECT COUNT(*) FROM rankings WHERE age_group = 'U14' AND gender = 'F') AS u14g_ranked_teams,
  (SELECT COUNT(*) FROM team_merges WHERE verified = true) AS verified_merges_before,
  (SELECT ROUND(AVG(rating), 1) FROM rankings WHERE age_group = 'U13' AND gender = 'F' AND rank IS NOT NULL) AS u13g_avg_rating,
  (SELECT ROUND(AVG(rating), 1) FROM rankings WHERE age_group = 'U14' AND gender = 'F' AND rank IS NOT NULL) AS u14g_avg_rating,
  (SELECT rating FROM rankings WHERE age_group = 'U13' AND gender = 'F' ORDER BY rank ASC LIMIT 1) AS u13g_rank1_rating,
  (SELECT rating FROM rankings WHERE age_group = 'U14' AND gender = 'F' ORDER BY rank ASC LIMIT 1) AS u14g_rank1_rating,
  NOW() AS snapshot_time;
```

**Save:** copy the output row to `02 - Tiger Tournaments/Projects/Reports/pre-migration-snapshot-2026-05-31.txt`

**Also save the pre-migration U13G top 10:**

```sql
-- Save to pre-migration-snapshot-2026-05-31.txt (append)
SELECT t.name, r.rating, r.rank
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND r.rank <= 10
ORDER BY r.rank;
```

**And generate the ECNL spot-check team list (save this too):**

```sql
-- ECNL teams expected to migrate — save for June 2 spot check
-- These are the teams whose ratings must survive the migration
SELECT DISTINCT t.id AS team_id, t.name AS team_name,
  MAX(r.rating) AS current_max_rating,
  r.age_group, r.gender
FROM games g
JOIN events e ON e.id = g.event_id
JOIN teams t ON t.id = g.home_team_id
JOIN rankings r ON r.team_id = t.id
WHERE e.event_tier IN ('ecnl', 'ecnl_regional')
  AND r.rating > 1200
GROUP BY t.id, t.name, r.age_group, r.gender
ORDER BY current_max_rating DESC
LIMIT 100;
```

This list is the June 2 spot-check roster. Any team on this list showing a rating < 1100 on June 2 warrants immediate investigation.

---

## Section 2: June 2 Morning Verification (Run in Sequence)

Allow ~45 minutes. Run checks in order — if check 2B fails, still run 2C–2E to understand the full scope of any issue.

---

### Check 2A: Game history retention (Risk A + Risk B)

The total game count must not decrease. Decreasing game count means games were orphaned or deleted during migration.

```sql
SELECT COUNT(*) AS total_games_june2,
       COUNT(DISTINCT home_team_id) + COUNT(DISTINCT away_team_id) AS unique_teams_in_games
FROM games;
```

**Compare against `total_games` in the May 31 snapshot.**

| Result | Action |
|---|---|
| `total_games_june2` ≥ May 31 baseline | Safe to proceed |
| `total_games_june2` < May 31 baseline by 1–100 | Investigate — could be deletion of bad records; check recent changes |
| `total_games_june2` < May 31 baseline by > 100 | **ROLLBACK TRIGGER** — games were orphaned during migration |

---

### Check 2B: ECNL migration merge coverage (Risk B)

This is the highest-risk check. Every ECNL team that migrated must have a verified `team_merges` record. Unverified records are invisible to the ranking engine and mean rating reset.

```sql
SELECT COUNT(*) AS total_merge_records,
       SUM(CASE WHEN verified = true THEN 1 ELSE 0 END) AS verified_count,
       SUM(CASE WHEN verified = false OR verified IS NULL THEN 1 ELSE 0 END) AS unverified_count,
       method
FROM team_merges
WHERE method ILIKE '%ecnl%'
   OR method ILIKE '%migration%'
   OR method = 'ecnl_transition'
GROUP BY method
ORDER BY total_merge_records DESC;
```

**Expected:**
- The `ecnl_transition` method row should have `unverified_count = 0`
- `verified_count` should be in the range 500–2,000 (based on ECNL team estimates from the Risk Assessment)
- No null `method` rows should appear in this result

| Result | Action |
|---|---|
| `ecnl_transition` method: all verified = true | Patch 1 worked — proceed |
| Any records with `verified = false` or NULL | **ROLLBACK TRIGGER** — `ecnl-transition-dedup.js` did not set verified=true; ratings will be reset on next recompute |
| `verified_count` = 0 for ecnl_transition | Dedup script did not run — run it before next ranking recompute |

If this check fails: **do NOT run `compute-rankings.js` until the team_merges records are corrected to `verified = true`**. Running a recompute with unverified merges will orphan the rating history permanently.

---

### Check 2C: Rating continuity for ECNL spot-check teams (Risk B)

Cross-reference against the ECNL team list saved in Section 1. A team from the pre-migration spot-check list that now shows a rating near 1000 has had its history orphaned.

```sql
-- Check spot-check teams for rating drops
-- Replace with actual team names/IDs from the May 31 spot-check list
SELECT t.name, r.rating, r.rank, r.age_group, r.gender
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.rating BETWEEN 950 AND 1050   -- Glicko default ≈ 1000; this range catches resets
  AND r.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND r.rank IS NOT NULL
ORDER BY r.rating ASC
LIMIT 30;
```

Then cross-reference: does any team in this output also appear in the May 31 ECNL spot-check list with `current_max_rating > 1200`? If yes — that team lost its history.

```sql
-- Direct spot-check: verify top ECNL teams by name still have their ratings
-- (Adapt with real team names from the spot-check list)
SELECT t.name, r.rating, r.rank, r.age_group, r.gender,
  CASE
    WHEN r.rating < 1100 THEN 'RATING_DROP — INVESTIGATE'
    WHEN r.rating < 1200 THEN 'Rating lower than expected — monitor'
    ELSE 'OK'
  END AS status
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group IN ('U13', 'U14', 'U15')
  AND (t.name ILIKE '%ECNL%' OR t.name ILIKE '%FC%')  -- adapt to actual naming patterns
  AND r.rank <= 50
ORDER BY r.rank ASC
LIMIT 25;
```

| Result | Action |
|---|---|
| No teams from spot-check list at rating < 1100 | Rating continuity confirmed |
| 1–5 teams from list at rating < 1100 | Investigate individually — may be legitimate drops (team declined) or orphaned history |
| > 5 teams from list at rating < 1100 | **ROLLBACK TRIGGER** — systemic merge failure; do not let next recompute run |

---

### Check 2D: Age group transition completeness (Risk A)

After transition, no active team should appear in both their old and new age group bracket. A team appearing in both suggests the transition script ran but did not clean up old records.

```sql
-- Teams appearing in more than one age group (suspect duplicates)
SELECT t.id, t.name, array_agg(DISTINCT r.age_group ORDER BY r.age_group) AS age_groups,
  COUNT(DISTINCT r.age_group) AS group_count
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group IN ('U12', 'U13', 'U14', 'U15', 'U16', 'U17')
  AND r.season = '2026-2027'
GROUP BY t.id, t.name
HAVING COUNT(DISTINCT r.age_group) > 1
LIMIT 30;
```

**Expected:** 0 rows, or a very small number of teams that legitimately compete in multiple age groups (e.g., a U13/U14 hybrid program).

| Result | Action |
|---|---|
| 0 rows | Age group transition completed cleanly |
| 1–10 rows | Review manually — likely legitimate cross-age-group programs |
| > 10 rows | **Investigate** — transition script may have created duplicates; do not roll back immediately but review before next recompute |

Also verify U19 teams were archived, not advanced:

```sql
-- U19 teams should NOT appear in 2026-27 season rankings
SELECT COUNT(*) AS u19_in_new_season
FROM rankings
WHERE age_group = 'U19' AND season = '2026-2027';
-- Expected: 0
```

---

### Check 2E: Top-team stability (Risk A + Risk B)

Run this and compare against the pre-migration U13G top 10 saved in Section 1.

```sql
SELECT t.name, r.rating, r.rank
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND r.rank <= 15
ORDER BY r.rank;
```

**Interpretation:**
- The age group transition will cause some reshuffling — teams advancing from U12G to U13G join the pool. Some reranking is expected.
- New U13G teams (advanced from U12G) may appear in the top 20; this is normal if they carried strong ratings forward.
- Any previously top-5 U13G team dropping to rank 50+ in one transition is suspicious and should be investigated.

| Result | Action |
|---|---|
| Top 10 is stable (same teams ± 3 positions) | Normal transition behavior |
| 1–3 new teams in top 10 that weren't in top 20 before | Check if they're U12G advance teams with strong carry-forward ratings — likely correct |
| > 5 new teams in top 10, previous top teams absent | **Investigate** — could be rating resets or incorrect age group advancement |

---

### Check 6: Rating Orphan Detection — ECNL Transition Teams (Risk B)

This check scans all teams involved in `ecnl_transition` merges for the rating reset signature. Check 2C covers the 100-team spot-check list. This check covers all remaining ECNL-migration teams.

```sql
-- Teams with ecnl_transition merges showing near-initialization ratings
-- A team previously rated >1200 that now shows ~1000 has had its history orphaned
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
WHERE tm.method = 'ecnl_transition'
  AND r.age_group IN ('U13', 'U14')
  AND r.gender = 'F'
  AND r.rating BETWEEN 950 AND 1050
GROUP BY t.name, r.rating, r.rank, r.age_group, r.gender
ORDER BY r.rank NULLS LAST;
```

Run for all age groups if time permits (replace the `age_group IN ('U13', 'U14') AND gender = 'F'` filter with no filter, or iterate).

**Pass criterion:** Returns 0 rows.
**Failure criterion:** Any rows returned. Pull the team name, check whether it appeared in the May 31 ECNL spot-check list with `current_max_rating > 1200`. If yes — that team's merge record did not load correctly into Phase 1. Check `verified` flag on its specific `team_merges` entry and confirm `canonical_team_id` mapping is not inverted.

See `Rankings/ECNL Rating Continuity Spec — June 2026.md` for the full investigation query.

| Result | Action |
|---|---|
| 0 rows | Rating orphan check passed — all ecnl_transition teams have non-reset ratings |
| 1–5 rows with ratings 950–1050 | Investigate individually — check their merge records; not necessarily systemic |
| > 5 rows or any pre-migration high-rated team | **ROLLBACK TRIGGER** — ecnl_transition merges are not loading correctly into Phase 1; do not run next recompute until fixed |

---

## Section 3: Rollback Decision Tree

### Immediate Rollback Triggers (act within 4 hours of discovery)

1. **Total game count decreased** from May 31 baseline by > 100 games (Check 2A)
2. **ECNL migration team_merges** show `verified = false` or NULL for the `ecnl_transition` method (Check 2B)
3. **Any team from the pre-migration ECNL spot-check list** (rating > 1200 on May 31) now shows rating < 1100 (Check 2C)

If any of these trigger:
- **Risk B rollback:** Delete the bad `team_merges` records, correct the dedup script to insert `verified = true`, re-run the dedup pass, then re-run `compute-rankings.js`
- **Risk A rollback:** Age group transition is date-based and not independently reversible. If both risks hit simultaneously, fix Risk B first (dedup correction), then re-run the age group advancement sequence

### Watchful Waiting (monitor, do not roll back immediately)

1. A small number of teams (1–10) appear in two age groups — review manually; likely legitimate
2. Top 10 U13G shuffle is significant but involves teams that were in the top 20 before transition — expected
3. U13G average rating has shifted by ±30–50 points — age group transition effect is expected (new U12G teams entering the U13G pool)
4. 1–5 ECNL spot-check teams show ratings between 1100 and 1200 — lower than expected but not a reset; monitor through one recompute cycle

### Safe to Proceed

1. All 5 checks complete without red flags
2. Game count equals or exceeds baseline
3. ECNL merge records all have `verified = true`
4. No team from the spot-check list shows an unexplained reset to ~1000
5. U19 count in 2026-27 season = 0

---

## Section 4: Rollback Window

| Time Since Migration | Rollback Status |
|---|---|
| 0–48 hours | Clean rollback: correct team_merges + re-run dedup + re-run compute |
| 48–96 hours | Rollback is still correct but may cause visible ranking instability (one cycle of shuffling) — communicate to Presten before executing |
| After 96 hours | Rolling back may do more harm than good — teams have played new games under new IDs; escalate to Presten judgment case-by-case |

---

## Acceptance Criteria Summary

| Check | Passes When | Fails When |
|---|---|---|
| 2A: Game count | `total_games_june2` ≥ May 31 baseline | Decreases by > 100 |
| 2B: ECNL merge coverage | All `ecnl_transition` merges `verified = true` | Any `verified = false` or NULL |
| 2C: Rating continuity | No spot-check team (was >1200) now at <1100 | Any spot-check team reset to <1100 |
| 2D: Age group completeness | 0 teams in both old and new age group (2026-27 season) | > 10 teams in duplicate brackets |
| 2D: U19 archived | 0 U19 rows in 2026-27 season | Any U19 rows in new season |
| 2E: Top-team stability | Top 10 stable ± 3 positions | > 5 previous top-10 teams absent from top 30 |
| 2F (Check 6): Rating orphan detection | 0 ecnl_transition teams near rating 1000 | Any previously high-rated team shows ~1000 post-migration |

---

## References

- [[June 1 Risk Assessment — Age Group Transition + ECNL Migration]] — complete risk inventory this spec addresses
- [[Calibration Production Runbook — May 2026]] — rollback patterns (reuse Section 3 for ECNL rollback)
- [[Database Schema]] — team_merges, rankings, games table structures
- `Reports/ecnl-dedup-verified-fix-2026-04-24.md` — FORGE's dedup fix confirmation (Patch 1 from Risk Assessment)
- `Reports/pre-migration-snapshot-2026-05-31.txt` — baseline snapshot (save on May 31; does not exist yet)
