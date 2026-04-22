---
title: Age Group Transition Migration Spec 2026-06-01
aliases: [Age Group Migration, Transition Spec, June 2026 Migration]
tags: [rankings, migration, age-groups, birth-year, school-year, spec, evo-draw]
created: 2026-04-22
updated: 2026-04-22
author: ELO Agent
status: draft — awaiting Presten review and approval
blocker_resolved: Calibration Validation 2026-04-22
transition_date: 2026-06-01
deadline: June 1, 2026
---

# Age Group Transition Migration Spec — June 1, 2026

> [!warning] SPECIFICATION ONLY — Do Not Implement Without Approval
> This document is a technical specification for Presten to review and approve before any code is written or any migration runs on production data. It is NOT an implementation guide for someone to act on immediately.

> [!success] Blocker Resolved
> This spec was blocked on `task-2026-04-22-brier-score-completion`. That analysis is now complete: see [[Calibration Validation 2026-04-22]]. Calibration corrections for U12, U13, U14, and U19 are specified and ready for approval. This spec is written against those corrected values.

---

## Background and Scope

On **June 1, 2026**, most US youth soccer leagues shift from birth-year to school-year (Aug 1 – Jul 31) age group cutoffs:

- **Shifting to school-year:** ECNL, ECNL RL, NPL, DPL, most state leagues
- **NOT shifting:** MLS NEXT (remains on birth-year cutoff)

This creates a dual-system environment for the 2026-27 season. A player born January 10, 2011 will be classified as U16 under school-year rules (because they turn 16 before Aug 1, 2026) but as U15 under MLS NEXT birth-year rules (because the MLS NEXT cutoff is December 31, 2025-born = U16, 2011-born = U15 in 2026-27). The [[Ranking Engine]], [[Database Schema]], and [[Parsing Rules]] all need to accommodate this split correctly.

**This spec covers:**

1. Rating carryover logic — which ratings carry forward and which reset
2. RD reset values by age group at season start
3. `birth_year_cohort` column addition to database tables
4. Dual-system age group handling for cross-league games
5. Rollback plan

**This spec does NOT cover:**

- The MLS NEXT tier split calibration (see [[MLS NEXT Tier Split Analysis 2026-04-22]])
- The ECNL platform migration from TGS to GotSport (separate data pipeline spec needed)
- UI/frontend changes

---

## Section 1: Rating Carryover Logic

### Core Principle

At the June 1 transition, each team's Glicko-2 rating in `rankings` table represents their performance during the 2025-26 season. The transition migration must:

1. Determine whether a team's rating carries forward or resets
2. If carrying forward, advance the team's age group (a 2025-26 U13 team becomes a 2026-27 U14 team)
3. Apply season-start RD expansion to all carried-forward ratings
4. Tag all records with `birth_year_cohort` and `season` (see Section 3)

### Carryover Decision Rules

#### Carry Forward with Adjustment (Glicko-2 teams)

A team's rating carries forward if ALL of the following are true:
- `match_count >= 10` (team has crossed the provisional Elo threshold into Glicko-2)
- Team has at least one game with `match_date >= 2026-04-01` (active in last 60 days of 2025-26 season cutoff: May 31 minus 60 days = April 1)
- Team is not a known merged/duplicate (`is_hidden = false` in games, and `team_id` is the canonical side of any team_merges records)

If all three conditions are met: carry forward the Glicko-2 `rating` value, apply season-start RD expansion (Section 2), and update `age_group` to the new season's bracket.

#### Reset to Starting Rating (Provisional teams)

A team's rating resets to the age group starting rating if ANY of the following are true:
- `match_count < 10` (still in provisional Elo phase — the rating is not a reliable Glicko-2 estimate)
- No games with `match_date >= 2026-04-01` (inactive for 60+ days before transition)

If a team resets: set `rating` to the age group starting rating for their NEW age group in 2026-27, set `rd` to the starting RD (high uncertainty), set `volatility` to default, set `match_count = 0`.

### Age Group Advancement

For every team that carries forward (and for reset teams entering their new age group):

```
new_age_group = age_group_advance(current_age_group, cutoff_system)
```

**For school-year leagues (ECNL, ECNL RL, NPL, DPL, most others):**

| 2025-26 Age Group | 2026-27 Age Group |
|-------------------|-------------------|
| U12 | U13 |
| U13 | U14 |
| U14 | U15 |
| U15 | U16 |
| U16 | U17 |
| U17 | U18 |
| U18 | U19 |
| U19 | Aged out (no carryover) |

**For MLS NEXT (birth-year, no change in cutoff system):**

Same advancement table as above — the teams still age up one bracket — but the `birth_year_cohort` value does not change. See Section 3 for how `birth_year_cohort` preserves the distinction.

### Edge Cases

**Edge Case 1: Team played both ECNL (school-year) and MLS NEXT (birth-year) at the same age group in 2025-26**

This happens at showcase events where teams from different league structures compete. Example: a U15 ECNL team plays U15 MLS NEXT teams at a summer showcase. Both teams have `age_group = U15` in their 2025-26 ratings, but they are from different birth cohorts under the two systems.

**Resolution:** The carryover uses the team's PRIMARY league affiliation to determine age group advancement. Primary league = the league that accounts for >50% of the team's rated games. If a team played 20 ECNL games and 4 MLS NEXT showcase games, they advance under ECNL rules. If the split is ambiguous (close to 50/50), default to the school-year advancement (ECNL rule). Flag these edge cases in the migration output for manual review — there should be fewer than 100 such teams based on current cross-league participation patterns.

**Edge Case 2: Team crossed the 10-match threshold in the final week of the 2025-26 season (May 24-31)**

These teams technically qualify for Glicko-2 carryover, but their Glicko-2 rating is based on very limited post-provisional data. The Glicko-2 RD will still be high (>100) since they just crossed the threshold.

**Resolution:** Apply the normal carryover rules. Because the RD is already high for these teams, the season-start RD expansion (Section 2) will have minimal additional effect. The existing Glicko-2 machinery handles this correctly — high RD means the first games of the new season move the rating quickly. No special case needed.

**Edge Case 3: U19 teams**

U19 players typically age out of youth soccer at the end of the 2025-26 season (they are 19 turning 20 in 2026). Their ratings should NOT carry forward to any new age group.

**Resolution:** For teams whose current age group is U19: write their final end-of-season ratings to a `season_archive` table (see Section 3 — this is separate from the operational `rankings` table), then delete or archive the operational ranking record. Do not advance them to U20 — that bracket is not in scope.

---

## Section 2: RD Reset Values by Age Group

At the start of a new season, Glicko-2 prescribes expanding RD to account for uncertainty introduced by offseason inactivity. The formula:

```
RD_new = min( sqrt(RD_end^2 + c^2), RD_max )
```

Where `c` is the uncertainty increase per rating period during inactivity. For our system, the offseason spans approximately 60-90 days (June 1 to August) — roughly 2-3 rating periods of inactivity.

For each age group, the RD reset works as follows:
- **`RD_end`:** The team's RD value at the end of 2025-26 (whatever it converged to)
- **`RD_floor_new`:** The updated floor from [[Calibration Validation 2026-04-22]] — the rating cannot drop below this in 2026-27
- **`RD_season_start`:** The target RD at season open — should be meaningfully above the floor to give the first games of the season proper influence

The RD expansion uses a minimum `RD_season_start` value that respects the new floor from the calibration fixes and adds a season-start buffer:

| Age Group | New RD Floor (post-fix) | Typical End-of-Season RD | Season-Start RD Reset | Notes |
|-----------|--------------------------|--------------------------|----------------------|-------|
| U9        | 60                       | 60-90                    | max(end_RD × 1.2, 90) | Young teams; high variance expected |
| U10       | 60                       | 60-85                    | max(end_RD × 1.2, 88) | |
| U11       | 55                       | 55-80                    | max(end_RD × 1.2, 85) | |
| U12       | **60** (raised)          | 60-80                    | max(end_RD × 1.15, 82) | RD floor raised from 50 |
| U13       | **75** (raised)          | 75-110                   | max(end_RD × 1.15, 95) | Significant floor raise; first 11v11 age group |
| U14       | **65** (raised)          | 65-95                    | max(end_RD × 1.15, 88) | |
| U15       | 45                       | 45-75                    | max(end_RD × 1.2, 78) | RD often near floor; expand aggressively |
| U16       | 40                       | 40-70                    | max(end_RD × 1.2, 75) | |
| U17       | 40                       | 40-65                    | max(end_RD × 1.2, 72) | |
| U19       | 35                       | 35-60                    | max(end_RD × 1.2, 68) | K-factor now 24; RD recovery more controlled |

**Implementation note:** The `max(end_RD × 1.2, floor)` formula means:
- If a team ended the season with a very low RD (overconfident), we expand by 20% to restore some uncertainty
- We never let the season-start RD drop below the stated minimum (which is above the floor but below the starting RD for a new team)
- Teams that reset to starting rating get a high starting RD (~120-150 depending on age group) — handled by the reset path, not this table

---

## Section 3: `birth_year_cohort` Column Addition

### Required Schema Changes

Two schema changes are required. Both can run as online schema migrations (adding a nullable column does not require a table rewrite in PostgreSQL).

#### Change 1: `rankings` table

```sql
-- Add birth_year_cohort and season columns to rankings table
ALTER TABLE rankings ADD COLUMN birth_year_cohort INTEGER NULL;
ALTER TABLE rankings ADD COLUMN season VARCHAR(10) NULL;

-- Index for common query patterns
CREATE INDEX idx_rankings_cohort ON rankings (birth_year_cohort);
CREATE INDEX idx_rankings_season ON rankings (season);

-- Example data after backfill:
-- team_id: TM001, age_group: 'U13', season: '2025-2026', birth_year_cohort: 2013
-- team_id: TM001, age_group: 'U14', season: '2026-2027', birth_year_cohort: 2013
-- The cohort (2013) is stable across seasons; age_group changes.
```

#### Change 2: `games` table

```sql
-- Add birth_year_cohort to games table
ALTER TABLE games ADD COLUMN birth_year_cohort INTEGER NULL;

-- Index for cross-cohort queries
CREATE INDEX idx_games_cohort ON games (birth_year_cohort);
```

### Mapping Rule: Deriving `birth_year_cohort`

Given `age_group` (e.g., "U13") and `season` (e.g., "2025-2026"), derive `birth_year_cohort`:

**For birth-year leagues (MLS NEXT — season year = age group year):**

```
base_year = first year of season (e.g., 2025 from "2025-2026")
birth_year_cohort = base_year - age_group_number + 1

Example: U13 in 2025-2026 → 2025 - 13 + 1 = 2013
Example: U16 in 2025-2026 → 2025 - 16 + 1 = 2010
```

**For school-year leagues (ECNL, ECNL RL, NPL, DPL — starting 2026-27):**

School-year cutoff is Aug 1 - Jul 31. A player is in age group UN if they turn N years old AFTER Aug 1 of the season start year. This shifts the birth year cohort by 1 relative to birth-year leagues:

```
-- For school-year leagues in 2026-27:
birth_year_cohort = base_year - age_group_number + 1 - school_year_offset

Where school_year_offset = 0 for birth-year leagues (MLS NEXT)
      school_year_offset = 1 for school-year leagues (ECNL, ECNL RL, etc.) starting 2026-27

Example: U16 ECNL in 2026-2027 → 2026 - 16 + 1 - 1 = 2010
Example: U16 MLS NEXT in 2026-2027 → 2026 - 16 + 1 - 0 = 2011
```

This correctly reflects that a school-year U16 (cutoff Aug 1) includes players who turn 16 before Aug 1, 2026 — meaning players born in 2010. A birth-year U16 in MLS NEXT includes players born in 2011 (turning 16 in 2027 under birth-year rules).

**Important:** The `birth_year_cohort` column should ONLY use the school-year offset formula for games/ratings tagged `season = '2026-2027'` or later, and only for non-MLS NEXT tiers. For 2025-26 and earlier (when all leagues were on birth-year), use the standard formula (offset = 0) for all leagues.

### Backfill Logic

```sql
-- Backfill rankings table (2025-26 and earlier — all birth-year)
UPDATE rankings
SET
  birth_year_cohort = (
    CAST(LEFT(season, 4) AS INTEGER) - CAST(REPLACE(age_group, 'U', '') AS INTEGER) + 1
  ),
  season = COALESCE(season, '2025-2026')
WHERE birth_year_cohort IS NULL
  AND age_group ~ '^U[0-9]+$'
  AND season IS NOT NULL;

-- Backfill games table (same logic for historical data)
UPDATE games
SET birth_year_cohort = (
    CAST(LEFT(season, 4) AS INTEGER) - CAST(REPLACE(age_group, 'U', '') AS INTEGER) + 1
  )
WHERE birth_year_cohort IS NULL
  AND age_group ~ '^U[0-9]+$'
  AND season IS NOT NULL;
```

For 2026-27 forward, the pipeline must apply the school-year offset during Step 2 (`backfill-age-gender.js`) based on `event_tier`:
- `event_tier IN ('mlsnext', 'mlsnext_homegrown', 'mlsnext_academy')` → offset 0
- All other event tiers → offset 1 (starting 2026-27 season only)

---

## Section 4: Dual-System Age Group Handling

### The Problem

For the 2026-27 season, showcase events that draw teams from both school-year and birth-year leagues will produce cross-system matchups. Example:

> ECNL U16 team (players born 2010, school-year U16) vs MLS NEXT U15 team (players born 2011, birth-year U15) at a July 2026 summer showcase.

These teams are competing in the same bracket (set by the event), but their `age_group` labels differ, and their `birth_year_cohort` differs by one year.

### Proposed Rule

**The game is scored in the age group of the EVENT (the tournament sets the bracket).** The `birth_year_cohort` column on the game record captures the actual birth cohort of each participating team independently.

For the example above:
- `age_group` on the game record = "U16" (the bracket age group set by the event)
- `birth_year_cohort` on the home team's game record = 2010 (ECNL U16)
- `birth_year_cohort` on the away team's game record = 2011 (MLS NEXT U15 entering U16 bracket)

This means a single game can have two different `birth_year_cohort` values — one for each team. The column should be understood as a **team-level** attribute on the game record, not a game-level attribute.

**Implementation:** The `games` table currently has `home_team_id` and `away_team_id` but a single `age_group` field. The `birth_year_cohort` column should be split into `home_birth_year_cohort` and `away_birth_year_cohort` to capture the dual-system case:

```sql
ALTER TABLE games ADD COLUMN home_birth_year_cohort INTEGER NULL;
ALTER TABLE games ADD COLUMN away_birth_year_cohort INTEGER NULL;

-- The existing birth_year_cohort column (added in Section 3) can remain
-- as a "game-level" cohort using the event's age group label for simple queries
-- that don't need to distinguish per-team cohort
```

For simple queries (e.g., "show me all U16 games this season"), `birth_year_cohort` from the event age group is sufficient. For cross-league comparisons and the ranking engine, `home_birth_year_cohort` and `away_birth_year_cohort` provide the precision needed.

### Changes Needed in `backfill-age-gender.js`

Current behavior: `backfill-age-gender.js` derives `age_group` from the `division` or `event_name` field using the parsing priority rules in [[Parsing Rules]].

Required additions for 2026-27:
1. After deriving `age_group`, look up the `event_tier` for the game's event
2. If `event_tier` is MLS NEXT and season >= 2026-27: set `home_birth_year_cohort` and `away_birth_year_cohort` using birth-year formula (offset 0)
3. If `event_tier` is non-MLS NEXT and season >= 2026-27: use school-year formula (offset 1) for the non-MLS NEXT team; use birth-year formula for any MLS NEXT team participating in a cross-league event
4. For showcase events tagged as multi-league: derive cohorts per-team based on the team's primary league affiliation (from `team_merges` or `teams` table)

**Parsing rule addition for dual-system age groups:**

When a game's `age_group` would be parsed as "U16" from the event bracket but one of the participating teams is known to be an MLS NEXT team competing in a school-year bracket:
- Do not change the `age_group` field (keep it at "U16" — the event bracket)
- Set the MLS NEXT team's `birth_year_cohort` to reflect their birth-year cohort (one year newer than the school-year cohort)
- Flag the game with `cross_system_age_group = true` for downstream filtering

This flag is important for the [[Cross-League Analysis]] methodology: cross-system games should be analyzed separately from same-system games to avoid confounding the win-rate data that drives calibration.

---

## Section 5: Rollback Plan

### If Calibration Corrections Are Still Under Revision on June 1

The minimum viable migration (MVP) that can run safely even without perfect calibration:

**MVP Migration:**
1. Schema changes only: Add `birth_year_cohort`, `season` columns — zero risk, additive
2. Backfill historical records with `birth_year_cohort` — reversible, no ranking impact
3. Apply age group advancement to team records (U13 → U14 etc.) — reversible via a reversal script
4. Apply season-start RD expansion — this is safe even with suboptimal calibration values because it widens uncertainty (more conservative), not narrows it

**Do NOT run in MVP mode:**
- Calibration parameter updates to `calibration.json` — defer if not approved
- Full rating recalculation with new parameters — defer until calibration is confirmed

The rating recalculation is the only step that can lock in calibration errors. The schema and structural changes can proceed independently.

### Rollback Mechanism

If the migration produces obviously wrong results (e.g., top-ranked teams appearing at rank 500 or all teams clustering at the starting rating):

```bash
# Step 1: Restore rankings table from pre-migration backup
pg_restore -d youth_soccer --table=rankings backup_pre_migration_2026-05-31.dump

# Step 2: Restore calibration.json from version control
git checkout v2025-26-final -- config/calibration.json

# Step 3: Re-run compute-rankings.js with restored parameters
node compute-rankings.js

# Step 4: Verify ranking distribution against pre-migration snapshot
node scripts/validate-rankings-distribution.js --compare=2026-05-31
```

**Pre-migration backup is mandatory.** The migration should not run unless a verified backup of the `rankings` table (and `calibration.json`) exists and has been tested for restore.

The expected rollback time: approximately 2-3 hours (restore + recalculate + verify). This is acceptable given the June 1 transition is not a hard public-facing deadline — it affects internal data quality, not a live customer event.

### Post-Migration Monitoring

After the migration runs:

| Check | Method | Timing | Pass Criteria |
|-------|--------|--------|---------------|
| Rating distribution shape | Compare histogram of ratings by age group before/after | Immediately | Distribution shape preserved; mean within 15 points of pre-migration |
| Top team rank stability | Spot-check top 50 teams by age group | Immediately | >80% of top 20 teams remain in top 30 |
| Brier score on first week of games | Run Brier calculation on July 2026 games (first post-transition games) | July 5-10, 2026 | All age groups ≤ 0.23 |
| Cross-league game tagging | Verify `cross_system_age_group` flag appearing on expected games | First week of 2026-27 season | Flag appears on >90% of expected cross-system matchups |
| birth_year_cohort backfill coverage | Count null values in `birth_year_cohort` on games table | After backfill | <1% null for games with known age_group and season |

---

## Pre-Migration Checklist

Before the migration runs, the following must be confirmed:

- [ ] **Calibration fixes applied and validated:** U12 (RD floor 60), U13 (K=24, RD floor 75), U14 (K=28, RD floor 65), U19 (K=24) are in `calibration.json` and have been tested against held-out match data. See [[Calibration Validation 2026-04-22]].
- [ ] **MLS NEXT tier split decision made:** Either `mlsnext_academy` calibration (135) is implemented, or explicit decision to defer to 2026-27. See [[MLS NEXT Tier Split Analysis 2026-04-22]].
- [ ] **`birth_year_cohort` schema migration run on production and backfill complete.** Verify with: `SELECT COUNT(*) FROM games WHERE birth_year_cohort IS NULL AND age_group IS NOT NULL AND season IS NOT NULL;` — should return <1% of total.
- [ ] **Pre-migration backup verified:** `pg_restore` test run completed successfully on staging environment.
- [ ] **Rollback script tested:** Rollback scenario simulated on staging; expected restore time confirmed.
- [ ] **`resolveTeamTier()` update plan confirmed:** Either event_tiers table updated for MLS NEXT split, or explicit decision to defer.
- [ ] **DSS 2026 concluded and division placements frozen:** Do not run migration while DSS brackets are live.

---

## Estimated Pipeline Run Time for Migration

The migration requires a full recalculation of all ratings (570K+ games). Based on current `compute-rankings.js` performance:

| Step | Estimated Time |
|------|----------------|
| Phase 1: Load team merges (18,684 records) | ~2 minutes |
| Phase 2: Load event tiers (10,342 events) | ~1 minute |
| Phase 3: Elo computation (570K+ games) | ~18-25 minutes |
| Phases 4-7: H2H, transitive H2H, common opponent, SOS | ~12-18 minutes |
| Phase 8: League calibration | ~3 minutes |
| Phase 9: Division calibration (308 DSS teams) | ~1 minute |
| Phase 10: H2H enforcement (iterative) | ~8-15 minutes |
| `birth_year_cohort` backfill (games table) | ~15-20 minutes |
| Rankings table write + indexing | ~5 minutes |
| **Total estimated run time** | **~65-90 minutes** |

Allow a 2-hour window for the migration to complete, plus a 30-minute post-run validation window.

---

## Related Notes

- [[Calibration Validation 2026-04-22]] — Brier score analysis; blocker document (resolved)
- [[MLS NEXT Tier Split Analysis 2026-04-22]] — parallel calibration work
- [[Ranking Engine]] — 9+1 phases; migration touches Phases 1, 2, 3, 8, 9, 10
- [[Age Groups and Birth Years]] — mapping table and formula reference
- [[Recent Changes 2024-2026]] — age group shift documentation; MLS NEXT exception
- [[Parsing Rules]] — `backfill-age-gender.js` changes required in Section 4
- [[Database Schema]] — tables being modified: `rankings`, `games`
- [[Data Pipeline]] — pipeline steps affected
- [[Strategic Insights]] — Insight 3 (age group shift risk); Action Item 4 (birth_year_cohort)
