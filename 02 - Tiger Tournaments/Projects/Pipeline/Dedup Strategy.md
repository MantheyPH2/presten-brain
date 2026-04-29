---
title: Dedup Strategy
aliases: [Deduplication, Dedup, Source Priority, is_hidden]
tags: [pipeline, dedup, deduplication, source-priority, evo-draw]
created: 2026-04-21
updated: 2026-04-21
---

# Dedup Strategy

Deduplication is Step 6 of the [[Data Pipeline]] (`dedup-and-link-teams.js`). It identifies and resolves duplicate game records both within and across data sources.

---

## Source Priority Table

Lower number = higher priority = kept as canonical.

| Priority | Source          | Notes                        |
| -------- | -------------- | ---------------------------- |
| 1        | [[GotSport]] API  | Richest structured data   |
| 2        | [[GotSport]] HTML | Fallback, less structured |
| 3        | [[TGS AthleteOne]] | ECNL primary             |
| 4        | [[MLS NEXT Source]] | Tagged subset            |
| 5        | [[Sinc Sports]]    | Bulk imported            |
| 6        | Coast              | Via Sinc pipeline        |
| 7        | Heartland          | Via Sinc pipeline        |
| 8        | Squadi             | Minimal volume           |

---

## Cross-Source Dedup

When the same game appears in multiple sources:
1. Match on: teams + date + score (fuzzy)
2. Keep the **highest priority** version (lowest priority number)
3. Mark lower-priority versions: `is_hidden = true`
4. Set `duplicate_of` to point to the canonical game's ID

**Example:** A game appears in both GotSport API (pri 1) and MLS NEXT (pri 4):
- GotSport API version → `is_hidden = false` (canonical)
- MLS NEXT version → `is_hidden = true`, `duplicate_of = <gotsport_game_id>`

---

## Within-Source Dedup

Exact duplicates from the same source:
- Same `(source, source_id)` → upsert keeps latest version
- Same game with different source_ids → compare data completeness, keep best version

---

## Team Merges

The `team_merges` table links the same real-world team across different platforms:

| Field            | Description                                |
| ---------------- | ------------------------------------------ |
| Total entries    | **27,820**                                 |
| Verified         | 18,684 (used by [[Ranking Engine]])        |
| Unverified       | 9,136 (33% — under active review)          |
| Linking method   | Coach names, fuzzy match, manual review    |
| Table            | `team_merges` in [[Database Schema]]       |

Team merges are critical for the [[Ranking Engine]] — without them, the same team would have separate ratings on each platform.

**Verification status (2026-04-29):** 9,136 unverified entries. FORGE initiated triage 2026-04-29:
- Safe bulk-verify criteria defined (name variants, same club/age/gender, zero-game entries)
- Risk flags identified: cross-gender merges (delete), cross-age-group merges (review), asymmetric merges (spot-check)
- Queue item posted: `10 - Agents/FORGE/Queue/pending-2026-04-29-team-merges-ambiguous.md`
- **Blocked on:** Presten running flag queries to identify count in each risk category before bulk-verify executes

---

## Key Database Columns

All in the `games` table (see [[Database Schema]]):

| Column         | Purpose                                           |
| -------------- | ------------------------------------------------- |
| `source`       | Which data source (gotsport_api, gotsport, tgs, etc.) |
| `source_id`    | Unique ID within that source                      |
| `source_priority` | Integer priority (1=highest)                   |
| `is_hidden`    | `true` if this is a duplicate                     |
| `duplicate_of` | Points to canonical game version                  |

> [!tip] Site Query Pattern
> All user-facing queries MUST include `WHERE is_hidden = false`. This is the universal filter that ensures only canonical game versions are shown. The [[Server and Frontend|server API]] enforces this.

---

## Impact by Source

| Source          | Total    | Visible  | Hidden | % Hidden |
| --------------- | -------- | -------- | ------ | -------- |
| [[GotSport]]   | 1,489,861| 1,487,049| 2,812  | 0.2%     |
| [[TGS AthleteOne]] | 91,401 | 91,345 | 56     | 0.06%    |
| [[Sinc Sports]] | 34,961  | 29,827   | 5,134  | 14.7%    |
| [[MLS NEXT Source]] | 27,635 | 21,580 | 6,055  | 21.9%    |

> [!info] MLS NEXT High Dedup Rate
> MLS NEXT has the highest dedup rate (21.9%) because many MLS NEXT games also appear in [[GotSport]] data. Since GotSport has higher priority (1-2 vs 4), the GotSport versions are kept and MLS NEXT versions are hidden.

---

## Related Notes
- [[Data Pipeline]] — Step 6 in the processing chain
- [[Data Sources Overview]] — volume by source
- [[Database Schema]] — `is_hidden`, `duplicate_of`, `team_merges` columns
- [[Ranking Engine]] — Phase 1 loads team merges
- [[Team Name Matching]] — name-based matching challenges
