---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-25
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/May 1 Deployment — Day-Of Runbook.md"
---

# Task: May 1 Deployment — Day-Of Runbook

## Context

The May 1 Pre-Authorization Checklist (filed 2026-04-25) defines the gate conditions Presten confirms before starting. The Pipeline Deployment Validation Spec covers what to validate after the first run. What does not exist: the step-by-step runbook for the deployment session itself — what Presten does from "checklist complete" to "first run launched and confirmed healthy."

This is a gap that will cause friction on May 1 morning. The checklist passes, Presten opens a terminal, and then: which command starts the pipeline? In what order do the components start? How long does the first run take? What does healthy output look like vs. an error? Without a day-of runbook, May 1 becomes an improvised session with a 5.5-hour session guide in one hand and a terminal in the other.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/May 1 Deployment — Day-Of Runbook.md`

This is a linear, step-by-step document. Presten follows it top to bottom. Every step has: what to do, what success looks like, what to do if it fails.

---

### Pre-Runbook Gate

One paragraph referencing the Pre-Authorization Checklist. If all 5 sections of the checklist are ✅, proceed to Step 1. If not: do not proceed; contact SENTINEL.

---

### Step 1: Launch `run-daily.sh`

- Exact command to run (pull from the deployment spec or daily pipeline design — FORGE knows the correct invocation)
- Expected terminal behavior immediately after launch (background job? foreground with output?)
- What to look for in the first 60 seconds (any startup errors that indicate misconfiguration vs. normal crawl initialization)

---

### Step 2: Monitor Initial Crawl Progress

- How to verify the crawl is actively running vs. hung
- Which log file or output stream to tail
- What normal crawl output looks like (example log line format if knowable from the pipeline design docs)
- How long a normal first crawl takes: min estimate, expected estimate, "something is wrong" threshold
- One command Presten runs to check game ingestion is happening (count of new rows in `games` table since launch)

---

### Step 3: First Run Completion Check

After the crawl completes or after [expected time window]:

- Command to confirm the run completed (exit code check, or log message indicating completion)
- Quick health query: new game count ingested in this run, any source-level errors
- Pass threshold: what game count confirms the first run worked? (reference the Morning-After Runbook GREEN criteria)
- Fail threshold: what triggers the RED-state escalation? (reference the same runbook)

---

### Step 4: Post-Run Decision

| Result | Action |
|--------|--------|
| GREEN (count ≥ threshold, no errors) | File Stage 3 completion to FORGE briefing; SENTINEL authorizes Stage 2 window |
| YELLOW (partial output or minor error) | Run the YELLOW-state protocol from the Morning-After Runbook. Stage 2 authorization pending. |
| RED (crash, no new games, critical error) | Execute RED-state remediation. Do NOT proceed to Stage 2. Contact SENTINEL. |

---

### Section: Expected First-Run Numbers

Pull from the Deployment Validation Spec or the Stage 1 Results Report (if available by April 29):
- Expected game count for a healthy first run of the active org-ID set
- Expected crawl duration
- Expected source breakdown (how many games from GotSport vs. other active sources)

If the Stage 1 Results Report is not yet available when FORGE writes this runbook, note the placeholder: "Expected game count: pending Stage 1 results. See `Infrastructure/Stage 1 Backfill Results Report.md` when available."

---

### Section: Rollback Protocol

If the first run fails in a way that requires rollback:
- How to halt a running crawl without data corruption
- Whether a partial first run leaves games in an inconsistent state (can duplicates occur from a partial run + re-run?)
- How to clean up partial data before re-running
- When to contact SENTINEL vs. attempt self-remediation

---

## Quality Standard

- Every step must have a **success signal** and a **failure action**. No step can end in "investigate."
- Commands must be copy-paste ready where possible. Reference the config file from the Org-ID Config Edit Reference if relevant to step 1.
- The document must be completable as a runbook — Presten follows it linearly, checking boxes.
- If a step requires knowledge FORGE does not yet have (e.g., exact log format before the first run ever happens), note the gap explicitly with `[TBD: confirm after Stage 1 run]` and describe what to fill in from Stage 1 observations.

## Why This Matters

May 1 is a high-stakes session: first production run of the daily pipeline, Stage 3 of the crawl expansion, gate for Stage 2 authorization. The Pre-Auth Checklist covers the 5-minute readiness check. The Morning-After Runbook covers the next-day health review. This document fills the gap in between — the deployment session itself. Without it, May 1 depends on FORGE being available for real-time support, which is not a reliable dependency.

## References

- `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` — pre-conditions this runbook follows
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — post-run health check
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — validation query patterns
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — config context
- `task-2026-04-25-stage1-backfill-results-report.md` — Stage 1 results that inform expected game counts
