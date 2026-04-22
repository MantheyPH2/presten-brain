---
title: Data Sources Overview
aliases: [Data Sources, Sources MOC]
tags: [moc, data-sources, scraping, evo-draw]
created: 2026-04-21
updated: 2026-04-21
---

# Data Sources Overview

Evo Draw aggregates youth soccer game data from **7 sources**. Each source has a priority level used in [[Dedup Strategy]] — lower number = higher priority. All game counts reflect the `games` table in the [[Database Schema|youth_soccer database]].

---

## Volume by Source

| Source                      | Priority | Total Games | Visible Games | Scraper                          |
| --------------------------- | -------- | ----------- | ------------- | -------------------------------- |
| [[GotSport]] (API + HTML)  | 1 / 2    | 1,489,861   | 1,487,049     | [[GotSport API Scraper]] / [[GotSport HTML Scraper]] |
| [[TGS AthleteOne]]         | 3        | 91,401      | 91,345        | [[TGS Scraper]]                  |
| [[Sinc Sports]]            | 5        | 34,961      | 29,827        | Bulk import (no live scraper)    |
| [[MLS NEXT Source]]        | 4        | 27,635      | 21,580        | `crawl-mlsnext.js`               |
| Coast                       | 6        | 5,934       | 5,934         | Via [[Sinc Sports]] pipeline     |
| Heartland                   | 7        | 1,291       | 1,291         | Via [[Sinc Sports]] pipeline     |
| Squadi                      | 8        | 84          | 84            | Via [[Sinc Sports]] pipeline     |
| **TOTAL**                   |          | **~1,651,167** | **~1,637,110** |                              |

> [!info] Visible vs Total
> Games with `is_hidden=true` are duplicates identified by the [[Dedup Strategy]]. All site queries filter on `WHERE is_hidden=false`. The gap (~14K) represents cross-source duplicates where lower-priority versions are hidden.

---

## Source Architecture

```
GotSport API (pri 1) ──┐
GotSport HTML (pri 2) ─┤
TGS/AthleteOne (pri 3) ┤──→ games table ──→ [[Data Pipeline]] ──→ [[Ranking Engine]]
MLS NEXT (pri 4) ───────┤
Sinc (pri 5) ───────────┤
Coast (pri 6) ──────────┤
Heartland (pri 7) ──────┤
Squadi (pri 8) ─────────┘
```

Each scraper inserts into the `games` table with a unique `(source, source_id)` pair. The [[Data Pipeline]] then normalizes, backfills missing fields, and deduplicates.

---

## Source ID Formats

| Source    | Format                            | Example                          |
| --------- | --------------------------------- | -------------------------------- |
| GotSport API | `gs_api_{eventId}_{matchId}`  | `gs_api_31228_4587321`           |
| GotSport HTML | `gs_{eventId}_{matchNum}`    | `gs_45488_127`                   |
| TGS       | `tgs_{eventId}_{gameId}`          | `tgs_3900_18234`                 |
| MLS NEXT  | `mlsnext_{matchId}`               | `mlsnext_98712`                  |
| Sinc      | `sinc_{gameId}`                   | `sinc_44321`                     |

---

## Key Relationships

- [[GotSport]] provides ~90% of all data and is the PRIMARY source
- [[TGS AthleteOne]] covers [[ECNL]] exclusively until June 2026 platform migration
- [[Sinc Sports]] is the umbrella importer for Coast, Heartland, and Squadi
- [[MLS NEXT Source]] is a tagged subset, often duplicated in [[GotSport]] data
- Source priority drives the [[Dedup Strategy]] — when the same game appears in multiple sources, the highest-priority version wins

> [!warning] ECNL Migration
> [[ECNL]] is moving from [[TGS AthleteOne]] to [[GotSport]] in **June 2026**. This will shift significant volume from source priority 3 to priority 1. The [[TGS Scraper]] may become obsolete for ECNL data after migration.
