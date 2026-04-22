---
title: Sinc Sports
aliases: [Sinc, SincSports]
tags: [data-source, sinc, bulk-import, evo-draw]
created: 2026-04-21
updated: 2026-04-21
source_priority: 5
total_games: 34961
visible_games: 29827
---

# Sinc Sports

Sinc Sports is a tournament management platform that is **Cloudflare-protected**, making live scraping infeasible. All data was obtained via **bulk import**.

---

## Current Status

| Field             | Value                         |
| ----------------- | ----------------------------- |
| Live scraper      | **None** (Cloudflare blocked) |
| Import method     | Bulk data import              |
| Source priority    | 5 (per [[Dedup Strategy]])    |
| Total games       | 34,961                        |
| Visible games     | 29,827                        |
| Hidden (deduped)  | 5,134                         |

> [!warning] No Live Scraper
> Cloudflare's bot protection blocks automated access to Sinc Sports. Any future data updates would need to be manually exported or obtained through a partnership/API agreement.

---

## Sub-Sources

Sinc Sports also serves as the import pipeline for three smaller platforms that share similar data formats:

| Sub-Source   | Priority | Total Games | Notes                     |
| ------------ | -------- | ----------- | ------------------------- |
| **Coast**    | 6        | 5,934       | Full visibility           |
| **Heartland**| 7        | 1,291       | Full visibility           |
| **Squadi**   | 8        | 84          | Minimal volume            |

These were all imported through the same bulk pipeline and share the Sinc data format conventions.

---

## Dedup Impact

The gap between total (34,961) and visible (29,827) reflects **5,134 games** that were identified as duplicates of higher-priority source records (likely [[GotSport]] API/HTML versions of the same games). See [[Dedup Strategy]] for the cross-source dedup logic.

---

## Related Notes
- [[Data Sources Overview]] — all sources with volume comparison
- [[Dedup Strategy]] — explains the 5K hidden games
- [[Data Pipeline]] — processing chain for imported data
