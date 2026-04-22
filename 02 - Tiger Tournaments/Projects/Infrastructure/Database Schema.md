---
title: Database Schema
aliases: [Database, PostgreSQL, youth_soccer, DB Schema]
tags: [infrastructure, database, postgresql, schema, evo-draw]
created: 2026-04-21
updated: 2026-04-21
database: youth_soccer
host: localhost
port: 5432
user: p
---

# Database Schema

The Evo Draw platform uses **PostgreSQL** with a database named `youth_soccer`.

---

## Connection

| Parameter | Value        |
| --------- | ------------ |
| Host      | localhost    |
| Port      | 5432         |
| Database  | youth_soccer |
| User      | p            |

---

## Core Tables

### games (Primary Table)
The central table with **44 columns**. Upsert key: `(source, source_id)`.

**Key columns:**

| Column           | Type    | Purpose                                    |
| ---------------- | ------- | ------------------------------------------ |
| `source`         | text    | Data source identifier                     |
| `source_id`      | text    | Unique ID within source                    |
| `source_priority`| integer | Priority for [[Dedup Strategy]] (1=highest)|
| `is_hidden`      | boolean | `true` if duplicate (filtered in queries)  |
| `duplicate_of`   | text    | Points to canonical game ID                |
| `home_team_name` | text    | Home team display name                     |
| `away_team_name` | text    | Away team display name                     |
| `home_team_id`   | text    | Home team platform ID                      |
| `away_team_id`   | text    | Away team platform ID                      |
| `home_score`     | integer | Home team score                            |
| `away_score`     | integer | Away team score                            |
| `event_name`     | text    | Tournament/event name                      |
| `division`       | text    | Division within event                      |
| `age_group`      | text    | Normalized: U7-U19                         |
| `gender`         | text    | Boys/Girls                                 |
| `round`          | text    | Normalized round (see [[Parsing Rules]])   |
| `game_status`    | text    | completed/scheduled/cancelled              |
| `match_date`     | date    | Game date                                  |
| `season`         | text    | "2025-2026" format                         |
| `event_tier`     | text    | League tier classification                 |

> [!tip] Schema-Adaptive Insertion
> Scrapers query `information_schema.columns` at startup to discover available columns. This allows scrapers to work even if new columns are added — they only insert into columns that exist. No schema migration needed for new scraper fields.

### teams
| Attribute     | Value    |
| ------------- | -------- |
| Total entries | 144,676  |
| Key fields    | team ID, team name, coach names, club |

Coach names are critical for [[Dedup Strategy|cross-platform team matching]].

### team_merges
| Attribute     | Value    |
| ------------- | -------- |
| Total rows    | 27,820   |
| Verified      | 18,684   |
| Purpose       | Links same team across platforms |

Used by [[Ranking Engine]] Phase 1. See [[Dedup Strategy]] for merge logic.

### managed_events
Stores managed tournament configurations like [[Dallas Spring Showcase]].
- Event slug, name, dates, location
- GotSport event ID mapping
- Bracket configuration

### managed_event_teams
Links teams to managed events with division placements.
- Team ID, event slug, division assignment
- Used by [[Division Calibration]] in [[Ranking Engine]] Phase 9

### rankings
Output table for [[Ranking Engine]].
- Team ID, Elo rating, rank position
- Age group and gender breakdowns
- Updated each time `compute-rankings.js` runs

---

## Key Query Patterns

```sql
-- All visible games (standard filter)
SELECT * FROM games WHERE is_hidden = false;

-- Games for a specific team by GotSport ID
SELECT * FROM games
WHERE (home_team_id = '12345' OR away_team_id = '12345')
  AND is_hidden = false;

-- Cross-source duplicate check
SELECT source, source_id, duplicate_of
FROM games
WHERE duplicate_of IS NOT NULL;

-- Team merge lookup
SELECT * FROM team_merges
WHERE team_id = '12345' OR merged_team_id = '12345';
```

> [!warning] Always Filter is_hidden
> Every user-facing query must include `WHERE is_hidden = false`. Without this filter, duplicate games will inflate statistics and corrupt rankings. The [[Server and Frontend|server API]] enforces this automatically.

---

## Related Notes
- [[Data Pipeline]] — processes that write to these tables
- [[Dedup Strategy]] — `is_hidden`, `duplicate_of`, `team_merges` usage
- [[Ranking Engine]] — reads games + merges, writes rankings
- [[Server and Frontend]] — API layer querying these tables
- [[Dallas Spring Showcase]] — `managed_events` + `managed_event_teams`
- [[Data Quality]] — completeness metrics across columns
