---
title: Data Quality
aliases: [Completeness, Quality Metrics, Audit Results]
tags: [knowledge, data-quality, audit, completeness, evo-draw]
created: 2026-04-21
updated: 2026-04-21
---

# Data Quality

Current completeness metrics from `audit-data.js` (Step 9 of [[Data Pipeline]]). All percentages are against visible games (~1.64M with `is_hidden=false`).

---

## Field Completeness

| Field            | Completeness | Null Count | Notes                         |
| ---------------- | ------------ | ---------- | ----------------------------- |
| event_name       | **100%**     | 0          | Always populated              |
| game_status      | **100%**     | 0          | Always populated              |
| round            | **100%**     | 0          | Defaults to `group` if unknown|
| source_priority  | **99.9%**    | ~1.6K      | Rare edge cases               |
| season           | **99.0%**    | ~16K       | Some undated games            |
| age_group        | **98.5%**    | ~5,600     | 1.5% null — see gaps below    |
| gender           | **96.7%**    | ~12,000    | 3.3% null — see gaps below    |

---

## Known Gaps

### ~12K Null Gender (3.3%)
- Primarily from [[GotSport]] API data where gender isn't explicit
- `fix-api-gender.js` (Pipeline Step 5) addresses many cases
- Remaining nulls are events with ambiguous naming (e.g., "U14 Division 1" with no gender indicator)
- [[Parsing Rules|Sibling inference]] catches some but not all

### ~5.6K Null Age Group (1.5%)
- Events with non-standard division naming
- Open/adult divisions that slipped past [[Scope Rules]] filters
- Some international events with unfamiliar formats

> [!tip] Improvement Strategy
> Most remaining nulls can be resolved by:
> 1. Expanding [[Parsing Rules]] pattern matching for edge cases
> 2. Manual review of high-volume null-gender events
> 3. Adding event-level gender/age overrides to the pipeline

---

## Scope Constraints

All data is filtered to (see [[Scope Rules]]):
- **Outdoor club youth** soccer only
- **U7-U19** age groups
- **4v4 / 7v7 / 9v9 / 11v11** formats
- **2025+** games only
- US-focused (some international tournament data included)

---

## Data Integrity Checks

The audit script also verifies:
- No orphaned `duplicate_of` references (pointing to non-existent games)
- Score consistency (no negative scores, no unreasonable values)
- Date validity (no future dates beyond reasonable scheduling window)
- Source priority consistency (matches [[Dedup Strategy]] table)

---

## Related Notes
- [[Data Pipeline]] — Step 9 (`audit-data.js`) generates these metrics
- [[Parsing Rules]] — age/gender parsing that drives completeness
- [[Scope Rules]] — what's included vs excluded
- [[Dedup Strategy]] — hidden game filtering
- [[Database Schema]] — column definitions
