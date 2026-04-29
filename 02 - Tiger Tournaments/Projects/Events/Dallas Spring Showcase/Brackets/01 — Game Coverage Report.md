---
type: dss-bracket-deliverable
event: Dallas Spring Showcase
deliverable: Game Coverage Report
filed_by: FORGE+ELO (run by Claude Code as the night-of executor for the autonomy directive)
date: 2026-04-28
status: completed
---

# DSS Game Coverage Report

## Summary

| Status | Definition | Count | % |
|--------|-----------|-------|---|
| GREEN  | ≥10 games last 365d AND last game within 60 days | 357 | 69.6 |
| YELLOW | 5–9 games OR last game 60–180 days ago | 79 | 15.4 |
| RED    | <5 games OR last game >180 days ago | 37 | 7.2 |
| MISSING | Team ID not in Evo Draw `teams` table | 40 | 7.8 |
| **Total** | | **513** | |

## By Event Age

| Event Age | Total | GREEN | YELLOW | RED | MISSING |
|-----------|-------|-------|--------|-----|---------|
| U10 | 75 | 59 | 10 | 5 | 1 |
| U11 | 74 | 57 | 12 | 3 | 2 |
| U12 | 53 | 41 | 7 | 2 | 3 |
| U13 | 57 | 45 | 8 | 3 | 1 |
| U14 | 35 | 18 | 8 | 8 | 1 |
| U15 | 29 | 20 | 2 | 3 | 4 |
| U16 | 22 | 11 | 5 | 5 | 1 |
| U17 | 16 | 8 | 4 | 1 | 3 |
| U18 | 4 | 3 | 0 | 0 | 1 |
| U19 | 6 | 0 | 1 | 0 | 5 |
| U6 | 12 | 3 | 2 | 0 | 7 |
| U7 | 29 | 23 | 3 | 1 | 2 |
| U8 | 53 | 34 | 9 | 4 | 6 |
| U9 | 48 | 35 | 8 | 2 | 3 |

## Methodology

For each registered team:
- Linked CSV `Team ID` directly to `games.home_team_id` / `games.away_team_id` (these columns store the GotSport external_id, NOT `teams.id` — found this during the run).
- Counted games where `match_date >= CURRENT_DATE - INTERVAL '365 days'` and `is_hidden = false`.
- MISSING = team_id from CSV not present in `teams` table at all (40 teams). These are likely teams not yet ingested into Evo Draw — FORGE should investigate post-DSS.

## Companion CSV

`Brackets/01 — Game Coverage.csv` has the per-team data with: team_id, name, club, event_age, gender, division, bracket, coverage status, games_365, distinct_opponents, last_game_date, rating, in_db.
