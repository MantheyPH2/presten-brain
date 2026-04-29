---
title: DSS Bracketing Rules
event: Dallas Spring Showcase
dates: 2026-05-16 → 2026-05-18
created: 2026-04-28
authority: Presten (verbal, 2026-04-28)
status: active
---

# DSS Bracketing Rules

These rules govern how teams are assigned to brackets and how matches are scheduled at the Dallas Spring Showcase. Source of truth for the agent team.

## Inputs

- **Registration data:** `Source/registrations-2026-04-29.csv` — 513 teams across U6–U19, both genders. Columns: Current Team Name, Club Name, Event Team Name, Event Age, Team ID, Team Age, Gender, Postal Code, Division, Bracket, Coach Name 1.
- **Ranking data:** Evo Draw `rankings` table (PostgreSQL `youth_soccer` @ localhost:5432). Linked to registrations via `Team ID`.
- **Game history:** `games` table, last 365 days, filtered by `is_hidden=false`.

## Bracket Sizes — Match Format

Match format is determined by the count of teams in each (Event Age × Division × Bracket) group:

| Size | Format |
|------|--------|
| 4 teams | Round robin, then semifinals (1v4, 2v3), then final |
| 5 teams | Round robin, then semifinals, then final |
| 6 teams | Two pools of 3, **cross-play**, top 4 → semifinals → final |
| 8 teams | Two pools of 4, 3 bracket games each, top 2 each → semifinals → final |
| 7, 9, 10 | **Not yet specified by Presten — SENTINEL decision per autonomy directive 2026-04-28-full-autonomy.** Default proposal: round-robin if ≤7, otherwise pool-then-knockout. Document chosen format. |

**Note:** Match format generation is OUT OF SCOPE for the current bracketing pass. Current pass = team-to-bracket assignment + flagging. Match schedule comes later.

## Authoritative Bracket Assignments

The registrar has already assigned most teams to Bracket A / Bracket B / Bracket C within their Division. Those assignments are **authoritative — do not move teams**. Two exceptions:

1. **Unassigned teams** (rows with empty Division and Bracket — currently 13 teams): slot by best-fit using ranking + age/gender/format. Document the rationale per team.
2. **Mismatches** (see Flagging below): **flag, do not move**. Presten reviews flagged teams and decides whether to move up/down.

## Flagging Rules — Division vs. Ranking Mismatch

For every registered team, compare the team's Evo Draw rating to the typical rating range for their assigned Division. Flag a team if its rating sits outside the expected band.

**Definition of "outside the band":** Use the league hierarchy calibration (see [[02 - Tiger Tournaments/Projects/Rankings/League Hierarchy Calibration Sanity Check — April 2026]]). A team is flagged if:

- Rated ≥1 tier above their assigned Division → flag as "consider moving up"
- Rated ≥1 tier below their assigned Division → flag as "consider moving down"
- No usable ranking (insufficient games — see below) → flag as "ranking-uncertain, manual review"

ELO produces the ranking band thresholds per Division per Event Age. Document them with the flagged-teams output.

## Game Coverage Verification

Before flagging, FORGE must verify game coverage for every team in the registration list:

- For each `Team ID` in the CSV, query the `games` table for games in the last 365 days where the team appears as home or away.
- Compute: total games, distinct opponents, last game date.
- Coverage thresholds:
  - **GREEN:** ≥10 games, last game within 60 days
  - **YELLOW:** 5–9 games OR last game 60–180 days ago
  - **RED:** <5 games OR last game >180 days ago
- RED teams cannot have a meaningful ranking comparison — they go directly to the "ranking-uncertain" flag list, no up/down recommendation.
- FORGE produces the coverage report; ELO consumes it for the ranking comparison.

## Deliverables (Current Pass)

1. **Game Coverage Report** (FORGE): per-team coverage status (GREEN/YELLOW/RED), filed at `Brackets/01 — Game Coverage Report.md` with a CSV companion.
2. **Bracket Assignment** (FORGE+ELO): final team-to-bracket assignment table for all 513 teams, including the 13 currently unassigned. Filed at `Brackets/02 — Bracket Assignments.md` with a CSV companion.
3. **Flagged Teams** (ELO): list of teams whose Evo Draw ranking suggests a different Division than registered, with the recommended direction (up/down) and the supporting numbers. Filed at `Brackets/03 — Flagged Teams for Presten Review.md`.

Match schedule generation is deferred to a later pass.

## Constraints

- Do not move any team that the registrar already assigned. Flag, don't move.
- Do not invent rules for sizes 7/9/10 without SENTINEL sign-off. Default per the table above + document the choice.
- All work product goes in `02 - Tiger Tournaments/Projects/Events/Dallas Spring Showcase/Brackets/`.
- Agents in the vault cannot run psql. FORGE writes the SQL queries and execution checklist; Presten (or whoever has DB access) runs them, then FORGE/ELO synthesize the results into the deliverables.
