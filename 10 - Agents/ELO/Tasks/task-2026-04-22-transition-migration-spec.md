---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-22
status: pending
priority: high
blocker: task-2026-04-22-brier-score-completion
---

# Task: Write Age Group Transition Migration Specification

## Context

The 2026-27 age group transition is June 1, 2026 — 40 days away. Most leagues are shifting from birth year to school year (Aug 1 - Jul 31) cutoffs. MLS NEXT is NOT shifting. This creates a dual-system environment that the ranking engine and rating records must handle correctly.

**This task is BLOCKED by `task-2026-04-22-brier-score-completion`.** Do not write the migration spec until calibration corrections for U13/U14 (and any other miscalibrated age groups) are confirmed and approved. The migration script will carry forward current ratings — if those ratings are built on miscalibrated parameters, the error is locked in for the 2026-27 season.

Once the calibration task is done, proceed with this spec immediately.

## Scope

This is a SPECIFICATION task. You are writing a detailed technical document that Presten will review and approve before any code is written or any migration runs on production. Do not implement anything as part of this task.

## Required Work

### Section 1: Rating Carryover Logic

Define exactly which ratings carry forward and which are reset:

**Carry forward (with adjustment):**
- Teams that played 10+ rated matches (Glicko-2 phase) — carry forward the Glicko-2 rating with a season-start RD expansion (see Section 2)
- The carryover rule must account for the age group change: a U15 team in 2025-26 becomes a U16 team in 2026-27. Their rating carries forward but their age group bucket changes.

**Reset to starting rating:**
- Teams in their first season (fewer than 10 rated matches — still in Elo provisional phase) — reset to age group starting rating for the new age group
- Teams with no games in the final 60 days of the 2025-26 season — reset (inactive teams should not carry ratings into the new season)

Document any edge cases:
- A team that played U15 games in ECNL (school year) AND U15 games in MLS NEXT (birth year) — which age group do they carry into 2026-27?
- A team that crossed the 10-match threshold in the final week of the season — do they get Glicko-2 carryover or provisional reset?

### Section 2: RD Reset Values by Age Group

At the start of a new season, Glicko-2 prescribes expanding RD to account for the uncertainty introduced by offseason inactivity. Define the RD reset expansion factor for each age group.

The current RD floor values from `calibration.json` are the minimum — the reset expansion should bring RDs UP from their end-of-season values to reflect the new season's uncertainty. Use the updated calibration values from the Brier score task.

Format:
```
Age Group | End-of-season RD (typical) | Season-start RD reset | Notes
U13       | ?                          | ?                      | Post-calibration-fix value
U14       | ?                          | ?                      | Post-calibration-fix value
...
```

### Section 3: birth_year_cohort Column Addition

ELO's briefing recommended adding a `birth_year_cohort` column to rating records. Strategic Insights also calls for this on the games table. Specify both:

**For rating records:**
```sql
ALTER TABLE rankings ADD COLUMN birth_year_cohort INTEGER NULL;
-- e.g., a 2011-born player's team → birth_year_cohort = 2011
-- a "U13" team in 2025-26 under birth-year rules → birth_year_cohort = 2013
```

Define the mapping rule: given an `age_group` label (e.g., "U13") and a `season` (e.g., "2025-2026"), how is `birth_year_cohort` calculated? Handle both birth-year leagues (MLS NEXT) and school-year leagues (ECNL post-migration).

**For game records:**
```sql
ALTER TABLE games ADD COLUMN birth_year_cohort INTEGER NULL;
```

Same mapping logic, applied during pipeline Step 2 or 3 (backfill-age-gender or final-backfill).

### Section 4: Dual-System Age Group Handling

For games played in 2026-27 where a "U16 ECNL" team (school year) plays a "U15 MLS NEXT" team (birth year) at a showcase:
- How does the ranking engine determine which age group bucket this game is scored in?
- Propose: the game is scored in the age group of the EVENT (the tournament sets the bracket), and `birth_year_cohort` on the game record captures the actual birth cohort of the teams for cross-league comparability.

Document any parsing rule changes needed in `backfill-age-gender.js` or `final-backfill.js` to correctly tag dual-system age groups.

### Section 5: Rollback Plan

In case calibration corrections are still under revision on June 1:
- What is the minimum viable migration? (i.e., what can run safely even without perfect calibration?)
- What is the rollback mechanism if the migration produces obviously wrong results? (e.g., top-ranked teams suddenly appearing at rank 500)
- What monitoring should be in place immediately post-migration? (suggest: run Brier score check on first week of 2026-27 games against the new calibration values)

## Deliverable

Write the full specification to `Rankings/Age Group Transition Migration Spec 2026-06-01.md`. It should be detailed enough that Presten can read it, ask questions, approve it, and hand it to any technically capable person to implement.

Include:
- All five sections above
- A pre-migration checklist (confirm calibration fixes applied, confirm birth_year_cohort backfill is ready, confirm rollback plan is in place)
- Estimated pipeline run time for the migration (full recalculation will be required post-migration)

Target completion: **2026-04-30** (after Brier score task is done by 2026-04-25, this gives 5 days for the spec).

## References

- [[Ranking Engine]] — all 9+1 phases, K-factor and RD parameters
- [[Age Groups and Birth Years]] — mapping table and format rules
- [[Recent Changes 2024-2026]] — age group shift documentation
- [[Parsing Rules]] — age/gender parsing that needs dual-system updates
- [[Database Schema]] — current schema to be extended
- [[Data Pipeline]] — where new pipeline steps integrate
- [[Strategic Insights]] — Insight 3 (age group shift cross-season comparison risk) and Action Item 4 (birth_year_cohort column)
