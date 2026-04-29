---
title: Archive Inactive Teams — Production Execution Authorization Request
tags: [infrastructure, archive, teams, sentinel, authorization, evo-draw]
created: 2026-04-28
updated: 2026-04-29
author: FORGE
status: pre-staged template — fill when dry-run results received
task: task-2026-04-28-archive-production-execution-template
---

# Archive Inactive Teams — Production Execution Authorization Request

> [!warning] Pre-staged template
> This document is pre-staged. Sections 1–3 are filled when Presten provides `archive-inactive-teams.js --dry-run` output (GREEN or YELLOW verdict from the dry-run authorization request). FORGE fills this document and files the SENTINEL authorization request within 30 minutes of dry-run receipt.
>
> **FORGE does NOT execute production run until SENTINEL signs Section 4.**

**Prepared by:** FORGE  
**Precursor document:** `Infrastructure/Archive Inactive Teams — Dry-Run Authorization Request.md` (filed after dry-run output received)  
**SENTINEL authorization:** PENDING

---

## Section 1: Dry-Run Results Summary

| Field | Value |
|-------|-------|
| Dry-run execution date | _[fill when received]_ |
| Total teams flagged for archive | _[fill — integer count]_ |
| Threshold classification | _[GREEN / YELLOW / RED]_ |
| Threshold band | _[fill — e.g., "161 teams — within GREEN 155–165 band"]_ |
| Unexpected teams in archive set? | _[YES / NO]_ |
| Sample review: spot-checked N teams | _[fill — how many reviewed and what was found]_ |
| Full dry-run output location | _[paste or reference]_ |

**Threshold reference (corrected per SENTINEL ruling 2026-04-28 — authoritative source: `Infrastructure/Archive Inactive Teams — SQL Fix Verification and Dry-Run Readiness.md`):**
- GREEN: 155–165 teams — proceed to Section 4 authorization request
- YELLOW: 140–154 or 166–185 teams — proceed with SENTINEL note; SENTINEL may request expanded spot check before authorizing
- RED: < 140 or > 185 teams — do NOT file this document; file SENTINEL anomaly queue item instead

> [!caution] RED verdict protocol
> If dry-run returns RED, FORGE does NOT file this authorization request. FORGE files a SENTINEL queue item (priority: urgent) with the anomaly count and requests guidance. Production archive does not proceed until SENTINEL resolves the anomaly.

---

## Section 2: Archive Criteria Confirmation

Confirm that all teams in the archive set meet the documented criteria from the archive workflow specification.

- [ ] No games ingested in the past 180 days for any archived team
- [ ] No active org-ID match in current crawl config for any archived team
- [ ] No `team_merges` entry pointing from an archived team to an active team
- [ ] Spot-check: _[N]_ teams reviewed manually — all meet the three criteria above

**Verification query used:**
```sql
-- Confirm archivable teams meet all criteria
SELECT t.id, t.name, t.state,
       MAX(g.ingested_at) AS last_game_ingested,
       COUNT(g.id) AS recent_games,
       COUNT(tm.id) AS active_merges
FROM teams t
LEFT JOIN games g ON (g.home_team_id = t.id OR g.away_team_id = t.id)
  AND g.ingested_at > NOW() - INTERVAL '180 days'
LEFT JOIN team_merges tm ON tm.from_team_id = t.id
  AND tm.to_team_id IN (SELECT id FROM teams WHERE archived = false)
WHERE t.archived = false
GROUP BY t.id
HAVING MAX(g.ingested_at) IS NULL
   AND COUNT(g.id) = 0
   AND COUNT(tm.id) = 0
ORDER BY t.name;
```

Expected: All rows in dry-run output appear in this query result.

**Exceptions to document:** _[fill — list any teams in the archive set that do not strictly meet all three criteria, with justification for why they are still safe to archive, or "none"]_

---

## Section 3: Production Execution Plan

| Field | Value |
|-------|-------|
| Proposed production run date | _[fill — minimum 24h after SENTINEL authorization received]_ |
| Proposed production run time | _[fill — during low-traffic window, e.g., 02:00–04:00 local]_ |
| Command to execute | `node archive-inactive-teams.js` (no `--dry-run` flag) |
| Pre-run snapshot required | YES |
| Post-run verification required | YES |
| Rollback plan | Set `archived = false` for all rows where `archived_at >= '[run timestamp]'` |
| Rollback SLA | Within 2 hours of SENTINEL flag |

**Pre-run snapshot (run immediately before executing):**
```sql
-- Capture current active team count before archive run
SELECT
  COUNT(*) AS total_teams,
  COUNT(*) FILTER (WHERE archived = false) AS active_teams,
  COUNT(*) FILTER (WHERE archived = true) AS already_archived,
  NOW() AS snapshot_time
FROM teams;
```

Record result: `_[fill after running]_`

**Post-run verification (run within 15 minutes of execution):**
```sql
-- Confirm archive count matches dry-run count
SELECT
  COUNT(*) FILTER (WHERE archived = true AND archived_at >= '_[run timestamp]_') AS newly_archived,
  COUNT(*) FILTER (WHERE archived = false) AS remaining_active,
  NOW() AS verification_time
FROM teams;
```

Expected: `newly_archived` = dry-run team count ± 2 (small delta acceptable if a team received new data between dry-run and production run). If delta > 5: flag to SENTINEL before proceeding.

**Rollback SQL (execute only on SENTINEL instruction):**
```sql
-- ROLLBACK — only run if SENTINEL flags anomaly
UPDATE teams
SET archived = false,
    archived_at = NULL
WHERE archived_at >= '_[production run timestamp]_';
```

Confirm rollback with: `SELECT COUNT(*) FROM teams WHERE archived = false;` — should match pre-run snapshot `active_teams` value.

---

## Section 4: SENTINEL Authorization Request

**FORGE requests SENTINEL authorization to execute `archive-inactive-teams.js` in production (no `--dry-run`).**

All conditions below must be confirmed before SENTINEL authorizes.

Authorization conditions:

- [ ] Dry-run result classified **GREEN or YELLOW** (Section 1 threshold classification filled)
- [ ] No unexpected teams identified in spot check (Section 2 exceptions = none, or SENTINEL reviewed exceptions and approved)
- [ ] ELO has confirmed no teams in the archive set have active rating records used in the past 90 days (ELO check: `SELECT COUNT(*) FROM elo_ratings WHERE team_id IN ([archive set IDs]) AND last_game_date > NOW() - INTERVAL '90 days'` = 0)
- [ ] Presten has reviewed and acknowledged archive set size
- [ ] Pre-run snapshot captured (Section 3) and available for SENTINEL review

**SENTINEL authorization:**

SENTINEL authorization status: **PENDING**  
SENTINEL signature: _________________  
SENTINEL authorization date: _________________  
Notes from SENTINEL: _[fill if any conditions or scope changes before authorizing]_

---

## Section 5: Post-Execution Confirmation

FORGE files within 30 minutes of production execution completing.

| Check | Result |
|-------|--------|
| Rows archived (newly_archived count) | _[fill from post-run verification query]_ |
| Matches dry-run count? | _[YES / NO — delta: fill]_ |
| Delta within tolerance (≤ 5)? | _[YES / NO]_ |
| Pre-run active team count | _[fill from pre-run snapshot]_ |
| Post-run active team count | _[fill from post-run verification]_ |
| Post-archive rankings recompute triggered? | _[YES / NO — if YES: who triggered and when]_ |
| Any errors during execution? | _[YES / NO — if YES: paste error output]_ |
| SENTINEL notified of completion? | _[YES / NO]_ |

**Final status after post-execution confirmation:** _[COMPLETE / PARTIAL — flag to SENTINEL / FAILED — rollback initiated]_

---

## References

- `Infrastructure/Archive Inactive Teams — Dry-Run Authorization Request.md` — precursor document (dry-run stage)
- `task-2026-04-26-archive-sql-fix-and-dry-run.md` — GROUP BY fix and dry-run spec, archive criteria
- `task-2026-04-22-archive-workflow.md` — original archive workflow design
- `Infrastructure/FORGE-Blocked-Task-Dependency-Tracker.md` — row #3 (archive dry-run) unblocks this document

*FORGE — 2026-04-28*
