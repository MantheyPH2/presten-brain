---
title: Source Submission MVP — Intake Form Technical Spec
tags: [infrastructure, pipeline, source-submission, mvp, spec, evo-draw, forge]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: spec-complete — ready for MVP session execution
task: task-2026-04-27-source-submission-mvp-intake-form-spec
---

# Source Submission MVP — Intake Form Technical Spec

> Operationalizes the Source Submission MVP Session Plan (`Infrastructure/Source Submission MVP — Session Plan.md`). Write this spec before the build session — the MVP session is a fill-in-the-spec build, not an improvisational coding session.

---

## Section 1 — Source Entry Data Model

Each source submission must contain the following fields. Field names use snake_case as they will appear in config and DB.

| Field | Type | Required | Valid Values / Format | Purpose |
|-------|------|----------|----------------------|---------|
| `source_id` | string | **Required** | Unique alphanumeric slug, e.g. `gotsport_wysa`, `sincsports_ohsaa` | Primary key for the source entry; used in logs and dedup lookups |
| `source_type` | enum | **Required** | `gotsport_org`, `sincsports`, `affinitysoccer`, `manual_csv`, `tgs`, `tgs_athleteone` | Determines which crawl module handles this source. Must match an existing classifier enum. |
| `org_id` | integer | Required if `source_type = gotsport_org` | Numeric integer only, e.g. `12345` | GotSport-specific: the numeric org ID in `crawl-gotsport-api-parallel.js`. Not used for non-GotSport source types. |
| `source_url` | string | Required if `source_type ≠ gotsport_org` | Valid URL string, e.g. `https://sincsports.com/results?org=OHSAA` | Non-GotSport sources: the base URL or endpoint the crawler fetches. |
| `league_hint` | string | Optional | Free text, e.g. `NPL Southeast`, `OYSA State Cup` | Downstream event classifier uses this to assign `event_tier` when the source doesn't self-identify the league. Leave blank if the source type already provides reliable tier data. |
| `active` | boolean | **Required** | `true` / `false` | Whether the pipeline should crawl this source on the next run. New submissions should default to `true`. |
| `last_verified` | date | **Required** | ISO date string, `YYYY-MM-DD` | Date FORGE last confirmed the source is live (org exists on GotSport, URL is reachable). Prevents stale entries from silently failing. |
| `stage` | enum | **Required** | `stage1`, `stage2`, `stage3` | Expansion stage this source belongs to. Used for audit trail and SENTINEL authorization tracking. |
| `authorized_by` | string | **Required** | Name, e.g. `SENTINEL`, `Presten` | Who authorized this source addition. SENTINEL authorization is required for Stage 2+. |
| `date_added` | date | **Required** | ISO date string, auto-populated at submission time | Audit trail. |
| `notes` | string | Optional | Free text | Unusual config notes, access limitations, org naming edge cases, dedup flags. |

**Justification for additions beyond the minimum spec:**

- `stage`: Required for SENTINEL audit. Stage 2+ entries must be authorized — tracking stage in the source record makes authorization audit trivial.
- `authorized_by`: Required for SENTINEL authorization gate compliance. No Stage 2+ source should be in config without an authorization record.
- `date_added`: Auto-populated. Required for `Source Gap Inventory` update and `GotSport Org-ID Master Reference` Section 1 "Date Added" field.

---

## Section 2 — Validation Rules

Validation fires when a new source entry is submitted. All Required fields must pass before the entry is written to config.

| Field | Validation | Failure message |
|-------|-----------|----------------|
| `source_id` | Non-empty string, alphanumeric + underscores only, max 64 chars | "source_id must be a non-empty alphanumeric slug (underscores allowed, max 64 chars)" |
| `source_type` | Must match enum: `gotsport_org`, `sincsports`, `affinitysoccer`, `manual_csv`, `tgs`, `tgs_athleteone` | "Unknown source_type. Valid values: gotsport_org, sincsports, affinitysoccer, manual_csv, tgs, tgs_athleteone" |
| `org_id` | If `source_type = gotsport_org`: must be a positive integer | "org_id must be a positive integer for GotSport sources" |
| `source_url` | If `source_type ≠ gotsport_org`: must be a valid URL (starts with `https://`) | "source_url must be a valid HTTPS URL for non-GotSport sources" |
| `active` | Must be boolean `true` or `false` | "active must be true or false" |
| `last_verified` | Must be ISO date string in YYYY-MM-DD format, ≤ today's date | "last_verified must be a valid date (YYYY-MM-DD) and not in the future" |
| `stage` | Must be `stage1`, `stage2`, or `stage3` | "stage must be one of: stage1, stage2, stage3" |
| `authorized_by` | Non-empty string | "authorized_by must name the authorizing party (e.g. SENTINEL, Presten)" |

**Two global validations that fire for every submission regardless of field values:**

1. **Duplicate check:** Reject if `source_id` already exists in the source registry, OR if `org_id` already exists in `crawl-gotsport-api-parallel.js` (for GotSport sources).

   ```javascript
   // Duplicate source_id check
   if (existingSourceIds.includes(submission.source_id)) {
     throw new Error(`Duplicate source_id: '${submission.source_id}' already exists in config`);
   }
   // Duplicate org_id check (GotSport only)
   if (submission.source_type === 'gotsport_org' && existingOrgIds.includes(submission.org_id)) {
     throw new Error(`Duplicate org_id: ${submission.org_id} already in crawl-gotsport-api-parallel.js`);
   }
   ```

2. **Reachability check (optional but preferred):** For non-GotSport sources, attempt an HTTP HEAD request to `source_url`. If response is non-2xx, issue a warning (do not block submission):

   ```javascript
   // Warning-only — do not block on reachability failure
   const res = await fetch(submission.source_url, { method: 'HEAD' });
   if (!res.ok) {
     console.warn(`WARNING: ${submission.source_url} returned ${res.status}. Confirm source is live before first crawl run.`);
   }
   ```

---

## Section 3 — Intake → Pipeline Flow

How a validated source entry moves from the intake form to active crawling:

**Step 1 — FORGE submits source entry**

FORGE calls the intake function with the source entry object (JavaScript object or JSON). The intake form is implemented as a CLI script or Node.js function in the project root:

```bash
node add-source.js --source-id gotsport_wysa --source-type gotsport_org --org-id 99999 --stage stage2 --authorized-by SENTINEL --last-verified 2026-05-01
```

Or via JSON config file:
```bash
node add-source.js --config new-source.json
```

**Step 2 — Validation fires inline**

All field validations and duplicate checks run before any file write. If any validation fails, the process exits with a non-zero code and prints the failure message. No partial writes.

**Step 3 — Entry written to config**

For `gotsport_org` sources: the numeric `org_id` is appended to `crawl-gotsport-api-parallel.js` in the appropriate section (grouped by league/region, with a comment header). Format:

```javascript
// [League/Association Name] — added [DATE] by [authorized_by]
[ORG_ID], // [Entity name] — [region/state]
```

For non-GotSport sources: a new entry is appended to `source-registry.json` (create this file as part of the MVP build):

```json
{
  "source_id": "sincsports_ohsaa",
  "source_type": "sincsports",
  "source_url": "https://sincsports.com/results?org=OHSAA",
  "league_hint": "Ohio Soccer",
  "active": true,
  "last_verified": "2026-05-01",
  "stage": "stage2",
  "authorized_by": "SENTINEL",
  "date_added": "2026-05-01",
  "notes": ""
}
```

**Step 4 — Pipeline picks up on next run**

No pipeline restart required. `crawl-gotsport-api-parallel.js` reads the org_id array at runtime — the next scheduled cron run will include the new org_id. `source-registry.json` is also read at startup.

If a cron run is in progress when the entry is added: the change takes effect on the **next** run, not the current one. This is expected behavior.

**Step 5 — FORGE confirms ingestion**

Run 24–48 hours after the next pipeline run (per `Infrastructure/GotSport Org-ID Config Edit Reference.md` Section 4B):

```sql
-- For GotSport sources:
SELECT DATE(created_at) AS ingest_date, COUNT(*) AS game_count
FROM games
WHERE source = 'gotsport'
  AND source_org_id = 'NEW_ORG_ID'
ORDER BY ingest_date DESC
LIMIT 5;

-- For non-GotSport sources:
SELECT DATE(created_at) AS ingest_date, COUNT(*) AS game_count
FROM games
WHERE source = 'NEW_SOURCE_ID'
ORDER BY ingest_date DESC
LIMIT 5;
```

PASS: rows appear with `ingest_date` = today or yesterday. FAIL: zero rows after 48 hours → file queue item for FORGE with source_id and crawl log output.

---

## Section 4 — Session Execution Plan

Build plan for the MVP session. Target: ≤4 hours total.

| Step | What to Build | Time Estimate | Definition of Done |
|------|--------------|---------------|-------------------|
| 1 | Create `add-source.js` scaffold: CLI arg parsing, source entry object construction | 30 min | Script accepts `--source-id`, `--source-type`, `--org-id`, `--stage`, `--authorized-by`, `--last-verified` flags without error |
| 2 | Implement field validation (Section 2, rows 1–8) | 30 min | Each validation throws correct error message on invalid input; all validations pass for a valid test entry |
| 3 | Implement duplicate checks (Section 2, global validations) | 30 min | Duplicate `source_id` rejects; duplicate `org_id` rejects; new unique entry passes |
| 4 | Implement write to `crawl-gotsport-api-parallel.js` (GotSport path) | 45 min | Running `node add-source.js` with a valid GotSport entry appends the org_id with correct comment format; syntax check passes |
| 5 | Create `source-registry.json` and implement write path (non-GotSport path) | 30 min | Running `node add-source.js` with a valid SincSports entry writes a valid JSON entry to `source-registry.json` |
| 6 | Implement optional reachability check for non-GotSport sources | 20 min | Non-2xx response prints warning; 2xx response is silent; does not block submission on failure |
| 7 | Testing protocol (Section 5) | 30 min | All 5 test steps pass; test entry removed cleanly |
| 8 | Update `GotSport Org-ID Config Edit Reference.md` Section 3 to reference `add-source.js` workflow | 15 min | Config Edit Reference points to `add-source.js` as the new step 2 for GotSport additions |

**Total estimated time:** 3.5 hours (under 4-hour target).

**Phase alignment with Session Plan (Apr 26):**
- Phase 1 (data model + validation) = Steps 1–3 above
- Phase 2 (write path) = Steps 4–5
- Phase 3 (optional reachability + testing) = Steps 6–7
- Phase 4 (documentation update) = Step 8

---

## Section 5 — Testing Protocol

Before declaring the MVP complete, run all 5 steps using a known-good GotSport org_id already in production as the test subject.

**Test source entry (example — use a real confirmed org_id from `crawl-gotsport-api-parallel.js`):**
```json
{
  "source_id": "test_gotsport_temp",
  "source_type": "gotsport_org",
  "org_id": [USE_EXISTING_ORG_ID_FROM_CONFIG],
  "active": true,
  "last_verified": "2026-04-27",
  "stage": "stage1",
  "authorized_by": "FORGE-test",
  "notes": "TEST ENTRY — remove after validation"
}
```

| Test Step | Command | Expected Result |
|-----------|---------|----------------|
| 1. Submit test entry | `node add-source.js --source-id test_gotsport_temp --source-type gotsport_org --org-id [EXISTING_ORG_ID] --stage stage1 --authorized-by FORGE-test --last-verified 2026-04-27` | **Should fail** with "Duplicate org_id" — confirms duplicate check works correctly |
| 2. Submit with unique test org_id | Use a dummy org_id not in config (e.g., `9999999`): `node add-source.js --source-id test_gotsport_temp --source-type gotsport_org --org-id 9999999 --stage stage1 --authorized-by FORGE-test --last-verified 2026-04-27` | **Should succeed** — entry appended to config with correct comment format |
| 3. Validate appears in config | `grep -n "test_gotsport_temp\|9999999" crawl-gotsport-api-parallel.js` | Entry appears exactly once |
| 4. Syntax check | `node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"` | `Syntax OK` — no syntax errors introduced |
| 5. Remove test entry | Manually delete the `9999999` line from `crawl-gotsport-api-parallel.js`; re-run syntax check | `Syntax OK` after removal; `grep` returns 0 results for `9999999` |

**MVP complete when:** All 5 test steps pass and test entry is cleanly removed. File confirmation in next FORGE briefing.

---

## References

- `Infrastructure/Source Submission MVP — Session Plan.md` — high-level session plan this spec operationalizes
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — config file structure and verification queries
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — current org-ID inventory; cross-check for duplicates
- `Infrastructure/Non-GotSport Top Sources — Ingestion Design.md` — non-GotSport source types the MVP must support
- `Infrastructure/Pipeline Error Classification Taxonomy.md` — error codes if validation failures need taxonomy alignment

*FORGE — 2026-04-27*
