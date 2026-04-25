---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-25
priority: high
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/May 1 Daily Pipeline Deployment Session Guide.md"
---

# Task: May 1 Daily Pipeline Deployment Session Guide

## Context

May 1 is a hard deadline: `run-daily.sh` and `get-active-team-ids.sh` deployed and tested. The `implement-daily-pipeline` task is in_progress — spec is complete in `Infrastructure/Daily Pipeline Cadence Scope.md` Section 6, but script writing and cron configuration require Presten.

What does not exist: a session execution guide. The April 28 session has a Reference Card — a structured document Presten opens and works through linearly. May 1 needs the same. Without it, Presten must navigate Section 6 of the Cadence Scope doc and reconstruct the execution order in-session — fragile under time pressure.

This task: write the May 1 session guide. Format matches `Product & Planning/April 28 Escalation Reference Card.md`.

## What to Do

Read before starting:
- `Infrastructure/Daily Pipeline Cadence Scope.md` — especially Section 6 (the implementation spec)
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — validation criteria post-deploy
- `Infrastructure/April 29 Pipeline Health Check — Post-Fix Protocol.md` — reference for the GREEN/YELLOW/RED decision structure

### Section 1: Session Overview

| Item | Value |
|------|-------|
| Session name | May 1 Daily Pipeline Deployment |
| Hard deadline | May 1, 2026 |
| Estimated time | [pull from Daily Pipeline Cadence Scope.md Section 6] |
| Prerequisites | April 29 pipeline health check is GREEN; no active backfill runs |
| Deliverable | `run-daily.sh` and `get-active-team-ids.sh` live on server; cron configured and tested |

### Section 2: Pre-Session Checklist

Before writing a single line of code, Presten confirms:

- [ ] FORGE's April 29 briefing shows GREEN pipeline health check status
- [ ] No active backfill runs in progress (`ps aux | grep crawl-gotsport | grep -v grep` returns nothing)
- [ ] Server access confirmed
- [ ] `Daily Pipeline Cadence Scope.md` Section 6 open in a second tab

If any item is not confirmed: do not begin the May 1 session. Flag to SENTINEL.

### Section 3: Script Creation — Step by Step

For each script, write a subsection:

**Script: `get-active-team-ids.sh`**
- File path on server: [pull from Cadence Scope doc]
- Purpose: [one sentence]
- Complete script contents: [pull from Cadence Scope Section 6 — copy-paste ready, no placeholders]
- Verify after writing: `bash get-active-team-ids.sh 2>&1 | head -20`
- Expected output: [what a passing first run looks like]

**Script: `run-daily.sh`**
- File path on server: [pull from Cadence Scope doc]
- Purpose: [one sentence]
- Complete script contents: [pull from Cadence Scope Section 6 — copy-paste ready, no placeholders]
- Verify after writing: `bash run-daily.sh --dry-run 2>&1 | head -20` (or equivalent test command from the spec)
- Expected output: [what a passing dry-run looks like]

Scripts must contain NO `[placeholder]` blocks in the final deliverable. Pull all code directly from the Cadence Scope spec.

### Section 4: Cron Configuration

Exact cron entries to add (pull timing from Cadence Scope Section 6):
```
[cron schedule] [full command with absolute path]
```

Verify: `crontab -l`
Expected output: the two new cron entries visible in the listing.

Note: if the cron already contains entries for these scripts (from a prior partial attempt), document how to identify and remove the old entry before adding the new one.

### Section 5: Deployment Validation

After cron is configured, run the validation checks from `Pipeline Deployment Validation Spec — April 2026.md` relevant to the daily pipeline:

- [ ] `[check 1 from validation spec]` — expected: `[value]`
- [ ] `[check 2]` — expected: `[value]`

GREEN: all checks pass. Proceed to step 6.
YELLOW: one check fails. Do not kill cron — investigate the specific failure before deciding.
RED: both checks fail. Remove cron entries. File a FORGE queue item. Do not retry until FORGE issues a corrected spec.

### Section 6: Post-Deploy Actions

After successful deployment:
1. Note in FORGE's next briefing: "May 1 pipeline deployment complete. Cron live at [schedule]. First automated run scheduled for [datetime]."
2. Mark `task-2026-04-24-implement-daily-pipeline.md` status: `completed`.
3. Watch the first cron run: at [scheduled datetime], confirm the run fires, completes without error, and logs output to [expected log file location].
4. File `implement-daily-pipeline` closure note in FORGE briefing: actual vs. expected game count from first automated run.

## Definition of Done

- Session guide written at `Infrastructure/May 1 Daily Pipeline Deployment Session Guide.md`
- All 6 sections present
- Scripts are complete and copy-paste ready — no `[placeholder]` blocks remain
- Cron config is exact and verifiable
- GREEN/YELLOW/RED validation decision criteria are present
- Pre-session checklist explicitly requires April 29 GREEN health check
- Summary in briefing: "May 1 session guide filed. Estimated time: [N min]. Scripts ready: get-active-team-ids.sh, run-daily.sh. Prerequisites: April 29 GREEN."

## Why This Matters

May 1 is a hard deadline and the daily pipeline is the operational backbone of Evo Draw's ongoing data freshness. A failed May 1 session means manual crawl runs until the next Presten window. A session guide written in advance means Presten opens one document, works linearly, and completes deployment in the estimated window with no on-the-fly decision-making.

## References

- `Infrastructure/Daily Pipeline Cadence Scope.md` Section 6
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md`
- `Infrastructure/April 29 Pipeline Health Check — Post-Fix Protocol.md`
- `Product & Planning/April 28 Escalation Reference Card.md` — format reference
- `task-2026-04-24-implement-daily-pipeline.md`
