---
title: backfill-age-gender.js Changes Spec — Dual-System Age Groups
tags: [migration, age-groups, spec, pipeline, dual-system, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: complete
related_task: task-2026-04-23-calibration-staging-validation
---

# `backfill-age-gender.js` Changes Spec — Dual-System Age Group Handling

> [!info] Context
> This document executes Step 4 of `task-2026-04-23-calibration-staging-validation`. It documents the specific code changes needed in `backfill-age-gender.js` to handle the dual-system age group environment created by the June 1, 2026 transition. See Section 4 of [[Age Group Transition Migration Spec 2026-06-01]] for the specification this implements.

---

## Summary

After June 1, 2026, most leagues use school-year age group cutoffs but MLS NEXT retains birth-year cutoffs. Cross-system games (e.g., a "U16 ECNL" team playing a "U15 MLS NEXT" team at a showcase) currently have no handling for the age group mismatch. The `backfill-age-gender.js` pipeline step needs three additions:

1. **School-year offset calculation** — convert school-year age groups to birth-year-equivalent for cross-system comparison
2. **Event-tier lookup** — check whether the game's event is a cross-system event (one where both school-year and birth-year teams compete)
3. **`cross_system_age_group` flag** — mark games where the two teams are from different cutoff systems

---

## Functions That Need Modification

The `backfill-age-gender.js` script likely contains these key functions (names may vary slightly in production):

| Function | Current Behavior | Change Required |
|----------|-----------------|-----------------|
| `parseAgeGroup(teamName, eventName)` | Parses age group from name string | Add school-year offset for post-June-1 events |
| `assignGameAgeGroup(homeTeam, awayTeam, event)` | Assigns a single age group to the game | Add cross-system detection logic |
| `backfillTeamAgeGender(gameRecord)` | Updates age/gender on a game record | Add `cross_system_age_group` flag assignment |

---

## Section 1: School-Year Offset Calculation

### Current Behavior

`parseAgeGroup()` reads an age label (e.g., "U16") directly from the team name or event bracket. It does not account for whether the league uses school-year or birth-year cutoffs.

### Change Required

After June 1, 2026, a team labeled "U16" in an ECNL event is classified as school-year U16 (born between Aug 1, 2009 and Jul 31, 2010). A team labeled "U15" in an MLS NEXT event is birth-year U15 (born in 2011). These are **different players** despite the fact that some birth years overlap.

```javascript
// BEFORE (current):
function parseAgeGroup(label, eventTier) {
  const match = label.match(/U(\d+)/i);
  return match ? `U${match[1]}` : null;
}

// AFTER (with school-year offset):
const SCHOOL_YEAR_LEAGUES = new Set([
  'ecnl', 'ecnl_rl', 'npl', 'dpl', 'ga', 'ga_aspire', 'state'
]);

const BIRTH_YEAR_LEAGUES = new Set([
  'mlsnext', 'mlsnext_homegrown', 'mlsnext_academy', 'mlsnext_unclassified'
]);

/**
 * Returns the age group label AND the cutoff system for the team.
 * @param {string} label - Team name or bracket label (e.g., "U16")
 * @param {string} eventTier - The event_tier value from event_tiers table
 * @param {Date} matchDate - Date of the match
 * @returns {{ ageGroup: string, cutoffSystem: 'school_year' | 'birth_year' | 'unknown' }}
 */
function parseAgeGroupWithSystem(label, eventTier, matchDate) {
  const match = label.match(/U(\d+)/i);
  if (!match) return { ageGroup: null, cutoffSystem: 'unknown' };
  
  const ageNum = parseInt(match[1]);
  const ageGroup = `U${ageNum}`;
  
  // Pre-transition: all teams use birth year
  const TRANSITION_DATE = new Date('2026-06-01');
  if (matchDate < TRANSITION_DATE) {
    return { ageGroup, cutoffSystem: 'birth_year' };
  }
  
  // Post-transition: cutoff system depends on event tier
  if (SCHOOL_YEAR_LEAGUES.has(eventTier)) {
    return { ageGroup, cutoffSystem: 'school_year' };
  }
  if (BIRTH_YEAR_LEAGUES.has(eventTier)) {
    return { ageGroup, cutoffSystem: 'birth_year' };
  }
  
  return { ageGroup, cutoffSystem: 'unknown' };
}
```

### Birth Year Cohort Derivation

Once we know the cutoff system, `birth_year_cohort` can be calculated:

```javascript
/**
 * Derives the birth year cohort for an age group in a given season.
 * @param {string} ageGroup - e.g., "U16"
 * @param {string} season - e.g., "2026-2027"
 * @param {'school_year' | 'birth_year'} cutoffSystem
 * @returns {number} birth year cohort (e.g., 2010)
 */
function deriveBirthYearCohort(ageGroup, season, cutoffSystem) {
  const ageNum = parseInt(ageGroup.replace('U', ''));
  const seasonStartYear = parseInt(season.split('-')[0]);
  
  if (cutoffSystem === 'school_year') {
    // School year U16 in 2026-27: born Aug 1, 2009 – Jul 31, 2010
    // Cohort = year the player turns the age number, counting from Aug 1
    // = (season start year) - age + 1
    // For U16 in 2026-27: 2026 - 16 + 0 = 2010 (turns 16 before Aug 1, 2026)
    return seasonStartYear - ageNum;
  } else {
    // Birth year U15 in 2026-27: born Jan 1, 2011 – Dec 31, 2011
    // = (season start year) - age + 1
    // For U15 in 2026-27: 2026 - 15 + 0 = 2011
    return seasonStartYear - ageNum;
  }
  // NOTE: Both formulas are the same numerically for a given season year.
  // The difference is WHICH season year to use: school-year uses the fall start year,
  // birth-year also uses the calendar year. They align for ages where the cutoffs overlap.
}
```

---

## Section 2: Cross-System Game Detection

### What is a Cross-System Game?

A cross-system game occurs when:
- Home team's event tier is a school-year league (ECNL, ECNL RL, etc.)
- Away team's event tier is a birth-year league (MLS NEXT)
- OR both teams are in a "showcase" event that explicitly invites teams from both systems

This happens at multi-league showcase events (e.g., ECNL Showcase at a destination tournament that also has MLS NEXT teams in adjacent brackets).

### Detection Logic

```javascript
/**
 * Assigns the canonical age group for a game, handling cross-system cases.
 * @param {object} homeTeam - { teamId, ageGroup, cutoffSystem }
 * @param {object} awayTeam - { teamId, ageGroup, cutoffSystem }
 * @param {object} event - { eventName, eventTier, matchDate }
 * @returns {object} { ageGroup, crossSystemAgeGroup, birthYearCohort }
 */
function assignGameAgeGroup(homeTeam, awayTeam, event) {
  const homeInfo = parseAgeGroupWithSystem(homeTeam.ageGroup, homeTeam.primaryEventTier, event.matchDate);
  const awayInfo = parseAgeGroupWithSystem(awayTeam.ageGroup, awayTeam.primaryEventTier, event.matchDate);
  
  const isCrossSystem = (
    homeInfo.cutoffSystem !== awayInfo.cutoffSystem &&
    homeInfo.cutoffSystem !== 'unknown' &&
    awayInfo.cutoffSystem !== 'unknown'
  );
  
  // Canonical age group = the EVENT's bracket (the event determines who plays whom)
  // The event_tier of the game itself takes precedence over the teams' home leagues
  const canonicalAgeGroup = parseAgeGroupFromEventBracket(event) || homeInfo.ageGroup;
  
  // Birth year cohort: derive from the event's age group and cutoff system
  const eventCutoffSystem = SCHOOL_YEAR_LEAGUES.has(event.eventTier) ? 'school_year' : 
                            BIRTH_YEAR_LEAGUES.has(event.eventTier) ? 'birth_year' : 'unknown';
  const season = deriveSeason(event.matchDate);
  const birthYearCohort = eventCutoffSystem !== 'unknown' 
    ? deriveBirthYearCohort(canonicalAgeGroup, season, eventCutoffSystem) 
    : null;
  
  return {
    ageGroup: canonicalAgeGroup,
    crossSystemAgeGroup: isCrossSystem,
    homeCutoffSystem: homeInfo.cutoffSystem,
    awayCutoffSystem: awayInfo.cutoffSystem,
    birthYearCohort
  };
}
```

---

## Section 3: `cross_system_age_group` Flag Assignment

### Schema Change Required

```sql
-- Add to games table (if not already present):
ALTER TABLE games 
  ADD COLUMN cross_system_age_group BOOLEAN DEFAULT false,
  ADD COLUMN birth_year_cohort INTEGER NULL,
  ADD COLUMN home_cutoff_system VARCHAR(15) NULL,  -- 'school_year', 'birth_year', 'unknown'
  ADD COLUMN away_cutoff_system VARCHAR(15) NULL;
```

### `backfillTeamAgeGender()` Changes

```javascript
// BEFORE:
async function backfillTeamAgeGender(gameRecord) {
  const ageGroup = parseAgeGroup(gameRecord.event_name || gameRecord.home_team_name);
  const gender = parseGender(gameRecord.home_team_name);
  
  await db.query(
    'UPDATE games SET age_group = $1, gender = $2 WHERE id = $3',
    [ageGroup, gender, gameRecord.id]
  );
}

// AFTER:
async function backfillTeamAgeGender(gameRecord, homeTeamRecord, awayTeamRecord) {
  const gender = parseGender(gameRecord.home_team_name);
  
  const assignment = assignGameAgeGroup(
    { ageGroup: homeTeamRecord.age_group, primaryEventTier: homeTeamRecord.primary_tier },
    { ageGroup: awayTeamRecord.age_group, primaryEventTier: awayTeamRecord.primary_tier },
    { eventName: gameRecord.event_name, eventTier: gameRecord.event_tier, matchDate: gameRecord.match_date }
  );
  
  await db.query(`
    UPDATE games SET 
      age_group = $1, 
      gender = $2,
      cross_system_age_group = $3,
      birth_year_cohort = $4,
      home_cutoff_system = $5,
      away_cutoff_system = $6
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

---

## Section 4: Before/After Behavior Example

### Cross-System Game Example

**Scenario:** An ECNL Showcase in September 2026. FC Dallas MLS NEXT U15 (birth-year) plays LAFC ECNL U16 (school-year). These teams are competing in the same bracket because the showcase invites teams from both leagues with overlapping birth cohorts.

**Players involved:**
- FC Dallas MLS NEXT U15: born 2011 (birth-year cutoff, U15 in MLS NEXT 2026-27)
- LAFC ECNL U16: born 2010 (school-year cutoff, turns 16 before Aug 1, 2026)

These players are in **different birth cohorts** despite competing in the same game.

| Field | Before (Current) | After (With Changes) |
|-------|-----------------|----------------------|
| `age_group` | "U15" (parsed from FC Dallas team name) | "U16" (from event bracket — the showcase uses U16 bracketing) |
| `cross_system_age_group` | NULL | `true` |
| `birth_year_cohort` | NULL | `2010` (event is school-year-based showcase: U16 = 2026 - 16 = 2010) |
| `home_cutoff_system` | NULL | `birth_year` (FC Dallas = MLS NEXT) |
| `away_cutoff_system` | NULL | `school_year` (LAFC = ECNL) |

**Ranking engine behavior:**
- The game is scored in the U16 age group bucket (event bracket wins)
- The `cross_system_age_group = true` flag allows Phase 8 to apply a cross-system correction factor if needed (Phase 2 feature — not implemented yet)
- For Phase 1: game is scored normally in U16. The flag is informational only.
- Long-term: cross-system games may need a slight calibration adjustment because the birth cohorts are different (one team is typically a year older in development terms)

---

## Section 5: Estimated Implementation Time

| Task | Estimated Hours |
|------|----------------|
| Add `cross_system_age_group`, `birth_year_cohort`, `home_cutoff_system`, `away_cutoff_system` columns to games table (migration) | 0.5 hrs |
| Implement `parseAgeGroupWithSystem()` function | 1.0 hrs |
| Implement `assignGameAgeGroup()` with cross-system detection | 1.5 hrs |
| Update `backfillTeamAgeGender()` to call new functions | 1.0 hrs |
| Update `deriveBirthYearCohort()` and season detection | 0.5 hrs |
| Write unit tests for cross-system detection (5 edge cases) | 1.5 hrs |
| Run backfill on test DB and validate | 1.0 hrs |
| **Total** | **7.0 hrs** |

**Recommended timing:** Implement by May 15 (before DSS). The migration is additive (new nullable columns) so it can run without disrupting existing functionality. The `cross_system_age_group` flag adds no risk to the current ranking computation — it's informational only in Phase 1.

---

## Edge Cases

| Case | Handling |
|------|---------|
| Pre-transition game (before June 1, 2026) | `cutoffSystem = 'birth_year'` for all teams; `cross_system_age_group = false` |
| Event tier is unknown/null | `cutoffSystem = 'unknown'`; `cross_system_age_group = false`; `birth_year_cohort = null` |
| Both teams are from birth-year leagues (all MLS NEXT) | `cross_system_age_group = false` |
| Both teams are from school-year leagues (all ECNL) | `cross_system_age_group = false` |
| Team's primary tier is Pre-ECNL (U10-U12, no transition impact) | Pre-ECNL age groups are unaffected by transition; `cross_system_age_group = false` |
| U19 teams (no transition impact at upper bound) | U19 is the same cutoff in both systems (graduating class logic applies) |

---

## Related

- [[Age Group Transition Migration Spec 2026-06-01]] — Section 4 (the specification this document implements)
- [[Ranking Engine]] — how age groups flow through the 9+1 phases
- [[Age Groups and Birth Years]] — mapping table and format rules
- [[Parsing Rules]] — current parsing logic context
- [[Database Schema]] — current schema to be extended with new columns
