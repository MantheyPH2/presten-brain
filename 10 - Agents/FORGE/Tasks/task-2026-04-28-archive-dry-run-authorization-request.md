---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
due: 2026-04-28
priority: high
status: pending
tags: [forge, task, archive, dry-run, authorization, sentinel-gate]
---

# Task: Archive Dry-Run — SENTINEL Authorization Request

## What to Build

A structured authorization request document FORGE files within 30 minutes of receiving the archive dry-run output from Presten. This document replaces the informal "dry-run results posted to briefing" pattern. SENTINEL cannot formally authorize the live archive execution without a filed authorization request.

## Why This Exists

The Archive SQL Fix (GROUP BY correction in `archive-inactive-teams.js`) has been specced and reviewed. The dry-run is the final gate before production. Once the dry-run runs, FORGE will have raw output. That output must be structured, verified against GREEN thresholds, and submitted for SENTINEL authorization in a reviewable format — not embedded in a briefing paragraph.

SENTINEL authorizes the live archive run from this document. If SENTINEL does not receive this document, the archive does not execute.

## Trigger

File this document within 30 minutes of Presten sending FORGE the `--dry-run` output from `archive-inactive-teams.js`.

## Required Sections

### 1. Dry-Run Metadata
- Date and time dry-run was executed
- Script version / commit hash (if known)
- Flag applied: `--dry-run` confirmed (not live)
- Raw output line count

### 2. Output Summary
Fill in from dry-run output:
- Total teams flagged for archive: `[N]`
- Date range of last_game_date for flagged teams: `[EARLIEST]` to `[LATEST]`
- Number of teams with last_game_date > 90 days ago: `[N]`
- Number of teams with 0 games ever: `[N]`
- Any teams flagged that FORGE identifies as unexpected (active teams, known clubs): list or "None"

### 3. GREEN / YELLOW / RED Verdict

Reference threshold from `task-2026-04-22-archive-workflow.md` and `task-2026-04-26-archive-sql-fix-and-dry-run.md`:

| Signal | GREEN | YELLOW | RED |
|--------|-------|--------|-----|
| Total teams flagged | 80–150 | 50–80 or 150–200 | < 50 or > 200 |
| No active/known teams in list | Confirmed | 1–2 found, explainable | 3+ found OR unexplained |
| All flagged teams have last_game_date > 90 days | Confirmed | 1–2 exceptions | Any exception unexplained |

FORGE states overall verdict: **GREEN / YELLOW / RED**

If YELLOW: FORGE explains each exception and recommends whether to proceed.
If RED: FORGE does not request authorization — files a separate queue item to SENTINEL explaining the anomaly.

### 4. SENTINEL Authorization Request

> FORGE requests SENTINEL authorization to execute `archive-inactive-teams.js` (live, without `--dry-run`) against the production database. Dry-run output reviewed above. Verdict: [GREEN/YELLOW]. No active teams identified in flagged set. FORGE will execute within 30 minutes of SENTINEL written authorization and file a post-execution results confirmation.

### 5. Post-Authorization Execution Commitment
State the exact steps FORGE will take within 30 minutes of authorization:
1. Run script without `--dry-run`
2. Capture output (row count confirmed)
3. File one-paragraph execution confirmation in `Infrastructure/Archive Inactive Teams — Execution Confirmation.md`
4. Update this task to `status: completed`

## File When Done

`02 - Tiger Tournaments/Projects/Infrastructure/Archive Inactive Teams — Dry-Run Authorization Request.md`

Mark this task `status: completed` in frontmatter after filing.

## Dependencies

- Presten executes `--dry-run` during April 28 session and shares output with FORGE
- `archive-inactive-teams.js` GROUP BY fix must be applied before dry-run (confirm with Presten)
- Reference: `Infrastructure/Archive Workflow.md` (thresholds and SQL spec)
