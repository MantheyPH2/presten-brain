---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: high
due: 2026-05-05
---

# Task: ECNL Migration Pre-Flight Checklist (Pre-June-1)

## Context

ELO delivered the June 2 post-migration verification spec — what to check the morning after migration. What does not exist: what must be true BEFORE the migration runs on June 1.

The June 1 ECNL migration is the highest-risk event of the year. If pre-conditions are not met:
- If the P0 dedup fix is not deployed, ECNL teams lose game history during migration
- If the backfill is incomplete, newly-discovered ECNL teams carry stale ratings into the migration
- If the migration script has not been tested on a sample, June 1 is a live experiment on production data
- If the daily pipeline is unstable, the first post-migration run may fail silently

ELO is separately writing the rating continuity spec (whether Elo ratings carry through correctly). This task covers FORGE's side: infrastructure, database state, and code readiness. Together, the FORGE pre-flight and ELO's rating continuity spec constitute the full pre-migration checklist.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Infrastructure/ECNL Migration Pre-Flight Checklist — June 2026.md`

---

### Section 1: Code Prerequisites

For each item, write the confirmation check Presten can run to verify it:

**1.1 ECNL dedup P0 fix deployed**
- Confirm by: `grep -n "verified" [path to ecnl-transition-dedup.js]` and confirm the fix is present
- Alternatively, run:
```sql
SELECT COUNT(*) FROM team_merges 
WHERE merge_type = 'ecnl_transition' 
  AND (verified IS NULL OR verified = false);
```
Expected result: 0. If > 0, the fix is not deployed.

**1.2 Exponential backoff deployed in sync runner**
- Confirm by: `grep -n "fetchWithBackoff" [path to crawl-gotsport-api-parallel.js]`
- Expected result: function found. If not found, the sync runner is still on the fixed 500ms delay and vulnerable to rate-limit casualties.

**1.3 Daily pipeline tested and stable**
- Confirm by: review the pipeline log for at least 3 consecutive successful daily runs before June 1
- Document the expected log location: `[path to pipeline logs]`
- A "successful" run is defined as: crawl completes + pipeline produces updated rankings + no error exit code

---

### Section 2: Data Prerequisites

**2.1 Stale team backfill complete (all 3 stages passed)**
- Confirm by: check the backfill log for Stage 3 completion marker (document the expected log location)
- Post-backfill verification: run the 3 verification queries from the Stale Team Backfill Runbook (`Infrastructure/Stale Team Backfill Runbook — April 2026.md`) — all three must pass
- If Stage 3 has not run, the 480-team batch is still stale. Proceed with caution — stale teams interacting with the ECNL migration may produce orphaned records.

**2.2 No new wave of stale teams in the 7 days before June 1**
- Run:
```sql
SELECT COUNT(*) 
FROM teams 
WHERE last_game_date < NOW() - INTERVAL '90 days' 
  AND is_active = true;
```
- Document the count as of migration day. If the count is more than 50 new stale teams since the backfill completed, investigate before proceeding — this suggests the backoff fix is not working or a new rate-limit event occurred.

**2.3 GotSport org-ID expansion deployed**
- Confirm at least TX and FL org IDs are in the production discovery config (the file from `Data Sources/GotSport USYS Org-ID Master List.md`)
- If neither org ID is confirmed, document: "org-ID expansion not deployed — ECNL state-association teams may not be discovered before migration." This is not a blocker for June 1 but should be noted in the pre-flight report.

---

### Section 3: Migration Script Prerequisites

**3.1 Confirm which script executes the ECNL migration**
- Locate the migration script (check `scripts/`, `migrations/`, or wherever DB migration scripts are stored)
- Document the file path here. If the script does not exist yet, this is a critical gap — flag to SENTINEL immediately.

**3.2 Confirm the script has been tested on a sample**
- Required: at least 10 ECNL teams tested (mix of merged and unmerged cases)
- Confirm: the test run produced no duplicate `team_merges` entries and no orphaned game records
- Document the test date and sample size.

**3.3 Confirm a pre-migration database backup exists**
- Backup must be taken within 24 hours of the June 1 migration run
- Document the backup command and expected location: `[pg_dump command and output path]`
- This backup is the rollback target if the migration produces bad data and cannot be reversed in-place.

---

### Section 4: Pre-Flight Go/No-Go Rule

Write this as the final section of the document:

```
CLEARED TO PROCEED if ALL of the following are true:
□ Section 1.1: P0 fix deployed (query returns 0)
□ Section 1.2: fetchWithBackoff present in sync runner
□ Section 1.3: At least 3 successful daily pipeline runs logged
□ Section 2.1: Backfill Stages 1–3 complete and verified
□ Section 3.1: Migration script file path identified
□ Section 3.3: Pre-migration backup taken within 24 hours

NOT CLEARED — escalate to SENTINEL if ANY of the following:
□ Section 1.1: ecnl_transition merges with verified = false or NULL > 0
□ Section 3.1: Migration script does not exist
□ Section 2.1: Stage 3 backfill did not complete
□ Section 3.3: No recent backup exists
```

Note: Sections 1.2, 2.2, 2.3, and 3.2 are recommended but not blocking. If they fail, document the gap and proceed with awareness.

---

## Deliverable

`02 - Tiger Tournaments/Projects/Infrastructure/ECNL Migration Pre-Flight Checklist — June 2026.md`

The document must:
- Have executable confirmation steps for every Section 1 and Section 2 item (not "confirm X is done" — must include the command or query)
- Identify the migration script file path (or flag if unknown)
- Include the go/no-go rule with explicit pass/fail criteria
- Summary in briefing: checklist filed, migration script [identified at: path / NOT FOUND — escalate]

## Definition of Done

- Section 1 items have specific commands to confirm deployment
- Section 2 items have specific SQL queries with expected results
- Section 3.1 migration script path is identified (or flagged as missing)
- Go/no-go rule is written with checkbox format
- Summary in briefing includes: migration script path, any pre-conditions currently not met

## Why Now (Due May 5)

If written after May 14 (DSS), the checklist competes with post-DSS recovery work. May 5 leaves enough lead time to identify gaps (especially if the migration script is missing or untested) and address them before the June 1 window.

## Coordination Note

ELO is writing a separate spec — `Rankings/ECNL Rating Continuity Spec — June 2026.md` — covering how Elo ratings carry through the migration. That spec may produce an additional pre-flight requirement for FORGE (e.g., a re-attribution step in the migration script). When ELO's spec is complete, review it and add any FORGE-side requirements to Section 3 of this checklist.

## References

- `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — post-backfill verification queries (Section 2.1)
- `Reports/ecnl-dedup-verified-fix-2026-04-24.md` — P0 fix spec (Section 1.1)
- `Reports/backoff-implementation-2026-04-24.md` — backoff spec (Section 1.2)
- ELO task `task-2026-04-24-ecnl-rating-continuity-spec.md` — companion spec, may add Section 3 requirements
