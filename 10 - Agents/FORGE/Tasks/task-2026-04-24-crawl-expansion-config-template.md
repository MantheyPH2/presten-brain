---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/GotSport Crawl Config Expansion Template.md"
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/GotSport Crawl Config Expansion Template.md"
---

# Task: GotSport Crawl Config Expansion Template

## Context

Every new org-ID added to Evo Draw requires the same six steps: verify org_id, pre-flight check, update crawl config, test crawl, validate ingestion, update Source Gap Inventory. This process was designed for the TX cohort expansion and will repeat for USL Academy, EDP, Elite 64, and any future GotSport leagues.

Without a template, each expansion requires designing the process from memory or re-reading the full Pipeline Deployment Validation Spec. With a template, adding a confirmed org_id is a fill-in-the-blanks operation — approximately 5 minutes of FORGE time per expansion.

Write the template now, before the USL Academy / EDP browser session, so it is ready to use the moment Presten provides confirmed org IDs.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Infrastructure/GotSport Crawl Config Expansion Template.md`

### Template Header

```
GotSport Org-ID Expansion — [LEAGUE NAME]
Executed by: [FORGE | Presten]
Date: [YYYY-MM-DD]
Org IDs added: [org_id_1, org_id_2, ...]
Source confirmed by: [Presten browser session / FORGE research]
Session type: [config-only | config + test crawl | full expansion]
```

---

### Step 1: Pre-Flight Check

Before adding any org_id, confirm it is not already in the database:

```sql
-- Confirm org_id not already ingesting
SELECT source_org_id, COUNT(*) as existing_games
FROM games
WHERE source_org_id IN ([NEW_ORG_IDS])
GROUP BY source_org_id;
```

Expected: 0 rows (new source). If rows exist: source is already being ingested. Investigate why it appears as a gap in the Source Gap Inventory before adding to config. Do not duplicate.

---

### Step 2: Config File Update

File: `crawl-gotsport-api-parallel.js` (or current crawl config file — confirm filename before editing)

Locate the org_id array. Add new org_id(s) in the appropriate league section:

```javascript
// [LEAGUE_NAME] org_ids — added [YYYY-MM-DD] by [FORGE/Presten]
[ORG_ID_1], // [League Name] — [state/region, e.g., "Northeast"]
[ORG_ID_2], // [League Name] — [state/region]
```

Placement rule: org_ids must be grouped by league. Do not add to an undifferentiated list. If no league section exists, create one with a comment header.

---

### Step 3: Test Crawl (Presten Execution)

Before committing the full org_id list, run a test crawl against ONE org_id from the new set.

Test crawl command: `[fill in from Pipeline Deployment Validation Spec Section 2]`

This is a Presten execution step. FORGE cannot run without execution access. FORGE documents the outcome when Presten reports results.

---

### Step 4: Ingestion Validation

Run after the test crawl completes:

```sql
-- Confirm test crawl wrote games for the new org_id
SELECT source_org_id, COUNT(*) as games_ingested, MAX(created_at) as ingested_at
FROM games
WHERE source_org_id = [TEST_ORG_ID]
  AND created_at > NOW() - INTERVAL '1 hour';
```

Expected: `games_ingested > 0`, `ingested_at` is recent.

```sql
-- Duplicate check on new games
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*)
FROM games
WHERE created_at > NOW() - INTERVAL '1 hour'
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1;
```

Expected: 0 rows. If duplicates: STOP. Do not add remaining org_ids until duplicate source is identified and resolved.

---

### Step 5: Source Gap Inventory Update

In `Infrastructure/Source Gap Inventory — April 2026.md`, update the row for this league:

| Field | New Value |
|-------|-----------|
| Source ID / Config Key | [org_id_1, org_id_2] |
| Last Known Ingestion | [date of test crawl] |
| Status | Active |
| Notes | Added [YYYY-MM-DD]. [N] games confirmed in test crawl. Source confirmed via [browser session / research]. |

If the league was previously listed as Priority 1 or Priority 2 in the gap summary section, update or remove that entry.

---

### Step 6: Completion Record

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

## Definition of Done

- Template written at `Infrastructure/GotSport Crawl Config Expansion Template.md`
- All six steps present in fill-in-the-blanks format
- Pre-flight SQL and duplicate check SQL present and copy-paste ready (with `[PLACEHOLDERS]` clearly marked)
- Config file update format includes the comment-header convention
- Completion record section included
- Summary in briefing: "Crawl config expansion template written. Ready to use when Presten confirms USL Academy / EDP / Elite 64 org IDs."

## Why This Matters

When Presten completes the 15-minute GotSport browser session and confirms org IDs, the next action should be immediate. If FORGE has to design the expansion process in-session, 15 minutes of confirmed org IDs turns into hours of process overhead. This template makes the handoff take 5 minutes, not 2 hours.

## References

- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — validation pattern this generalizes
- `Infrastructure/Source Gap Inventory — April 2026.md` — inventory to update in Step 5
- `10 - Agents/FORGE/Tasks/task-2026-04-24-gotsport-org-id-expansion.md` — TX cohort expansion this template generalizes
- `10 - Agents/FORGE/Tasks/task-2026-04-25-usl-academy-edp-org-id-verification.md` — first expected use of this template
