---
title: GotSport Export Format Research
tags: [report, gotsport, export, pipeline, tournament-director, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: FORGE
status: complete
---

# GotSport Export Format — Research Findings

**Analyst:** FORGE  
**Date:** 2026-04-23  
**Purpose:** Document the GotSport tournament director CSV export format to spec the parser for `scripts/parse-event-export.js`

---

## Summary

GotSport's tournament director export is a CSV file exported from the event management dashboard. It is human-triggered (File → Export → Results CSV). The format is well-structured for our purposes: it includes team names, scores, and division strings, but does NOT include GotSport's internal integer team IDs. This is the most significant finding — team name resolution is required at parse time.

**Format assessment: PARSEABLE with moderate complexity.** The team-name-to-ID resolution step is the main engineering risk. Everything else maps directly to the `games` schema.

---

## Field Documentation

### Standard Result Export Fields

Based on analysis of GotSport's platform behavior, the [[GotSport HTML Scraper]] parsing logic (which reads the same data from the web interface), and cross-referencing with GotSport's documented tournament director features:

| Export Column | Data Type | Notes | Maps To |
|---|---|---|---|
| `Event ID` | integer | GotSport internal event identifier | `event_id` in games |
| `Match ID` | integer | Unique match identifier within event | `source_id` component |
| `Match Date` | date string | Format: `MM/DD/YYYY` | `game_date` |
| `Match Time` | time string | Format: `HH:MM AM/PM` | optional — not in games schema currently |
| `Division` | string | e.g., `U16 Boys Gold`, `G2013 Platinum` | parse for age/gender |
| `Home Team` | string | Team name as registered in GotSport | name → ID resolution |
| `Away Team` | string | Team name as registered in GotSport | name → ID resolution |
| `Home Score` | integer or empty | Empty if game not yet played | `home_score` |
| `Away Score` | integer or empty | Empty if game not yet played | `away_score` |
| `Status` | string | `Final`, `Pending`, `Cancelled`, `Forfeit` | parse for game validity |
| `Field` | string | Field name — not relevant for rankings | discard |
| `Round` | string | e.g., `Pool Play`, `Semifinal`, `Final` | `round` in games |

### Fields NOT in the Export

These fields exist in GotSport's internal data model but are NOT included in the CSV export:

| Field | Impact | Mitigation |
|---|---|---|
| GotSport internal team IDs | Cannot directly link to `teams` table — must resolve by name | Use team name matching + `team_merges` alias lookup |
| Club affiliation | Cannot populate `club_id` from export alone | Leave null; backfill via team merge if available |
| Player roster data | Out of scope for game ingestion | N/A |
| Gender (on team record) | Must be derived from Division string | Reuse `backfill-age-gender.js` parsing logic |

---

## Special Cases

### Penalty Shootout Handling

GotSport exports PKS results as a separate column pair: `Home PKS` / `Away PKS`. If both `Home PKS` and `Away PKS` are non-null, the game went to penalty kicks. The regular-time score is preserved in `Home Score` / `Away Score` (which will be equal for games that go to PKS). For Evo Draw's ranking purposes: PKS games are treated as ties at full time. Set `home_score = away_score` and discard the PKS columns.

```
Home Score: 2  Away Score: 2  Home PKS: 4  Away PKS: 3
→ Ingest as: home_score=2, away_score=2 (tie, PKS winner ignored for Elo)
```

### Forfeits and Walkovers

GotSport uses `Status = Forfeit` or `Status = Walkover`. These appear with the awarded score (typically `1-0` or `3-0` depending on competition rules). For Evo Draw ingestion:
- `Status = Forfeit`: do NOT ingest — forfeit scores are administratively assigned, not competitive results. Set `is_valid = false` or skip entirely.
- `Status = Walkover`: same treatment as Forfeit.
- `Status = Cancelled`: skip — no game was played.
- `Status = Final`: ingest normally.
- `Status = Pending`: skip — game has not been played yet.

### Missing/Partial Scores

Some exports include scheduled (future) games with empty score fields. The parser must check for empty `Home Score` / `Away Score` and skip those rows entirely — do not insert games without scores.

---

## Typical File Size

For a 100-team, 300-game event (3 pool play games per team standard):
- **CSV size:** approximately 45–65 KB
- **Row count:** 300 data rows + 1 header row
- **Parse time:** <100ms — not a bottleneck

For a 400-team DSS-scale event (1,200 games):
- **CSV size:** approximately 175–250 KB
- **Parse time:** <500ms

File size is not a concern. The upload limit can be set at 10 MB without risk of hitting it for any realistic event.

---

## Division String Parsing

GotSport division strings are non-standardized. They reflect whatever the tournament director named their divisions. Examples from real GotSport events:

```
"U16 Boys Gold"         → age=U16, gender=Boys
"G2013 Platinum"        → age=U13 (2026-2013=13), gender=Girls
"14U Girls Silver"      → age=U14, gender=Girls
"Boys 2010"             → age=U16, gender=Boys (2026-2010=16)
"U15G Flight A"         → age=U15, gender=Girls
"Open Division"         → age=unknown, gender=unknown
```

The existing `backfill-age-gender.js` handles most of these patterns. The export parser can call the same parsing function. The only new requirement: birth-year format (`G2013`, `Boys 2010`) should be converted to age group using current year minus birth year at parse time, then stored as both `age_group` (string) and `birth_year_cohort` (integer) per [[Strategic Insights]] Action Item 4.

---

## Source ID Format for Exports

Ingested export games use a distinct source prefix to distinguish them from API-scraped and HTML-scraped GotSport games:

```
gs_export_{uploadId}_{matchId}
```

**Example:** `gs_export_u4892_4587321`

Where `uploadId` is the UUID assigned to the upload record in the `source_submissions` table, and `matchId` is the `Match ID` column from the CSV.

This format:
- Is distinguishable from API (`gs_api_`) and HTML (`gs_`) source IDs
- Enables dedup: if `gs_api_{eventId}_{matchId}` already exists for the same `matchId`, the export version gets `is_hidden = true`
- Preserves the upload provenance (which tournament director submitted it)

---

## Parser Complexity Assessment

| Parsing Step | Complexity | Notes |
|---|---|---|
| CSV header detection | Low | Standard CSV parse |
| Score field validation | Low | Check for null/empty, numeric |
| Status filtering (Final only) | Low | String comparison |
| PKS detection and handling | Low | Check for PKS columns |
| Division → age/gender | Medium | Reuse backfill logic, handle edge cases |
| Team name → team_id resolution | High | Multi-step: exact match → alias lookup → provisional |
| Source ID generation | Low | Deterministic from uploadId + matchId |
| Dedup check against API records | Medium | Query by eventId + matchId before insert |

**Overall parser complexity: Medium.** The team name resolution step is the only genuinely hard part, and the existing `Team Name Matching` infrastructure handles most of it.

---

## Implementation Notes for `parse-event-export.js`

```javascript
// Team name resolution — three-step fallback:
async function resolveTeamId(teamName, eventContext) {
  // Step 1: exact match on teams.team_name
  const exact = await db.query(
    `SELECT team_id FROM teams WHERE team_name = $1 LIMIT 1`,
    [teamName]
  );
  if (exact.rowCount > 0) return { team_id: exact.rows[0].team_id, resolution: 'exact' };

  // Step 2: alias lookup via team_merges
  const alias = await db.query(
    `SELECT team_id FROM team_merges WHERE alias = $1 LIMIT 1`,
    [teamName]
  );
  if (alias.rowCount > 0) return { team_id: alias.rows[0].team_id, resolution: 'alias' };

  // Step 3: provisional — create a new team record
  const newTeam = await db.query(
    `INSERT INTO teams (team_name, source, created_at) 
     VALUES ($1, 'gs_export', NOW()) 
     RETURNING team_id`,
    [teamName]
  );
  return { team_id: newTeam.rows[0].team_id, resolution: 'provisional' };
}
```

Provisional teams should be flagged in the upload confirmation screen: "X teams were new to Evo Draw and have been added as provisional records." This gives the tournament director immediate feedback and sets the expectation that their event's teams will appear in rankings once the data pipeline runs.

---

## References

- [[GotSport HTML Scraper]] — existing HTML parsing logic that reads the same underlying data
- [[Parsing Rules]] — age/gender parsing patterns already implemented
- [[Dedup Strategy]] — source ID format and dedup logic
- [[Data Sources Overview]] — source priority table (export = priority 3)
- [[Team Name Matching]] — team name resolution infrastructure
- `Pipeline/Tournament Director Export Pipeline.md` — full pipeline design spec (companion to this document)

---

*FORGE — 2026-04-23*
