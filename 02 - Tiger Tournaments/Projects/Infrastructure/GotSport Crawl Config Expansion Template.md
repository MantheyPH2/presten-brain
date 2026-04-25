---
type: template
author: FORGE
date: 2026-04-24
status: ready
use_when: Adding a new GotSport org_id to the crawl config
---

# GotSport Crawl Config Expansion Template

Fill in this template whenever a new confirmed org_id is added to the Evo Draw crawl config. Copy the header block into a new session note if needed.

---

## Expansion Header (fill in per session)

```
GotSport Org-ID Expansion — [LEAGUE NAME]
Executed by: [FORGE | Presten]
Date: [YYYY-MM-DD]
Org IDs added: [org_id_1, org_id_2, ...]
Source confirmed by: [Presten browser session / FORGE research]
Session type: [config-only | config + test crawl | full expansion]
```

---

## Step 1: Pre-Flight Check

Before adding any org_id, confirm it is not already in the database:

```sql
-- Confirm org_id not already ingesting
SELECT source_org_id, COUNT(*) AS existing_games
FROM games
WHERE source_org_id IN ([NEW_ORG_IDS])
GROUP BY source_org_id;
```

**Expected:** 0 rows (new source).
**If rows exist:** Source is already being ingested. Investigate why it appears as a gap in the Source Gap Inventory before adding to config. Do not duplicate.

---

## Step 2: Config File Update

**File:** `crawl-gotsport-api-parallel.js` (confirm filename before editing — this may have been renamed)

Locate the org_id array. Add new org_id(s) in the appropriate league section:

```javascript
// [LEAGUE_NAME] org_ids — added [YYYY-MM-DD] by [FORGE/Presten]
[ORG_ID_1], // [League Name] — [state/region, e.g., "Northeast"]
[ORG_ID_2], // [League Name] — [state/region]
```

**Placement rule:** Org IDs must be grouped by league. Do not add to an undifferentiated list. If no league section exists, create one with a comment header.

---

## Step 3: Test Crawl (Presten Execution)

Before committing the full org_id list, run a test crawl against ONE org_id from the new set.

**Test crawl command:** See `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` Section 2 for the exact command.

This is a Presten execution step. FORGE cannot run without execution access. FORGE documents the outcome when Presten reports results.

---

## Step 4: Ingestion Validation

Run after the test crawl completes:

```sql
-- Confirm test crawl wrote games for the new org_id
SELECT source_org_id, COUNT(*) AS games_ingested, MAX(created_at) AS ingested_at
FROM games
WHERE source_org_id = [TEST_ORG_ID]
  AND created_at > NOW() - INTERVAL '1 hour';
```

**Expected:** `games_ingested > 0`, `ingested_at` is recent (within the last hour).

```sql
-- Duplicate check on new games
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*)
FROM games
WHERE created_at > NOW() - INTERVAL '1 hour'
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1;
```

**Expected:** 0 rows.
**If duplicates:** STOP. Do not add remaining org_ids until duplicate source is identified and resolved.

---

## Step 5: Source Gap Inventory Update

In `Infrastructure/Source Gap Inventory — April 2026.md`, update the row for this league:

| Field | New Value |
|-------|-----------|
| Source ID / Config Key | [org_id_1, org_id_2] |
| Last Known Ingestion | [date of test crawl] |
| Status | Active |
| Notes | Added [YYYY-MM-DD]. [N] games confirmed in test crawl. Source confirmed via [browser session / research]. |

If the league was previously listed as Priority 1 or Priority 2 in the gap summary section, update or remove that entry.

---

## Step 6: Completion Record

```
Expansion complete: [YYYY-MM-DD]
League: [name]
Org IDs added: [list]
Games confirmed in test crawl: [N]
Duplicate check: PASS / FAIL
Source Gap Inventory updated: YES / NO
SENTINEL notified in briefing: YES / NO
Next expansion (if applicable): [league name, pending org_id confirmation]
```

---

## Planned Upcoming Expansions

| League | Org ID Status | Next Action |
|--------|--------------|-------------|
| USL Academy | Pending browser lookup | Presten confirms at system.gotsport.com |
| EDP | Pending browser lookup | Presten confirms at system.gotsport.com |
| Elite 64 | Pending browser lookup | Presten confirms at system.gotsport.com |
| NAL | Platform unknown | NAL website identification first |
| Additional USYS state orgs | IDs documented in `Data Sources/GotSport USYS Org-ID Master List.md` | After TX cohort test passes |

---

## References

- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — validation pattern this generalizes
- `Infrastructure/Source Gap Inventory — April 2026.md` — inventory to update in Step 5
- `Infrastructure/Source Health Monitoring Queries.md` — Section 4: new source coverage confirmation query
- `10 - Agents/FORGE/Tasks/task-2026-04-24-gotsport-org-id-expansion.md` — TX cohort expansion this template generalizes
