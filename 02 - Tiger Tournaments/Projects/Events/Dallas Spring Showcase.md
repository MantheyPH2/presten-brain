---
title: Dallas Spring Showcase
aliases: [DSS, Dallas Showcase, MoneyGram Park]
tags: [event, dss, dallas, tournament, managed-event, evo-draw]
created: 2026-04-21
updated: 2026-04-21
slug: dallas-spring-showcase
gotsport_event_id: 31228
location: MoneyGram Park, Dallas TX
dates: May 16-18, 2026
teams_registered: 308
teams_target: 390
---

# Dallas Spring Showcase (DSS)

The Dallas Spring Showcase is the **primary managed event** for Evo Draw, serving as the flagship showcase for the platform's ranking and bracket generation capabilities.

---

## Event Details

| Field              | Value                         |
| ------------------ | ----------------------------- |
| Slug               | `dallas-spring-showcase`      |
| Location           | MoneyGram Park, Dallas TX     |
| Dates              | May 16-18, 2026               |
| GotSport Event ID  | 31228                         |
| Teams registered   | 308 (target: 390)             |
| Brackets generated | 24 (across U7-U19 Boys & Girls) |

---

## Division Breakdown

| Division       | Teams | Tier Level |
| -------------- | ----- | ---------- |
| Bronze         | 75    | 4th        |
| Silver         | 71    | 3rd        |
| Bronze II      | 66    | 5th        |
| Gold           | 53    | 1st        |
| Silver Elite   | 24    | 2nd        |
| Recreational   | 19    | 6th        |
| **Total**      | **308** |          |

Division hierarchy: Gold > Silver Elite > Silver > Bronze > Bronze II > Recreational
See [[Division Calibration]] for how these map to ranking adjustments.

---

## Team Data Coverage

| Metric                     | Value   |
| -------------------------- | ------- |
| Teams with game history    | 295/308 (95.8%) |
| Total historical games     | 10,234  |
| Teams with 0 completed games | 13   |
| Game history file          | `data/dss-team-history.json` |

> [!info] Pre-Generated History
> Team game history is pre-generated and cached at `data/dss-team-history.json` for fast bracket generation. This avoids querying the full games table at bracket-creation time.

---

## Team Name Matching Challenge

> [!warning] GotSport Name Mismatches
> [[GotSport]] API returns different team names than DSS registration. For example:
> - DSS: "Toros FC 2017 Black" vs GotSport: "EPPL Toros FC Black 2017"
>
> **Solution:** Look up by GotSport team ID (`home_team_id`/`away_team_id`) instead of exact name match. See [[Team Name Matching]] for the full 24-mismatch resolution and the `dss-name-aliases.json` mapping file.

---

## API Endpoints

All served by the [[Server and Frontend|Node.js server]] on port 3000:

| Endpoint                              | Method | Purpose                |
| ------------------------------------- | ------ | ---------------------- |
| `/api/managed-events/:slug`           | GET    | Event metadata         |
| `/api/managed-events/:slug/generate-brackets` | POST | Generate brackets |
| `/api/managed-events/:slug/team-history` | GET | Single team history   |
| `/api/managed-events/:slug/all-team-history` | GET | All teams history  |

---

## Related Notes
- [[Division Calibration]] — how divisions affect rankings
- [[Ranking Engine]] — Phase 9 applies DSS division calibration
- [[Team Name Matching]] — resolving name mismatches
- [[Database Schema]] — `managed_events` and `managed_event_teams` tables
- [[Server and Frontend]] — API endpoints serving DSS data
- [[GotSport]] — source for team game history
- [[Data Quality]] — 13 teams with zero games
