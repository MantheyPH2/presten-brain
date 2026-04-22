---
title: TGS AthleteOne
aliases: [TGS, AthleteOne, The Game Source]
tags: [data-source, tgs, athleteone, ecnl, evo-draw]
created: 2026-04-21
updated: 2026-04-21
source_priority: 3
total_games: 91401
visible_games: 91345
---

# TGS (AthleteOne)

TGS (The Game Source) operates via the **AthleteOne** platform, providing JSON REST API access to event and game data. It is the exclusive data source for [[ECNL]] league games until the June 2026 migration to [[GotSport]].

---

## Technical Details

| Field        | Value                                      |
| ------------ | ------------------------------------------ |
| Endpoint     | `https://api.athleteone.com/api/Event`     |
| Format       | JSON REST                                  |
| Auth         | None required                              |
| Rate limit   | 3-second delay between requests            |
| Scraper file | `crawl-tgs.js` — see [[TGS Scraper]]      |
| Source ID     | `tgs_{eventId}_{gameId}`                   |
| Priority     | 3 (per [[Dedup Strategy]])                 |

---

## Division Format

TGS uses **birth year** format for divisions rather than age groups:
- `B2011` = Boys born 2011
- `G2013` = Girls born 2013

These are converted to standard age groups (U15, U13, etc.) by the [[Parsing Rules|backfill pipeline]]. See [[Age Groups and Birth Years]] for the full mapping table.

---

## Event Filtering

- Events before **2025** are skipped entirely
- Non-US events are filtered out
- Run command: `TGS_START=3823 TGS_END=999999 node crawl-tgs.js`

---

## Critical Bug Fix

> [!warning] Null-Name Event Bug
> When the scraper encounters events with `null` names, it **must increment** the `emptyStreak` counter, NOT reset it to 0. Resetting causes the scraper to loop indefinitely through thousands of empty event IDs instead of stopping at the configured threshold. This was a significant production bug.

```
// CORRECT:
if (event.name === null) { emptyStreak++; continue; }

// WRONG (causes infinite loop):
if (event.name === null) { emptyStreak = 0; continue; }
```

---

## ECNL Migration Impact

> [!info] Platform Migration — June 2026
> [[ECNL]] is migrating from AthleteOne to [[GotSport]] in June 2026. After migration:
> - New ECNL games will appear via [[GotSport API Scraper]] instead of TGS
> - Historical TGS data remains in the database
> - The [[TGS Scraper]] may become obsolete for ECNL but could still be useful for other AthleteOne-hosted leagues
> - [[Dedup Strategy]] will handle any overlap during the transition period

---

## Volume

- **91,401** total games
- **91,345** visible (56 hidden as duplicates)
- Covers [[ECNL]] boys and girls league play primarily

---

## Related Notes
- [[Data Sources Overview]] — all sources comparison
- [[TGS Scraper]] — scraper implementation details
- [[ECNL]] — the primary league served by this source
- [[Parsing Rules]] — birth year to age group conversion
- [[Age Groups and Birth Years]] — B2011 → U15 mapping
