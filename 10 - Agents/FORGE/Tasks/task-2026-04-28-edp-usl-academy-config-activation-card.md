---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
due: 2026-04-30
priority: high
status: completed
completed: 2026-04-28
tags: [forge, task, edp, usl-academy, config, gotsport, stage2]
---

# Task: EDP + USL Academy Config Activation — Rapid Execution Card

## What to Build

A one-page field reference card FORGE uses the moment Presten delivers EDP and USL Academy org-IDs from the browser session. This is not a spec document — it is a live execution reference. No design decisions are made at execution time. Everything is pre-filled except the org-ID placeholders.

## Why This Exists

The Non-GotSport Source Implementation Priority List (filed 2026-04-28) confirmed EDP and USL Academy can both be executed in under 1 hour of receiving org-IDs. That estimate assumes no ambiguity at execution time. This card eliminates all remaining ambiguity. When Presten drops the org-IDs, FORGE opens this document, fills in two placeholders, and executes.

## Required Sections

### 1. Pre-Activation Gate (5 minutes)
Two checks before touching config:
- Stage 1 TX crawl has completed and been verified (confirm via Stage 1 Verification Statement in `Infrastructure/Stage 1 Post-Run Verification Protocol.md`)
- No active pipeline run in progress (check cron status)

If either check fails: do not proceed. Notify SENTINEL via queue.

### 2. EDP Config Entry
Exact config file path and structure. Pre-written config block with `[EDP_ORG_ID]` placeholder. FORGE fills in org-ID and pastes. No reformatting required.

### 3. USL Academy Config Entry
Same structure as EDP section. `[USL_ACADEMY_ORG_ID]` placeholder. If USL Academy has multiple org-IDs (possible — check browser session output), provide exact multi-entry format.

### 4. Post-Add Validation SQL (run immediately after config edit, before next crawl)
Three queries:
- Q1: Confirm new org-ID appears in the org_ids table (or equivalent config source) with status active
- Q2: Confirm no duplicate entries for the same org-ID
- Q3: Confirm org-ID is not already present in an archived or inactive state

Expected results stated for each query.

### 5. Briefing Entry Template
Pre-written briefing paragraph FORGE pastes into the next session briefing. Placeholders: `[EDP_ORG_ID]`, `[USL_ACADEMY_ORG_ID]`, `[DATE_EXECUTED]`. One sentence per source: config added, validation passed, first crawl expected at next pipeline run.

### 6. Stage 2 Trigger Condition
Single sentence: if Stage 1 TX crawl has verified ≥ 5,000 games and SENTINEL has issued written Stage 2 authorization, FORGE may proceed to Stage 2 expansion immediately after EDP + USL Academy activation. Reference: `task-2026-04-27-stage1-to-stage2-transition-protocol.md`.

## Quality Standard

SENTINEL must be able to verify EDP + USL Academy activation from this card alone — no other document required. The card is self-contained. The validation SQL must match the actual database schema (reference `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` for table names).

## Dependencies

- Browser session org-ID delivery (Presten action, April 28 or next session)
- `Infrastructure/Non-GotSport Source Implementation Priority List.md` (filed 2026-04-28) — confirm EDP and USL Academy remain ranked 1 and 2 at time of execution
- `Infrastructure/Stage 1 Post-Run Verification Protocol.md` — pre-activation gate reference

## File When Done

`02 - Tiger Tournaments/Projects/Infrastructure/EDP + USL Academy — Config Activation Card.md`

Mark this task `status: completed` in frontmatter.
