---
title: Tier 2 Undefined Leagues — Calibration Implementation SQL
type: implementation-sql
author: ELO
date: 2026-04-27
status: ready-for-execution — pending Presten authorization
related: "[[Tier 2 Undefined Leagues — Calibration Recommendation]]", "[[League Hierarchy Calibration Sanity Check — April 2026]]", "[[League Hierarchy]]"
tags: [calibration, sql, tier2, implementation, evo-draw]
---

# Tier 2 Undefined Leagues — Calibration Implementation SQL

> **Purpose:** Ready-to-execute SQL package for applying the four Tier 2 calibration values recommended in `Rankings/Tier 2 Undefined Leagues — Calibration Recommendation.md`. Do not run until Presten authorizes and the pre-execution checklist is complete. Run the volume-check on April 29 first — the decision matrix in the Recommendation document determines which leagues apply pre-DSS.

---

## Section 1: Pre-Execution Checklist

Before running any SQL, confirm:

- [ ] Presten has filed written authorization (one-line confirmation in session, briefing, or Queue item)
- [ ] Volume-check SQL (from `Rankings/League Hierarchy Calibration Sanity Check — April 2026.md`) has run on April 29 and results are available
- [ ] Game counts meet the ≥ 500 games threshold for at least one league (per the volume-gated decision matrix)
- [ ] Current League Hierarchy snapshot saved (Section 2 baseline query has run)
- [ ] Recompute scope confirmed (full vs. partial — see Section 5)

**NAL-specific gate:** Apply NAL cal=40 only if the April 29 volume check shows ≥ 500 NAL games. If < 500 NAL games: apply USL Academy, EDP, and Elite 64, then defer NAL to post-DSS.

---

## Section 2: Current State Snapshot SQL

Run this **before any UPDATE** to capture the baseline. Save the output as the rollback reference.

```sql
-- Baseline snapshot: confirm event_tier strings and game counts for affected leagues
-- Step 1: Find the actual event_tier strings in use (do not assume names match League Hierarchy)
SELECT 
    event_tier,
    COUNT(*) AS game_count
FROM events
WHERE event_tier ILIKE ANY(ARRAY[
    '%usl%acad%', '%usl_acad%',
    '%edp%',
    '%elite%64%', '%elite64%',
    '%nal%', '%nat%acad%league%'
])
GROUP BY event_tier
ORDER BY game_count DESC;
```

Review the output carefully. Confirm the exact `event_tier` strings before running the UPDATE statements in Section 3. The April 26 sanity check flagged a potential typo ('uslacdemy') in an earlier audit — verify.

```sql
-- Step 2: Capture current calibration values for all four leagues
-- (Adjust table/column names to match actual schema if different)
SELECT 
    event_tier,
    calibration_value,
    updated_at
FROM league_hierarchy
WHERE event_tier IN ('<confirmed_usl_academy_string>', '<confirmed_edp_string>', 
                     '<confirmed_elite64_string>', '<confirmed_nal_string>');
```

---

## Section 3: League Hierarchy UPDATE SQL

After confirming the exact `event_tier` strings from Section 2:

```sql
-- USL Academy: cal = 55
-- Competitive anchor: ECNL RL (55). Professional club development pathway — Tier 2 floor.
UPDATE league_hierarchy
SET calibration_value = 55
WHERE event_tier = '<confirmed_usl_academy_string>';

-- Verify before continuing:
-- SELECT event_tier, calibration_value FROM league_hierarchy WHERE event_tier = '<confirmed_usl_academy_string>';
-- Expected: calibration_value = 55
```

```sql
-- EDP Soccer: cal = 55
-- Competitive anchor: NPL (55). Documented Northeast regional peer of NPL.
UPDATE league_hierarchy
SET calibration_value = 55
WHERE event_tier = '<confirmed_edp_string>';
```

```sql
-- Elite 64: cal = 55
-- Competitive anchor: ECNL RL (55). Showcase format drawing ECNL RL-caliber teams by selection.
UPDATE league_hierarchy
SET calibration_value = 55
WHERE event_tier = '<confirmed_elite64_string>';
```

```sql
-- NAL (National Academy League): cal = 40
-- NOTE: Apply ONLY if April 29 volume check shows ≥ 500 NAL games.
-- If < 500 games: skip this UPDATE and defer NAL to post-DSS.
-- Competitive anchor: Positioned between Pre-ECNL (25) and NPL (55). Variable competitive depth.
UPDATE league_hierarchy
SET calibration_value = 40
WHERE event_tier = '<confirmed_nal_string>';
```

**Schema note:** The UPDATE statements above do not include `updated_at` or `updated_by` columns. If the `league_hierarchy` table has these columns, add them:
```sql
-- Example with audit fields (use only if columns exist in schema):
UPDATE league_hierarchy
SET calibration_value = 55,
    updated_at = NOW()
WHERE event_tier = '<confirmed_string>';
```

---

## Section 4: Verification SQL (Post-UPDATE)

Run immediately after all UPDATEs to confirm changes applied correctly:

```sql
-- Verify all applied leagues updated correctly
SELECT 
    event_tier,
    calibration_value,
    updated_at
FROM league_hierarchy
WHERE event_tier IN ('<confirmed_usl_academy_string>', '<confirmed_edp_string>', 
                     '<confirmed_elite64_string>', '<confirmed_nal_string>')
ORDER BY event_tier;
```

**Expected results:**

| event_tier | calibration_value | notes |
|------------|------------------|-------|
| \<edp string\> | 55 | |
| \<elite64 string\> | 55 | |
| \<nal string\> | 40 | only if volume ≥ 500 games |
| \<usl_academy string\> | 55 | |

If any row shows NULL or the old value (0 or the pre-existing value): the UPDATE did not apply. Do **not** proceed to recompute. Diagnose before continuing — likely cause is a string mismatch in the WHERE clause.

---

## Section 5: Recompute Scope

After League Hierarchy is updated, the rating system must recompute to reflect the new calibration values.

**ELO position:** A **partial recompute** is sufficient for these four leagues. The new calibration values affect only teams that have games tagged to these four `event_tier` values — they do not change the global K-factor, RD floor, or any cross-league comparison constant. A partial recompute covering only teams with games in the four affected leagues produces the correct result.

**Partial recompute command:**
```
-- Confirm the partial recompute command with Presten. Expected form:
node compute-rankings.js --event-tiers usl_academy,edp,elite_64,nal

-- If partial recompute is not supported by compute-rankings.js,
-- use: full recompute — confirm with Presten before running.
-- Full recompute command: node compute-rankings.js --full
```

**Full recompute required if:** Any of the four leagues has > 10,000 combined games in the DB AND SENTINEL authorizes a full run. At that volume, the downstream rating effects may be non-trivial and scope confirmation is warranted.

---

## Section 6: Post-Recompute Verification

After recompute completes, confirm calibration changes had the expected effect. Affected teams were previously treated as **untiered** (equivalent to below-Tier-3 / NULL calibration, which suppressed their ratings). After applying cal=55 (or cal=40 for NAL), their ratings should **increase** — they were underweighted.

```sql
-- Sample: top 10 teams with most games in newly-calibrated leagues
-- Confirm their ratings shifted upward post-recompute
SELECT 
    t.team_name,
    t.age_group,
    t.gender,
    t.league,
    t.rating AS current_rating
FROM teams t
WHERE t.league IN ('<confirmed_usl_academy_string>', '<confirmed_edp_string>', 
                   '<confirmed_elite64_string>', '<confirmed_nal_string>')
ORDER BY t.rating DESC
LIMIT 10;
```

**Expected direction:** Teams that played primarily in these leagues should now rate higher than pre-recompute. If top teams in USL Academy, EDP, or Elite 64 are still rating below 1,100 (which would indicate Tier 3 or lower calibration), the recompute may not have applied the new values. Re-run verification.

**If ratings moved in the wrong direction (decreased):** Stop. Do not file complete. Escalate to SENTINEL — may indicate a logic error in how the recompute applied the new calibration values.

---

## Section 7: Rollback SQL

If the recompute produces incorrect results or post-recompute verification fails, restore the pre-execution state:

```sql
-- Rollback USL Academy: restore pre-execution value
-- (Fill original_value from Section 2 baseline snapshot)
UPDATE league_hierarchy
SET calibration_value = <original_value_from_snapshot>
WHERE event_tier = '<confirmed_usl_academy_string>';

-- Rollback EDP Soccer
UPDATE league_hierarchy
SET calibration_value = <original_value_from_snapshot>
WHERE event_tier = '<confirmed_edp_string>';

-- Rollback Elite 64
UPDATE league_hierarchy
SET calibration_value = <original_value_from_snapshot>
WHERE event_tier = '<confirmed_elite64_string>';

-- Rollback NAL (only if it was applied)
UPDATE league_hierarchy
SET calibration_value = <original_value_from_snapshot>
WHERE event_tier = '<confirmed_nal_string>';
```

After rollback: re-run recompute (same scope as Section 5). The system returns to the pre-implementation state. File rollback in next ELO briefing.

---

## Filing Note

After successful execution, ELO files in the next briefing:
```
Tier 2 calibration values applied: USL Academy (55), EDP (55), Elite 64 (55)[, NAL (40) if volume ≥ 500].
Recompute complete. Top affected teams shifted upward as expected.
League Hierarchy now covers [N] calibrated tiers.
```

---

## References

- `Rankings/Tier 2 Undefined Leagues — Calibration Recommendation.md` — calibration values and reasoning
- `Rankings/League Hierarchy Calibration Sanity Check — April 2026.md` — volume-check SQL (run first on April 29)
- `Rankings/League Hierarchy.md` — existing calibration values used as anchors
- `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md` — DSS accuracy claim context

*ELO — 2026-04-27*
