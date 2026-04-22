---
title: MLS NEXT Source
aliases: [MLS NEXT Data, MLS NEXT Games]
tags: [data-source, mls-next, evo-draw]
created: 2026-04-21
updated: 2026-04-21
source_priority: 4
total_games: 27635
visible_games: 21580
---

# MLS NEXT Source

The MLS NEXT data source contains games specifically tagged as [[MLS NEXT]] league play. This is a curated subset — many MLS NEXT games also appear in [[GotSport]] data.

---

## Volume

| Metric        | Value  |
| ------------- | ------ |
| Total games   | 27,635 |
| Visible games | 21,580 |
| Hidden (dedup)| 6,055  |
| Source priority| 4      |

The ~6K hidden games are duplicates that also exist in higher-priority sources ([[GotSport]] API or HTML). See [[Dedup Strategy]].

---

## Scraper

- **File:** `crawl-mlsnext.js`
- Scrapes MLS NEXT-specific game data
- Source ID format: `mlsnext_{matchId}`

---

## Relationship to GotSport

Many MLS NEXT events are also hosted on [[GotSport]], meaning the same games may be captured by both the MLS NEXT scraper and the [[GotSport API Scraper]] or [[GotSport HTML Scraper]]. The [[Dedup Strategy]] resolves this — GotSport versions (priority 1-2) take precedence over MLS NEXT (priority 4).

---

## Related Notes
- [[MLS NEXT]] — league details, structure, and tier information
- [[Data Sources Overview]] — all sources comparison
- [[Dedup Strategy]] — cross-source dedup handling
- [[League Hierarchy]] — MLS NEXT as Tier 1 Elite
- [[Ranking Engine]] — MLS NEXT calibration value: 160 points
