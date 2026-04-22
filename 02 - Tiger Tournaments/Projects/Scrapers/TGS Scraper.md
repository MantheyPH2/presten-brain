---
title: TGS Scraper
aliases: [crawl-tgs.js, AthleteOne Scraper]
tags: [scraper, tgs, athleteone, json, evo-draw]
created: 2026-04-21
updated: 2026-04-21
file: crawl-tgs.js
source_priority: 3
---

# TGS Scraper

Scrapes game data from the [[TGS AthleteOne]] JSON API. Primary source for [[ECNL]] league data until the June 2026 platform migration to [[GotSport]].

---

## Technical Details

| Attribute     | Value                                  |
| ------------- | -------------------------------------- |
| File          | `crawl-tgs.js`                         |
| API endpoint  | `https://api.athleteone.com/api/Event` |
| Format        | JSON REST                              |
| Rate limit    | **3-second delay** between requests    |
| Auth          | None required                          |
| Source priority | 3                                    |

---

## Running

```bash
TGS_START=3823 TGS_END=999999 node crawl-tgs.js
```

| Env Variable | Purpose                      |
| ------------ | ---------------------------- |
| `TGS_START`  | Starting event ID to scan    |
| `TGS_END`    | Ending event ID to scan      |

The scraper iterates through event IDs sequentially from `TGS_START` to `TGS_END`.

---

## Division Format

TGS uses birth year divisions: `B2011` (Boys 2011), `G2013` (Girls 2013). These are converted to standard age groups by the [[Parsing Rules|backfill pipeline]]. See [[Age Groups and Birth Years]] for the mapping.

---

## Event Filtering

- Events before **2025** are skipped
- Non-US events are filtered out
- Null-name events are handled with the empty streak counter

---

## Critical Bug: Null-Name Events

> [!warning] MUST READ — Production Bug
> When encountering events with `null` names, the scraper **must increment** `emptyStreak`, NOT reset it to 0.
>
> **Correct:**
> ```javascript
> if (event.name === null) { emptyStreak++; continue; }
> ```
>
> **WRONG (causes infinite loop):**
> ```javascript
> if (event.name === null) { emptyStreak = 0; continue; }
> ```
>
> Resetting the counter causes the scraper to loop through thousands of empty event IDs indefinitely, never reaching the streak threshold that triggers a stop. This was a significant production bug that wasted hours of scraping time.

---

## Future: ECNL Migration

After [[ECNL]]'s migration to [[GotSport]] in June 2026, this scraper may become largely obsolete for ECNL data. However, it could still be useful for other leagues hosted on the AthleteOne platform. See [[TGS AthleteOne]] for migration details.

---

## Related Notes
- [[TGS AthleteOne]] — data source overview (91,401 games)
- [[ECNL]] — primary league served
- [[Data Sources Overview]] — priority 3 source
- [[Parsing Rules]] — birth year division conversion
- [[Age Groups and Birth Years]] — B2011 → U15 mapping
