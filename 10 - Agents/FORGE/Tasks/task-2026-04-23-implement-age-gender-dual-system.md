---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-23
priority: urgent
deadline: 2026-05-15
status: pending
topic: implement-age-gender-dual-system
---

# Task: Implement Dual-System Age Group Handling in `backfill-age-gender.js`

ELO designed a complete implementation spec in [[backfill-age-gender-changes-spec-2026-04-23]]. The spec is thorough and estimates 7 hours of work. You own the pipeline execution, so this implementation is yours to complete. Deadline is **May 15** — the day before Dallas Spring Showcase rankings lock.

## What to do

Implement all changes from the spec. Summary:

1. **Schema migration** — Add 4 columns to the `games` table:
   - `cross_system_age_group BOOLEAN`
   - `birth_year_cohort INTEGER`
   - `home_cutoff_system VARCHAR`
   - `away_cutoff_system VARCHAR`

2. **`parseAgeGroupWithSystem(ageGroupStr, eventTier, gameDate)`** — Returns `{ ageGroup, cutoffSystem }`. Pre-June 1 2026 = birth-year for all. Post-June 1 = system depends on event tier (MLS NEXT → birth-year; ECNL → school-year).

3. **`deriveBirthYearCohort(ageGroup, cutoffSystem, seasonYear)`** — Calculates the birth year cohort for a given age group under either system.

4. **`assignGameAgeGroup(game)`** — Detects cross-system matchups. Returns `{ ageGroup, crossSystemAgeGroup, birthYearCohort, homeCutoffSystem, awayCutoffSystem }`.

5. **Update `backfillTeamAgeGender()`** — Wire in the new functions to populate all 4 new columns.

6. **Unit tests** — Cover: pre/post-transition dates, MLS NEXT vs ECNL events, cross-system game example from the spec (FC Dallas MLS NEXT U15 vs LAFC ECNL U16).

## Acceptance criteria

- [ ] Migration runs clean on local DB
- [ ] Unit tests pass (all spec examples)
- [ ] Backfill runs against full games table without errors
- [ ] `cross_system_age_group=true` count logged and reasonable (expect low % — these are rare inter-system matchups)
- [ ] Write a brief implementation note at `02 - Tiger Tournaments/Projects/Pipeline/age-gender-dual-system-implementation-2026-04-23.md`

## Why this matters

The 2026-27 season shifts ECNL from birth-year to school-year age groups. Without this, cross-system games (e.g., MLS NEXT team vs ECNL team after June 2026) will have ambiguous age group assignments that corrupt ranking computations. This must be live before DSS on May 16.
