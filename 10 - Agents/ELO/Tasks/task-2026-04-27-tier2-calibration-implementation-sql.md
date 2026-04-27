---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Tier 2 Undefined Leagues — Calibration Implementation SQL.md"
---

# Task: Tier 2 Undefined Leagues — Calibration Implementation SQL Package

## Context

ELO's Tier 2 Undefined Leagues Calibration Recommendation (filed today) contains a Section 4 "Implementation Spec" that describes the steps in prose. It names the four leagues (USL Academy, EDP, Elite 64, NAL) and the proposed calibration values (55, 55, 55, 40 respectively).

What Section 4 does not contain: the actual SQL UPDATE statements. Presten currently has a recommendation document and a description of "update League Hierarchy with 4 new cal values." To execute, Presten needs to either (a) figure out the SQL from the description, or (b) wait for ELO to write it.

This task eliminates that gap. ELO writes the complete, ready-to-run SQL package so that when Presten authorizes the values on April 29 (or shortly after the volume check runs), execution is a copy-paste operation.

The volume check SQL (filed in `Rankings/League Hierarchy Calibration Sanity Check — April 2026.md`) runs on April 29. The authorization decision follows from the volume-gated decision matrix. This SQL package must be ready before April 29 so there is no blocking gap between authorization and execution.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Tier 2 Undefined Leagues — Calibration Implementation SQL.md`

---

### Section 1: Pre-Execution Checklist

Before running any SQL, confirm:

- [ ] Presten has filed written authorization (one-line confirmation in session, briefing, or Queue item)
- [ ] Volume check SQL has run and result meets the ≥ 500 games threshold for at least some leagues
- [ ] Current League Hierarchy snapshot is saved (SELECT of affected rows before UPDATE)
- [ ] Recompute scope is confirmed (full vs. partial — see Section 5)

---

### Section 2: Current State Snapshot SQL

Run this BEFORE any UPDATE to capture the baseline:

```sql
-- Baseline snapshot: current state of affected leagues
SELECT 
    league_name,
    event_tier,
    calibration_value,
    COUNT(*) as game_count
FROM games
WHERE event_tier IN ('usl_academy', 'edp', 'elite_64', 'nal')
GROUP BY league_name, event_tier, calibration_value
ORDER BY game_count DESC;
```

Note: ELO should confirm the exact `event_tier` string values for these leagues before filing. The strings above are best-guess from league naming conventions. If the actual `event_tier` strings differ, replace them here.

Save the output of this query. It is the rollback reference.

---

### Section 3: League Hierarchy UPDATE SQL

```sql
-- USL Academy: cal = 55
UPDATE league_hierarchy
SET calibration_value = 55,
    updated_at = NOW(),
    updated_by = 'elo-agent-2026-04-29'
WHERE event_tier = 'usl_academy';

-- EDP Soccer: cal = 55
UPDATE league_hierarchy
SET calibration_value = 55,
    updated_at = NOW(),
    updated_by = 'elo-agent-2026-04-29'
WHERE event_tier = 'edp';

-- Elite 64: cal = 55
UPDATE league_hierarchy
SET calibration_value = 55,
    updated_at = NOW(),
    updated_by = 'elo-agent-2026-04-29'
WHERE event_tier = 'elite_64';

-- NAL: cal = 40
-- Note: Apply only if April 29 volume check shows ≥ 500 NAL games.
-- If < 500 games: defer NAL to post-DSS per the volume-gated decision matrix.
UPDATE league_hierarchy
SET calibration_value = 40,
    updated_at = NOW(),
    updated_by = 'elo-agent-2026-04-29'
WHERE event_tier = 'nal';
```

**ELO note:** The `updated_by` convention (if any) must match the existing schema. If `league_hierarchy` has no `updated_by` column, remove those lines. Adjust table name and column names to match the actual schema.

---

### Section 4: Verification SQL (Post-UPDATE)

Run immediately after the UPDATE to confirm changes applied:

```sql
-- Verify all 4 leagues updated correctly
SELECT 
    event_tier,
    calibration_value,
    updated_at
FROM league_hierarchy
WHERE event_tier IN ('usl_academy', 'edp', 'elite_64', 'nal')
ORDER BY event_tier;
```

Expected result:
| event_tier | calibration_value | updated_at |
|------------|------------------|------------|
| edp | 55 | [today] |
| elite_64 | 55 | [today] |
| nal | 40 | [today] (if applied) |
| usl_academy | 55 | [today] |

If any row shows the old value (NULL or 0): the UPDATE did not apply. Do not proceed to recompute. Diagnose before continuing.

---

### Section 5: Recompute Scope

After League Hierarchy is updated, the rating system must recompute to reflect the new calibration values.

ELO must determine: is a full recompute required, or only a partial recompute for affected teams?

**Full recompute required if:** calibration values affect cross-league rating comparisons globally (i.e., the change propagates through the entire rating graph).

**Partial recompute sufficient if:** only teams that have played in these 4 leagues need new ratings computed.

ELO states which applies here, and provides the recompute command:

```
# Full recompute:
[command]

# Partial recompute (affected leagues only):
[command with league filter]
```

If ELO cannot determine this from vault context, state "full recompute — confirm with Presten before running."

---

### Section 6: Post-Recompute Verification

After recompute completes, confirm the calibration change had the expected effect:

```sql
-- Sample: top 10 teams in newly-calibrated leagues
-- Check that their ratings shifted in the expected direction
SELECT 
    team_name,
    league,
    old_rating,  -- from pre-execution snapshot
    new_rating,
    (new_rating - old_rating) as delta
FROM [rating comparison view or join against snapshot]
WHERE league IN ('USL Academy', 'EDP', 'Elite 64', 'NAL')
ORDER BY ABS(new_rating - old_rating) DESC
LIMIT 10;
```

ELO defines expected direction: teams in USL Academy, EDP, Elite 64 (cal=55) should see ratings shift toward the range of ECNL RL / Regional Premier peers. NAL teams (cal=40) should shift toward lower Tier 2 positioning. If ratings move in the wrong direction, do not file "complete" — escalate to SENTINEL.

---

### Section 7: Rollback SQL

If the recompute produces incorrect results or the verification step fails:

```sql
-- Rollback: restore pre-execution calibration values
-- (Use the baseline snapshot from Section 2 to fill in original values)
UPDATE league_hierarchy
SET calibration_value = [original_value],
    updated_at = NOW(),
    updated_by = 'elo-agent-rollback'
WHERE event_tier = 'usl_academy';

-- Repeat for edp, elite_64, nal
```

After rollback: re-run recompute. The system returns to the pre-implementation state.

---

## Quality Standard

- All SQL must match the actual table and column names. ELO should confirm these against the vault's schema documentation before filing. "I don't know the exact table name" is acceptable and should be noted — do not guess schema details.
- Section 5 recompute scope must be specific: "full" or "partial with [command]." Not "depends on the system."
- The NAL conditional (Section 3) must be preserved — do not remove the volume gate. If SENTINEL later confirms NAL volume is < 500, this task's SQL package handles that case correctly.

## References

- `Rankings/Tier 2 Undefined Leagues — Calibration Recommendation.md` — values and reasoning (already filed)
- `Rankings/League Hierarchy Calibration Sanity Check — April 2026.md` — volume check SQL
- `Rankings/League Hierarchy.md` — existing calibration values (anchor for sanity check)
- `Rankings/DSS Ranking Accuracy Claims Spec.md` — DSS coverage context
