---
type: dss-bracket-deliverable
event: Dallas Spring Showcase
deliverable: Game Coverage Report
filed_by: FORGE+ELO (run by Claude Code as the night-of executor for the autonomy directive)
date: 2026-04-28
status: completed
scope: U11 and up (per Presten directive 2026-04-28)
---

# DSS Game Coverage Report — U11+

> **Scope:** Only U11 and up. U6–U10 (217 teams) are excluded from this analysis.

## Summary

| Status | Definition | Count | % |
|--------|-----------|-------|---|
| GREEN  | ≥10 games last 365d AND last game within 60 days | 203 | 68.6 |
| YELLOW | 5–9 games OR last game 60–180 days ago | 47 | 15.9 |
| RED    | <5 games OR last game >180 days ago | 25 | 8.4 |
| MISSING | Team ID not in Evo Draw `teams` table | 21 | 7.1 |
| **Total** | | **296** | |

## By Event Age

| Event Age | Total | GREEN | YELLOW | RED | MISSING |
|-----------|-------|-------|--------|-----|---------|
| U11 | 74 | 57 | 12 | 3 | 2 |
| U12 | 53 | 41 | 7 | 2 | 3 |
| U13 | 57 | 45 | 8 | 3 | 1 |
| U14 | 35 | 18 | 8 | 8 | 1 |
| U15 | 29 | 20 | 2 | 3 | 4 |
| U16 | 22 | 11 | 5 | 5 | 1 |
| U17 | 16 | 8 | 4 | 1 | 3 |
| U18 | 4 | 3 | 0 | 0 | 1 |
| U19 | 6 | 0 | 1 | 0 | 5 |

## Methodology

For each registered team:
- Linked CSV `Team ID` directly to `games.home_team_id` / `games.away_team_id` (these columns store the GotSport external_id, NOT `teams.id` — found this during the run).
- Counted games where `match_date >= CURRENT_DATE - INTERVAL '365 days'` and `is_hidden = false`.
- MISSING = team_id from CSV not present in `teams` table at all (40 teams). These are likely teams not yet ingested into Evo Draw — FORGE should investigate post-DSS.

## Companion CSV

`Brackets/01 — Game Coverage.csv` has the per-team data with: team_id, name, club, event_age, gender, division, bracket, coverage status, games_365, distinct_opponents, last_game_date, rating, in_db.
