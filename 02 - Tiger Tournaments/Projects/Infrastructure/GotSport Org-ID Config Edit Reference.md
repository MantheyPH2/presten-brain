---
title: GotSport Org-ID Config Edit Reference
tags: [infrastructure, gotsport, org-id, config, pipeline, evo-draw]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: ready — use after browser lookup session returns results
companion: "Infrastructure/GotSport Browser Lookup Session Prep Package.md"
---

# GotSport Org-ID Config Edit Reference

> Use this document immediately after the browser lookup session returns confirmed org IDs. It is the "what to do next" counterpart to the [[GotSport Browser Lookup Session Prep Package]].

---

## Section 1: Config File Location and Structure

**File:** `crawl-gotsport-api-parallel.js`  
**Location:** Project root (same directory as `run-daily.sh` and `run-overnight.sh`)  
**Language:** JavaScript  
**Restart required after edits:** No restart needed for org_id changes alone — the crawl script reads the config at runtime. However, if a scheduled cron run is in progress when you edit, the change will take effect on the next run.

**How org IDs are stored:**

The file contains a JavaScript array of numeric org IDs, grouped by league with comment headers. Example of an existing entry:

```javascript
// TX Soccer (USYS Texas) — added 2026-04-XX by Presten
12345, // TX Soccer — Texas statewide
12346, // TX Club Soccer — TX competitive clubs
```

Each entry is a bare integer followed by an inline comment identifying the league, region, and date added. The comment is for human reference — it is not parsed by the crawler.

> [!warning] Verify file name before editing
> The filename `crawl-gotsport-api-parallel.js` is documented in the GotSport Crawl Config Expansion Template (April 24). Confirm this is the actual filename before editing: `ls *.js | grep crawl-gotsport` from the project root.

---

## Section 2: Entry Template by Org Type

### USYS State Association

State associations index all affiliated teams and competitions within that state. A single org_id captures all USYS state league and state cup divisions for that association.

```javascript
// [STATE NAME] Soccer (USYS [State]) — added [YYYY-MM-DD] by Presten
REPLACE_WITH_ORG_ID, // [State Name] — [state abbreviation] statewide
```

**Example:**
```javascript
// California Soccer Association (USYS CA) — added 2026-05-01 by Presten
99999, // California Soccer Association — CA statewide
```

### League Entity (USL Academy, EDP Soccer, Elite 64)

League-level organizations span multiple states and may require per-season filtering downstream. Add the single org_id — do not attempt to add per-state sub-orgs unless Presten confirms they are separate GotSport entities.

```javascript
// [League Name] — added [YYYY-MM-DD] by Presten
REPLACE_WITH_ORG_ID, // [League Name] — national
```

**Example:**
```javascript
// USL Academy — added 2026-05-01 by Presten
88888, // USL Academy — national
```

### Unknown Platform (NAL)

NAL platform is unconfirmed. Do not add a NAL entry until the browser lookup session returns a confirmed GotSport org_id. If NAL is not on GotSport, a different ingestion path is required — see Source Gap Inventory for next action.

---

## Section 3: Adding an Entry — Step by Step

1. **Pre-flight:** Run the duplicate check query (Section 4A) to confirm the org_id is not already ingesting. Zero rows = safe to add.

2. **Open the config file:**
   ```bash
   nano crawl-gotsport-api-parallel.js
   ```
   Or use your preferred editor. Do not use a GUI editor that may introduce encoding issues.

3. **Locate the correct league section.** Org IDs must be grouped by league. If no section exists for the league, create one with a comment header above the entry:
   ```javascript
   // [LEAGUE NAME] — added [YYYY-MM-DD] by Presten
   ```

4. **Add the entry.** Use the template from Section 2 for the appropriate org type. Replace `REPLACE_WITH_ORG_ID` with the confirmed numeric ID. Add the inline comment.

5. **Verify syntax:**
   ```bash
   node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"
   ```
   Expected: `Syntax OK`. If a syntax error appears: fix before proceeding — a syntax error in this file will cause all crawl runs to fail.

6. **Check for duplicate org_ids in the file** (same ID added twice will double-crawl):
   ```bash
   node -e "
   const cfg = require('./crawl-gotsport-api-parallel.js');
   const ids = Array.isArray(cfg) ? cfg : cfg.orgIds || cfg.org_ids || [];
   const dupes = ids.filter((id, i) => ids.indexOf(id) !== i);
   console.log(dupes.length > 0 ? 'DUPLICATE IDs: ' + dupes : 'No duplicates');
   "
   ```
   Expected: `No duplicates`. If duplicates appear: remove the redundant entry before proceeding.

   > [!note] If the above command errors
   > The script exports format may differ. Manually grep for the new org_id: `grep -n "REPLACE_WITH_VALUE" crawl-gotsport-api-parallel.js` — it should appear exactly once.

---

## Section 4: Verification Queries

### 4A: Pre-flight — Confirm Org Not Already Ingesting

Run **before** adding the org_id to the config:

```sql
SELECT source_org_id, COUNT(*) AS existing_games
FROM games
WHERE source_org_id = 'REPLACE_WITH_NEW_ORG_ID'
GROUP BY source_org_id;
```

**Expected:** 0 rows (new source).  
**If rows exist:** The org_id is already in the pipeline via a different path. Investigate the existing ingestion before adding to config — do not create a duplicate crawl path.

### 4B: Post-crawl — Confirm Games Are Ingesting

Run 24–48 hours after the next scheduled crawl (or immediately after a manual test run):

```sql
SELECT DATE(created_at) AS ingest_date, COUNT(*) AS game_count
FROM games
WHERE source = 'gotsport'
  AND source_org_id = 'REPLACE_WITH_NEW_ORG_ID'
ORDER BY ingest_date DESC
LIMIT 5;
```

**PASS:** Rows appear with `ingest_date` = today or yesterday.  
**FAIL:** Zero rows after 48 hours → check crawl log for errors related to this org_id. File a queue item for FORGE with the org_id and crawl log output.

---

## Section 5: Deduplication Risk

If a new org_id covers a league whose teams were already partially ingested via a different GotSport org, the dedup pipeline will handle it automatically using `is_hidden` / `duplicate_of`. However:

- If game count after first crawl for the new org_id exceeds **5,000**, flag to FORGE — high volume on first ingest may indicate broad overlap with existing data, and FORGE should verify the dedup rate before the next ranking recompute.
- If any team appears with duplicate records (same game, two visible rows), stop and notify FORGE with the team ID and event name. Do not run ranking recompute until FORGE clears the duplicate.

---

## Section 6: Source Gap Inventory Update

After a successful post-crawl verification (Section 4B passes):

In `Infrastructure/Source Gap Inventory — April 2026.md`, update the row for the newly added league:

| Field | New Value |
|-------|-----------|
| Source ID / Config Key | `[org_id]` |
| Last Known Ingestion | `[date of first successful crawl]` |
| Status | Active |
| Notes | `Added [YYYY-MM-DD]. [N] games confirmed in first crawl. Org_id confirmed via browser session [date].` |

If the league was listed as Priority 1 or Priority 2 in the gap summary, update or remove that entry.

---

## Planned Additions (Post Browser Lookup Session)

| Entity | Org ID Status | Config Section to Add Under |
|--------|--------------|----------------------------|
| USYS CA (California Soccer Association) | Pending browser lookup | `// USYS State Associations` |
| USYS FL (Florida Youth Soccer) | Pending browser lookup | `// USYS State Associations` |
| USYS NY (New York Soccer Association) | Pending browser lookup | `// USYS State Associations` |
| USYS NJ (New Jersey Youth Soccer) | Pending browser lookup | `// USYS State Associations` |
| USYS WA (Washington Youth Soccer) | Pending browser lookup | `// USYS State Associations` |
| USYS PA (Pennsylvania Soccer Association) | Pending browser lookup | `// USYS State Associations` |
| USYS OH (Ohio South Soccer / Ohio North Soccer) | Pending browser lookup | `// USYS State Associations` |
| USL Academy | Pending browser lookup | `// USL Academy` |
| EDP Soccer | Pending browser lookup | `// EDP Soccer` |
| Elite 64 | Pending browser lookup — conditional GO | `// Elite 64` |
| NAL | Platform unconfirmed — deferred | N/A until platform confirmed |

---

## References

- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — pre-session companion doc
- `Infrastructure/GotSport Crawl Config Expansion Template.md` — per-expansion record template (fill in after each addition)
- `Infrastructure/Source Gap Inventory — April 2026.md` — update Section 6 after each confirmed ingestion
- `Infrastructure/Source Health Monitoring Queries.md` — Section 4: new source coverage confirmation query

*FORGE — 2026-04-25*
