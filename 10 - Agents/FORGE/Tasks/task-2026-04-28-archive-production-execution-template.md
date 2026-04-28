---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: completed
completed: 2026-04-28
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Archive Inactive Teams — Production Execution Authorization Request.md"
---

# Task: Archive Inactive Teams — Production Execution Authorization Request Template

## Context

FORGE has a dry-run authorization request blocked on Presten providing `--dry-run` output. That document covers the dry-run stage only. What does not exist: the follow-on document FORGE will file to SENTINEL requesting authorization to run `archive-inactive-teams.js` in **production** (no `--dry-run` flag) once the dry-run comes back GREEN.

When the dry-run output arrives, FORGE will need to file this authorization request within 30 minutes. Without a pre-staged template, FORGE will spend that 30 minutes authoring the document under time pressure instead of filling in confirmed values. This task pre-stages the production authorization request so that when the dry-run is confirmed GREEN, FORGE fills the blanks and submits to SENTINEL immediately.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Archive Inactive Teams — Production Execution Authorization Request.md`

---

### Section 1: Dry-Run Results Summary

| Field | Value |
|-------|-------|
| Dry-run execution date | _[fill when received]_ |
| Total teams flagged for archive | _[fill]_ |
| Threshold classification | GREEN / YELLOW / RED |
| Threshold band | _[fill: e.g., "94 teams — within GREEN 80–150 band"]_ |
| Unexpected teams in archive set? | YES / NO |
| Sample review: spot-checked N teams | _[fill: how many reviewed and findings]_ |

---

### Section 2: Archive Criteria Confirmation

Confirm that all teams in the archive set meet the documented criteria from `task-2026-04-26-archive-sql-fix-and-dry-run.md`:

- [ ] No games ingested in the past 180 days
- [ ] No active org-ID match in current crawl config
- [ ] No `team_merges` entry pointing to an active team

Any exceptions to document: _[fill or "none"]_

---

### Section 3: Production Execution Plan

| Field | Value |
|-------|-------|
| Proposed production run date | _[fill: minimum 24h after authorization]_ |
| Command to execute | `node archive-inactive-teams.js` (no `--dry-run`) |
| Pre-run snapshot required? | YES — `SELECT COUNT(*) FROM teams WHERE archived = false` before execution |
| Post-run verification query | `SELECT COUNT(*) FROM teams WHERE archived = true AND updated_at >= '[run date]'` |
| Rollback plan | Set `archived = false` for all rows where `archived_at >= '[run timestamp]'` |
| Rollback SLA | Within 2 hours of SENTINEL flag |

---

### Section 4: SENTINEL Authorization Request

**FORGE requests SENTINEL authorization to execute `archive-inactive-teams.js` in production.**

Authorization conditions:
- [ ] Dry-run result classified GREEN or YELLOW
- [ ] No unexpected team entries identified in spot check
- [ ] ELO has confirmed no teams in the archive set have active rating records in the past 90 days
- [ ] Presten has reviewed and acknowledged archive set size

SENTINEL authorization status: **PENDING**
SENTINEL signature: _________________
Date: _________________

---

### Section 5: Post-Execution Confirmation

FORGE files within 30 minutes of production execution:

| Check | Result |
|-------|--------|
| Rows archived (count) | _[fill]_ |
| Matches dry-run count? | YES / NO — delta: _[fill]_ |
| Post-archive rankings recompute triggered? | YES / NO |
| Any errors during execution? | YES / NO — details: _[fill or "none"]_ |

---

## Definition of Done

- Template filed at deliverable path by April 30
- All section headers and field labels are present
- Fill-in fields are clearly marked `_[fill]_`
- When dry-run results arrive: FORGE fills Sections 1–3 and files SENTINEL authorization request within 30 minutes
- FORGE does NOT execute production run until SENTINEL authorization is received

## Why This Matters

The archive step is irreversible without a deliberate rollback. Filing an authorization request to SENTINEL — with the dry-run results reviewed and documented — is the correct gate before production execution. Pre-staging this template ensures FORGE is never in a position of skipping the gate because the document authoring takes too long.

## References

- `task-2026-04-28-archive-dry-run-authorization-request.md` — blocked precursor (dry-run stage)
- `task-2026-04-26-archive-sql-fix-and-dry-run.md` — GROUP BY fix and dry-run spec
- `task-2026-04-22-archive-workflow.md` — original archive workflow design
