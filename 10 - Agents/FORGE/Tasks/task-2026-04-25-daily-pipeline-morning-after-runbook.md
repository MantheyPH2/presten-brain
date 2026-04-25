---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Daily Pipeline Morning-After Validation Runbook.md"
---

# Task: Daily Pipeline Morning-After Validation Runbook

## Context

The May 1 Daily Pipeline Deployment Session Guide (`Infrastructure/May 1 Daily Pipeline Deployment Session Guide.md`) covers everything Presten does on May 1 to deploy `run-daily.sh` and `get-active-team-ids.sh`. The guide ends at deployment confirmation. It does not cover what Presten does on May 2 morning to verify the pipeline ran correctly overnight.

This is a gap. The first unattended pipeline run is the highest-risk moment — cron timing errors, path issues, script failures, and silent empty-output runs are all more likely on the first run than any subsequent one. Without a morning-after runbook, Presten has no systematic check and no pass/fail criteria.

The May 1 Session Guide already notes: "Stage 3 backfill must NOT run same night as first daily pipeline run." The morning-after runbook is where Presten confirms the first run succeeded before any backfill work proceeds.

## What to Do

### Step 1: Define the Verification Sequence

Write a 5-step morning-after verification sequence. Each step should be a copy-paste command or specific check. Cover:

1. **Cron log check** — confirm `run-daily.sh` fired at scheduled time. What log file to check, what a successful fire looks like vs. a missed fire.
2. **Output file check** — confirm the pipeline produced output (game records, not empty result). What to check, expected row-count range for a first run (day N of the season — expected to be low; note that and set a non-zero floor).
3. **Error log check** — confirm no ERROR-level output in the pipeline log for that run. What the log location is, what to grep for.
4. **DB ingestion check** — one SQL query: confirm `MAX(game_date)` advanced since yesterday (proves games moved from pipeline output into the DB). Copy-paste query, expected output format.
5. **Stage 3 backfill gate** — confirm Step 2 and Step 4 pass before any Stage 3 backfill work proceeds. If either fails, STOP and contact FORGE.

Pull cron log paths, script names, and log file conventions from the May 1 Session Guide (`Infrastructure/May 1 Daily Pipeline Deployment Session Guide.md`) and the pipeline implementation specs (`Infrastructure/Daily Pipeline Cadence — Scope and Schedule.md` or equivalent).

### Step 2: Define GREEN/YELLOW/RED Decision for May 2

After the 5-step sequence, write a decision block:

- **GREEN (proceed):** All 5 checks pass. Stage 3 backfill can proceed. FORGE is informed pipeline is live.
- **YELLOW (investigate):** Steps 1–3 pass, Step 4 shows no new games. This may be correct if no games ran overnight — FORGE checks the game calendar for the prior day before escalating.
- **RED (halt):** Step 1 shows the cron did not fire, OR Step 3 shows ERROR in the log. Stage 3 is halted. Presten pings FORGE with the log output. Do not re-run manually until FORGE reviews.

### Step 3: Write the Delivery Document

Deliverable: `02 - Tiger Tournaments/Projects/Infrastructure/Daily Pipeline Morning-After Validation Runbook.md`

Format:

```
## When to Run
[May 2 morning, before any other pipeline or backfill work]

## Prerequisites
[May 1 deployment completed (GREEN from Session Guide Section 6)]

## 5-Step Verification Sequence
[Steps 1–5, each with copy-paste command or query]

## GREEN / YELLOW / RED Decision
[Three blocks with criteria and next action]

## If RED: What to Send FORGE
[Template for Presten to copy paste log output — reduces round-trips]
```

## Definition of Done

- All 5 checks have copy-paste commands or queries (no "check the log" without specifying which log and how)
- GREEN/YELLOW/RED criteria are numerical or binary — no judgment calls required
- Stage 3 backfill gate is explicit: Presten cannot miss it
- Document is under 1 page when printed — fast to use under pressure
- Summary in briefing: "Morning-After Runbook filed. 5-step check, GREEN/YELLOW/RED decision, Stage 3 gate included."

## Why This Matters

The May 1 deployment is the hardest deadline in the current sprint. If the pipeline fails silently on its first run, and Presten runs Stage 3 backfill on top of it, the result is a corrupted state that requires a manual rollback. The Morning-After Runbook is 10 minutes of work on May 2 that prevents that scenario entirely.

## References

- `02 - Tiger Tournaments/Projects/Infrastructure/May 1 Daily Pipeline Deployment Session Guide.md`
- `02 - Tiger Tournaments/Projects/Infrastructure/April 29 Pipeline Health Check — Post-Fix Protocol.md`
- `02 - Tiger Tournaments/Projects/Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` (for Stage 3 sequencing)
