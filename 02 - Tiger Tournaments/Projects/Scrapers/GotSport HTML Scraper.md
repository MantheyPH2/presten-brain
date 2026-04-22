---
title: GotSport HTML Scraper
aliases: [crawl-gotsport-v2.js, GS HTML Scraper]
tags: [scraper, gotsport, html, parsing, evo-draw]
created: 2026-04-21
updated: 2026-04-21
file: crawl-gotsport-v2.js
source_priority: 2
---

# GotSport HTML Scraper

An event-centric scraper that fetches **server-rendered HTML** from `system.gotsport.com` and parses schedule tables. Complements the [[GotSport API Scraper]] by catching events/games that may not appear via the team-based API approach.

---

## Technical Details

| Attribute       | Value                                  |
| --------------- | -------------------------------------- |
| File            | `crawl-gotsport-v2.js`                |
| Method          | Server-rendered HTML parsing           |
| Source ID       | `gs_{eventId}_{matchNum}`              |
| Source priority | 2 (see [[Dedup Strategy]])             |

---

## 3-Layer Filter

The scraper applies three sequential filters to avoid ingesting non-youth or irrelevant data:

### Layer 1: HARD_SKIP Keywords
Events with these keywords in the name are immediately skipped. See [[Scope Rules]] for the full keyword list (beach, futsal, indoor, adult, etc.).

### Layer 2: Page Content Scan
After fetching the event page, scan content for indicators of non-youth soccer (adult leagues, indoor facilities, etc.).

### Layer 3: Youth Age Verification
Verify that the event contains age groups within scope: **U7-U19 only**. Events outside this range are skipped.

---

## Score Parsing

| Format                    | Example              | Parsed               |
| ------------------------- | -------------------- | --------------------- |
| Normal                    | `3 - 1`              | Home: 3, Away: 1     |
| Penalties (PKS)           | `0 - 0PKS: 4 - 3`   | Home: 0, Away: 0 (PKS: 4-3) |
| In progress               | `1 - 0*`             | Home: 1, Away: 0     |

> [!info] Penalty Handling
> Penalty shootout results (`PKS`) are stored separately. The official score remains the regulation score (typically 0-0 or tied). The PKS result determines the winner for bracket advancement purposes.

---

## Test Event

| Event          | GotSport ID | Games Scraped |
| -------------- | ----------- | ------------- |
| Dallas Cup 2026| 45488       | 889           |

Dallas Cup 2026 (event 45488) with 889 games serves as the reference test case for the HTML scraper.

---

## Related Notes
- [[GotSport]] — source overview
- [[GotSport API Scraper]] — primary scraper (priority 1)
- [[Dedup Strategy]] — API vs HTML duplicate resolution
- [[Parsing Rules]] — score and age/gender parsing details
- [[Scope Rules]] — HARD_SKIP keywords and age filtering
