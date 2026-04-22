---
type: agent-queue
agent: ELO
category: recommendation
priority: medium
date: 2026-04-21
status: resolved
answer: ""
---

# Recommendation: Add `birth_year_cohort` Column for Age Group Transition

## Context

The 2026-27 age group transition is scheduled for June 1, 2026. Every year, teams and players advance one age bracket — a U13 team becomes U14, U14 becomes U15, and so on. The ranking engine needs to carry forward ratings into the new age group while simultaneously:

1. Preserving historical rating records tagged to the correct age group for each season
2. Allowing retrospective queries like "what was this team's U13 rating at end of season 2025-26?"
3. Preventing rating record collisions when a team's `age_group` field changes but their historical records remain in the same table

Currently, the `team_ratings` table schema looks like this:

```
team_ratings
  - team_id       (FK → teams)
  - age_group     (varchar: 'U13', 'U14', etc.)
  - rating        (float)
  - rd            (float)
  - volatility    (float)
  - match_count   (int)
  - last_updated  (timestamp)
```

The problem: `age_group` is a mutable field on the team record. When a team is promoted to U14, historical records tagged with their old `age_group` value become ambiguous — it's impossible to tell whether a record was generated when the team was genuinely competing as U13 or whether it's a post-migration artifact.

## Recommendation

Add a `birth_year_cohort` column (integer, e.g. `2013` for a team whose players were born in 2013) to `team_ratings`. This value is immutable and directly tied to the actual players, not the administrative age group label. It provides a stable, season-independent key for historical queries.

Additionally, add a `season` column (varchar, e.g. `'2025-26'`) to make multi-season queries explicit and clean.

Proposed new schema:

```
team_ratings
  - team_id             (FK → teams)
  - birth_year_cohort   (int) — NEW
  - season              (varchar) — NEW
  - age_group           (varchar) — now a derived/display field, not the primary key
  - rating              (float)
  - rd                  (float)
  - volatility          (float)
  - match_count         (int)
  - last_updated        (timestamp)
```

The composite key for a rating record would become `(team_id, birth_year_cohort, season)` rather than `(team_id, age_group)`.

## Why This Matters Before June 1

If the migration runs without this schema change, the transition script will likely overwrite current rating records with updated age group labels, destroying the ability to query last season's ratings accurately. The fix is straightforward, but the migration script needs to be written against the new schema — so the sooner this is approved, the more lead time there is before the transition deadline.

## Estimated Effort

- Schema migration: ~1 hour
- Backfill of `birth_year_cohort` on existing records (derivable from current `age_group` + `last_updated` season): ~2-3 hours
- Update to transition script: ~2 hours
- Testing: ~1 day

Total: approximately 2 days of work. Medium urgency given the June 1 deadline.

## Request

Approval to proceed with the schema change, or feedback if a different approach is preferred.
