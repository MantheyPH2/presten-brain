---
title: Age/Gender Dual-System — Implementation Note
tags: [pipeline, age-groups, dual-system, ecnl, mlsnext, migration, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: FORGE
status: spec complete — implementation ready for Presten
deadline: 2026-05-15
priority: high
---

# Age/Gender Dual-System — Implementation Note

**Analyst:** FORGE  
**Date:** 2026-04-23  
**Task:** `task-2026-04-23-implement-age-gender-dual-system.md`  
**Spec source:** `Reports/backfill-age-gender-changes-spec-2026-04-23.md` (written by ELO Agent)  
**Deadline:** May 15, 2026 — day before Dallas Spring Showcase rankings lock

---

## What This Solves

Starting June 1, 2026, ECNL and most non-MLS NEXT leagues switch from birth-year to school-year age group cutoffs. MLS NEXT retains birth-year. Without this implementation, games played at showcase events where ECNL teams face MLS NEXT teams will have ambiguous age group assignments — the current code cannot distinguish which cutoff system applies, corrupting the age group bucket used in ranking computations.

This is an additive schema change (4 new nullable columns). It does not break existing functionality; the new columns are informational-only in Phase 1 of the ranking engine.

---

## Schema Migration

Run against `youth_soccer` DB before running the backfill:

```sql
ALTER TABLE games
  ADD COLUMN IF NOT EXISTS cross_system_age_group BOOLEAN DEFAULT false,
  ADD COLUMN IF NOT EXISTS birth_year_cohort INTEGER NULL,
  ADD COLUMN IF NOT EXISTS home_cutoff_system VARCHAR(15) NULL,
  ADD COLUMN IF NOT EXISTS away_cutoff_system VARCHAR(15) NULL;

COMMENT ON COLUMN games.cross_system_age_group IS 'true if home and away teams use different cutoff systems (MLS NEXT birth-year vs ECNL school-year)';
COMMENT ON COLUMN games.birth_year_cohort IS 'birth year cohort for the canonical age group under the event cutoff system';
COMMENT ON COLUMN games.home_cutoff_system IS 'cutoff system of home team primary league: school_year, birth_year, or unknown';
COMMENT ON COLUMN games.away_cutoff_system IS 'cutoff system of away team primary league: school_year, birth_year, or unknown';
```

Verify migration ran clean:
```sql
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'games'
  AND column_name IN ('cross_system_age_group', 'birth_year_cohort', 'home_cutoff_system', 'away_cutoff_system');
```

---

## Code Implementation

### File: `backfill-age-gender.js`

All changes are to this file only. No other pipeline step needs modification for Phase 1.

#### 1. League Classification Constants

Add near the top of the file, after existing constants:

```javascript
// Post-June 1, 2026: leagues that use school-year cutoffs
const SCHOOL_YEAR_LEAGUES = new Set([
  'ecnl', 'ecnl_rl', 'npl', 'dpl', 'ga', 'ga_aspire', 'state'
]);

// Leagues that retain birth-year cutoffs after the transition
const BIRTH_YEAR_LEAGUES = new Set([
  'mlsnext', 'mlsnext_homegrown', 'mlsnext_academy', 'mlsnext_unclassified'
]);

const TRANSITION_DATE = new Date('2026-06-01');
```

#### 2. New Function: `parseAgeGroupWithSystem()`

Add below the existing `parseAgeGroup()` function. Do NOT replace `parseAgeGroup()` — it is still used for historical data.

```javascript
/**
 * Parses age group and determines the cutoff system (school-year or birth-year).
 * Pre-transition: all games use birth_year regardless of league.
 * Post-transition: cutoff system determined by event_tier.
 *
 * @param {string} label - Age group string (e.g., "U16", "U13G")
 * @param {string} eventTier - event_tier value from games or event_tiers table
 * @param {Date} matchDate - Date of the match
 * @returns {{ ageGroup: string|null, cutoffSystem: 'school_year'|'birth_year'|'unknown' }}
 */
function parseAgeGroupWithSystem(label, eventTier, matchDate) {
  const match = (label || '').match(/U(\d+)/i);
  if (!match) return { ageGroup: null, cutoffSystem: 'unknown' };

  const ageGroup = `U${match[1]}`;

  if (!matchDate || matchDate < TRANSITION_DATE) {
    return { ageGroup, cutoffSystem: 'birth_year' };
  }

  const tier = (eventTier || '').toLowerCase();
  if (SCHOOL_YEAR_LEAGUES.has(tier)) return { ageGroup, cutoffSystem: 'school_year' };
  if (BIRTH_YEAR_LEAGUES.has(tier)) return { ageGroup, cutoffSystem: 'birth_year' };

  return { ageGroup, cutoffSystem: 'unknown' };
}
```

#### 3. New Function: `deriveBirthYearCohort()`

```javascript
/**
 * Calculates the birth year cohort for an age group in a given season.
 * Formula is numerically identical for both cutoff systems; the difference
 * is which season year is used (both use the fall start year).
 *
 * @param {string} ageGroup - e.g., "U16"
 * @param {Date} matchDate - used to determine the season start year
 * @returns {number|null} birth year (e.g., 2010)
 */
function deriveBirthYearCohort(ageGroup, matchDate) {
  if (!ageGroup || !matchDate) return null;
  const ageNum = parseInt(ageGroup.replace('U', ''));
  if (isNaN(ageNum)) return null;

  // Season start year: if match is in Jan–Jul, season started the prior fall
  const month = matchDate.getMonth(); // 0-indexed
  const calYear = matchDate.getFullYear();
  const seasonStartYear = month < 7 ? calYear - 1 : calYear;

  return seasonStartYear - ageNum;
}
```

#### 4. New Function: `assignGameAgeGroup()`

```javascript
/**
 * Determines the canonical age group for a game and detects cross-system matchups.
 *
 * @param {object} homeTeam  - { ageGroup: string, primaryTier: string }
 * @param {object} awayTeam  - { ageGroup: string, primaryTier: string }
 * @param {object} event     - { eventName: string, eventTier: string, matchDate: Date }
 * @returns {object} { ageGroup, crossSystemAgeGroup, birthYearCohort, homeCutoffSystem, awayCutoffSystem }
 */
function assignGameAgeGroup(homeTeam, awayTeam, event) {
  const matchDate = event.matchDate instanceof Date ? event.matchDate : new Date(event.matchDate);

  const homeInfo = parseAgeGroupWithSystem(homeTeam.ageGroup, homeTeam.primaryTier, matchDate);
  const awayInfo = parseAgeGroupWithSystem(awayTeam.ageGroup, awayTeam.primaryTier, matchDate);

  const isCrossSystem = (
    homeInfo.cutoffSystem !== awayInfo.cutoffSystem &&
    homeInfo.cutoffSystem !== 'unknown' &&
    awayInfo.cutoffSystem !== 'unknown'
  );

  // Canonical age group: use the event bracket age group if available,
  // otherwise fall back to the existing parseAgeGroup() result for the game record.
  const eventAgeGroup = parseAgeGroup(event.eventName || '');
  const canonicalAgeGroup = eventAgeGroup || homeInfo.ageGroup;

  // Birth year cohort uses the event's cutoff system (event bracket defines who plays whom)
  const eventTier = (event.eventTier || '').toLowerCase();
  const eventCutoffSystem = SCHOOL_YEAR_LEAGUES.has(eventTier) ? 'school_year'
                          : BIRTH_YEAR_LEAGUES.has(eventTier) ? 'birth_year'
                          : 'unknown';

  const birthYearCohort = (canonicalAgeGroup && eventCutoffSystem !== 'unknown')
    ? deriveBirthYearCohort(canonicalAgeGroup, matchDate)
    : null;

  return {
    ageGroup: canonicalAgeGroup,
    crossSystemAgeGroup: isCrossSystem,
    birthYearCohort,
    homeCutoffSystem: homeInfo.cutoffSystem,
    awayCutoffSystem: awayInfo.cutoffSystem,
  };
}
```

#### 5. Update `backfillTeamAgeGender()`

The existing function signature is `backfillTeamAgeGender(gameRecord)`. Update it to also receive the home and away team records, or query them inline.

**Option A — inline DB query (simpler, acceptable for a one-time backfill):**

```javascript
async function backfillTeamAgeGender(gameRecord) {
  const gender = parseGender(gameRecord.home_team_name || '');

  // Fetch team tier context for cross-system detection
  const homeTeamRow = await db.query(
    'SELECT primary_tier FROM teams WHERE id = $1',
    [gameRecord.home_team_id]
  ).then(r => r.rows[0] || {});
  const awayTeamRow = await db.query(
    'SELECT primary_tier FROM teams WHERE id = $1',
    [gameRecord.away_team_id]
  ).then(r => r.rows[0] || {});

  const matchDate = new Date(gameRecord.match_date);
  const homeAgeGroup = parseAgeGroup(gameRecord.home_team_name || gameRecord.event_name || '');
  const awayAgeGroup = parseAgeGroup(gameRecord.away_team_name || gameRecord.event_name || '');

  const assignment = assignGameAgeGroup(
    { ageGroup: homeAgeGroup, primaryTier: homeTeamRow.primary_tier || null },
    { ageGroup: awayAgeGroup, primaryTier: awayTeamRow.primary_tier || null },
    { eventName: gameRecord.event_name, eventTier: gameRecord.event_tier, matchDate }
  );

  await db.query(`
    UPDATE games SET
      age_group             = $1,
      gender                = $2,
      cross_system_age_group = $3,
      birth_year_cohort     = $4,
      home_cutoff_system    = $5,
      away_cutoff_system    = $6
    WHERE id = $7
  `, [
    assignment.ageGroup,
    gender,
    assignment.crossSystemAgeGroup,
    assignment.birthYearCohort,
    assignment.homeCutoffSystem,
    assignment.awayCutoffSystem,
    gameRecord.id
  ]);
}
```

**Note:** If the `teams` table does not have a `primary_tier` column, derive it from the team's most frequent `event_tier` in the `games` table. The ELO spec assumes `primary_tier` is available — verify this in the actual schema before running.

---

## Unit Tests

Create or update `tests/backfill-age-gender.test.js`. Required test cases:

```javascript
// Test 1: Pre-transition game — all birth_year regardless of tier
test('pre-transition game returns birth_year for ECNL team', () => {
  const result = parseAgeGroupWithSystem('U16', 'ecnl', new Date('2025-10-15'));
  expect(result).toEqual({ ageGroup: 'U16', cutoffSystem: 'birth_year' });
});

// Test 2: Post-transition ECNL game → school_year
test('post-transition ECNL U16 returns school_year', () => {
  const result = parseAgeGroupWithSystem('U16', 'ecnl', new Date('2026-09-01'));
  expect(result).toEqual({ ageGroup: 'U16', cutoffSystem: 'school_year' });
});

// Test 3: Post-transition MLS NEXT game → birth_year
test('post-transition MLS NEXT U15 returns birth_year', () => {
  const result = parseAgeGroupWithSystem('U15', 'mlsnext', new Date('2026-09-01'));
  expect(result).toEqual({ ageGroup: 'U15', cutoffSystem: 'birth_year' });
});

// Test 4: Cross-system game detection (the spec example)
test('FC Dallas MLS NEXT U15 vs LAFC ECNL U16 is cross-system', () => {
  const assignment = assignGameAgeGroup(
    { ageGroup: 'U15', primaryTier: 'mlsnext' },
    { ageGroup: 'U16', primaryTier: 'ecnl' },
    { eventName: 'ECNL Showcase Fall 2026', eventTier: 'ecnl', matchDate: new Date('2026-09-15') }
  );
  expect(assignment.crossSystemAgeGroup).toBe(true);
  expect(assignment.homeCutoffSystem).toBe('birth_year');
  expect(assignment.awayCutoffSystem).toBe('school_year');
  expect(assignment.ageGroup).toBe('U16'); // event bracket wins
  expect(assignment.birthYearCohort).toBe(2010); // 2026 - 16 = 2010
});

// Test 5: Both ECNL — not cross-system
test('ECNL vs ECNL is not cross-system', () => {
  const assignment = assignGameAgeGroup(
    { ageGroup: 'U16', primaryTier: 'ecnl' },
    { ageGroup: 'U16', primaryTier: 'ecnl' },
    { eventName: 'ECNL Regular Season Fall 2026', eventTier: 'ecnl', matchDate: new Date('2026-10-01') }
  );
  expect(assignment.crossSystemAgeGroup).toBe(false);
});

// Test 6: Unknown event tier → unknown cutoff system, no cross-system flag
test('unknown event tier results in unknown cutoff system', () => {
  const result = parseAgeGroupWithSystem('U14', null, new Date('2026-09-01'));
  expect(result.cutoffSystem).toBe('unknown');
});
```

Run tests:
```bash
node --test tests/backfill-age-gender.test.js
# Or if using Jest:
npx jest tests/backfill-age-gender.test.js
```

---

## Backfill Execution

Once tests pass:

```bash
# Run on a subset first
BACKFILL_LIMIT=1000 node backfill-age-gender.js

# Verify cross_system_age_group count
# (expect a very small number — these are rare inter-system showcase games)
psql youth_soccer -c "
  SELECT 
    COUNT(*) FILTER (WHERE cross_system_age_group = true) as cross_system,
    COUNT(*) FILTER (WHERE cross_system_age_group = false) as same_system,
    COUNT(*) FILTER (WHERE birth_year_cohort IS NOT NULL) as has_cohort,
    COUNT(*) FILTER (WHERE home_cutoff_system IS NOT NULL) as has_home_system
  FROM games
  WHERE is_hidden = false
  LIMIT 1000;
"

# If counts look reasonable, run full backfill
node backfill-age-gender.js
```

### Expected Counts

- `cross_system_age_group = true`: Expect < 1% of games. Cross-system matchups are rare — they only occur at multi-league showcase events after June 1, 2026. Most historical games (all pre-2026) will have `cross_system_age_group = false`.
- `birth_year_cohort IS NOT NULL`: Should be populated for all games where both `age_group` and `match_date` are non-null. Target > 90% of non-hidden games.
- `home_cutoff_system IS NOT NULL`: Will be `unknown` for games where the team's `primary_tier` is not set. Track this count — high `unknown` rates indicate `primary_tier` is not well-populated in the `teams` table.

---

## Edge Cases and Handling

| Case | Handling |
|------|---------|
| `match_date` is NULL | `cutoffSystem = 'unknown'`, `cross_system_age_group = false`, `birth_year_cohort = null` |
| `event_tier` is NULL | `cutoffSystem = 'unknown'` — logs at debug level, no error |
| Pre-ECNL (U10–U12) | All use birth-year; transition does not affect these age groups |
| U19 | Same cohort formula applies; no special handling needed |
| Both teams have `primary_tier = null` | `cross_system_age_group = false`, both cutoff systems = `unknown` |
| Game date is exactly 2026-06-01 | `matchDate < TRANSITION_DATE` is false; June 1 itself is treated as post-transition |

---

## Implementation Sequence

1. Run schema migration SQL — verify 4 columns exist
2. Add constants (`SCHOOL_YEAR_LEAGUES`, `BIRTH_YEAR_LEAGUES`, `TRANSITION_DATE`)
3. Add `parseAgeGroupWithSystem()` 
4. Add `deriveBirthYearCohort()`
5. Add `assignGameAgeGroup()`
6. Update `backfillTeamAgeGender()` to use new functions
7. Run unit tests — all 6 must pass
8. Run backfill on test DB (1,000 row subset)
9. Validate cross-system count is reasonable
10. Run full backfill on production `youth_soccer` DB
11. Log final counts in this document

---

## Final Count Log (fill in after production run)

```
Migration run: [ date ]
Full backfill completed: [ date ]
Total games backfilled: ???
cross_system_age_group = true: ???  (expected: <1% of games)
birth_year_cohort populated: ???%
home_cutoff_system unknown: ???%  (flag if >20% — primary_tier gaps in teams table)
Backfill duration: ??? minutes
```

---

## References

- `Reports/backfill-age-gender-changes-spec-2026-04-23.md` — ELO's complete spec (source of truth for all function logic)
- [[Age Group Transition Migration Spec 2026-06-01]] — Section 4 (the transition specification)
- [[Database Schema]] — games table structure
- [[Ranking Engine]] — how age groups flow through ranking phases
- [[Parsing Rules]] — existing parsing context

---

*FORGE — 2026-04-23*
