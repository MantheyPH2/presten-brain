---
title: Scope Rules
aliases: [Scope, Inclusion Criteria, HARD_SKIP, Exclusions]
tags: [knowledge, scope, filtering, hard-skip, evo-draw]
created: 2026-04-21
updated: 2026-04-21
---

# Scope Rules

Evo Draw focuses exclusively on **outdoor club youth soccer** for ages U7-U19. These rules govern what data is included and excluded across all [[Data Sources Overview|data sources]] and the [[Data Pipeline]].

---

## Include

| Criterion     | Values                              |
| ------------- | ----------------------------------- |
| Sport         | Outdoor soccer only                 |
| Type          | Club/competitive youth              |
| Age groups    | U7 through U19                      |
| Formats       | 4v4, 7v7, 9v9, 11v11               |
| Time range    | 2025 onwards                        |
| Geography     | Primarily US (international tournament games included) |

See [[Age Groups and Birth Years]] for the format-to-age mapping.

---

## Exclude

| Category        | Excluded Items                                  |
| --------------- | ----------------------------------------------- |
| Surface         | Beach soccer, futsal, indoor                    |
| Level           | High school, adult, recreational-only leagues   |
| Format          | 3v3, 5v5, 6v6                                  |
| Activity type   | Camps, clinics, tryouts, training sessions      |
| Age             | U6 and below, U20 and above                    |

---

## HARD_SKIP Keywords

The [[GotSport HTML Scraper]] immediately skips events containing these keywords in the event name. This is the first filter layer (Layer 1 of 3).

> [!warning] Case-Insensitive Matching
> All keywords are matched case-insensitively against event names.

**Surface/Format exclusions:**
- `beach`, `futsal`, `indoor`, `arena`

**Non-competitive exclusions:**
- `camp`, `clinic`, `tryout`, `training`, `practice`

**Non-youth exclusions:**
- `adult`, `over 30`, `over 40`, `over 50`, `coed adult`, `senior`
- `high school`, `hs varsity`, `hs jv`

**Format exclusions:**
- `3v3`, `3 v 3`, `5v5`, `5 v 5`, `6v6`, `6 v 6`

**Other exclusions:**
- `recreational` (as a standalone league, not division name)
- `walking soccer`, `parasoccer`, `special olympics`

> [!info] Division vs Event Filtering
> "Recreational" as a **division name** within a competitive tournament (e.g., [[Dallas Spring Showcase]] Recreational division) is NOT excluded. Only events that are entirely recreational leagues are filtered. The HARD_SKIP applies to event names only, not division names.

---

## Age Verification (Layer 3)

After passing the HARD_SKIP and content scan filters, events must contain age groups within the U7-U19 range:
- Events with only U6 or below → skipped
- Events with only U20+ → skipped
- Events with a mix (e.g., U5-U19) → included, but games outside U7-U19 are filtered during [[Data Pipeline|backfill]]

---

## Related Notes
- [[GotSport HTML Scraper]] — 3-layer filter implementation
- [[Data Quality]] — completeness within scope
- [[Age Groups and Birth Years]] — U7-U19 definitions
- [[Parsing Rules]] — how age groups are extracted
- [[Data Pipeline]] — where scope enforcement happens
