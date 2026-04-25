---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/GotSport Org-ID Config Edit Reference.md"
---

# Task: GotSport Org-ID Config Edit Reference

## Context

When the browser lookup session runs, Presten will return from `system.gotsport.com` with confirmed org IDs for some or all of: TX USYS state associations (7 states), USL Academy, EDP Soccer, Elite 64, NAL. There is currently no single reference for what to do next. Adding each org ID involves editing the crawl config — but Presten needs to know the file path, the exact JSON/YAML structure, any per-entity flags, and how to verify the edit worked.

This document is the "what to do after the browser session" counterpart to FORGE's Browser Lookup Session Prep Package.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/GotSport Org-ID Config Edit Reference.md`

Sections:

### 1. Config File Location and Structure

- Exact file path(s) for the crawl config (e.g., `crawlers/gotsport/config.js` or wherever org IDs are defined)
- Show one existing entry as an example with all fields annotated
- Clarify whether the file is JSON, YAML, or JS — and whether changes require a restart/redeploy

### 2. Entry Template by Org Type

For each org type, provide a copy-paste template block:

| Org Type | Notes |
|----------|-------|
| USYS state association (e.g., TX Soccer) | State org — likely indexes all teams and seasons in that state |
| League entity (USL Academy, EDP, Elite 64) | League org — may require per-season filtering |
| Unknown platform (NAL) | Deferred until platform confirmed |

Each template should have placeholders clearly marked (e.g., `"org_id": "REPLACE_WITH_VALUE"`).

### 3. Adding an Entry — Step by Step

5 steps: open config, add entry, save, verify syntax, confirm no duplicate org_ids. Include syntax validation command if one exists.

### 4. Post-Edit Verification Query

SQL query Presten can run 24 hours after adding a new org ID to confirm games are being ingested:

```sql
SELECT DATE(created_at) AS ingest_date, COUNT(*) AS game_count
FROM games
WHERE source = 'gotsport'
  AND org_id = '[NEW_ORG_ID]'
ORDER BY ingest_date DESC
LIMIT 5;
```

PASS: new rows appearing with `ingest_date` = today or yesterday.
FAIL: zero rows after 48 hours → check crawl log for that org.

### 5. Deduplication Risk

Brief note: if a new org ID's teams are already partially in the DB (ingested via a different path), dedup behavior applies. Flag to FORGE if game count after first crawl exceeds 5,000 or if any team appears with duplicate records.

## Quality Standard

Presten should be able to add a new org ID in under 5 minutes using this document alone. Every code block must be copy-paste ready with clear placeholder markers. Do not leave any step implicit.

## Notes

- Read the actual crawl config file to ensure the template reflects real structure. Do not guess field names.
- If there are different config structures for different org types (state vs. league vs. club), document each separately.
