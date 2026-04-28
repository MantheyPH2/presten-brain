---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-04-29
priority: high
status: completed
completed: 2026-04-27
topic: stage2-org-id-duplicate-cross-check
---

# Task: Stage 2 Org-ID Duplicate Cross-Check vs. Stage 1

## Context

FORGE has pre-staged the Stage 2 org-ID config additions in `Infrastructure/Stage 2 Org-ID Config Pre-Staging.md`. Before SENTINEL can authorize Stage 2 expansion, criterion 5.1 must be met: all Stage 2 org-IDs confirmed non-duplicate with Stage 1 entries.

This task satisfies 5.1 verification. It is separate from the Section 5 incorporation task because the cross-check requires actual analysis, not just a document edit.

The risk: if a Stage 2 org-ID duplicates a Stage 1 org-ID, the pipeline will attempt to ingest that org's games twice. Depending on the upsert/conflict logic, this could cause: (a) silent no-op on duplicates (harmless but wastes a crawl slot), (b) a duplicate key error that halts that source, or (c) a race condition if two crawlers hit the same org-ID in the same pipeline run.

SENTINEL needs confidence that no duplicate exists before authorizing Stage 2 to go live.

## Task

Produce a cross-check document at `Infrastructure/Stage 2 Org-ID Duplicate Cross-Check.md` that confirms zero overlap between Stage 1 and Stage 2 org-ID lists.

### Section 1: Stage 1 Org-ID List

List every org-ID currently active in Stage 1 (the current crawl config). Source this from the GotSport Org-ID Master Reference (`Infrastructure/GotSport Org-ID Master Reference — 2026-04-27.md`) or the crawl config file directly.

For each org-ID, include:
- Org ID value
- State / region
- Organization name

### Section 2: Stage 2 Proposed Org-ID List

List every org-ID proposed for Stage 2 (from the Pre-Staging document). Same format.

### Section 3: Cross-Check Results

Compare the two lists. For each Stage 2 org-ID:
- UNIQUE: Not present in Stage 1 (expected result for all entries)
- DUPLICATE: Present in Stage 1 — flag with explanation of why it appeared in both lists

If any DUPLICATE is found, do NOT proceed to Section 4. File a queue item to SENTINEL and hold the Section 5.1 criterion.

### Section 4: Verification Statement

If zero duplicates found, write:

> FORGE confirms: all [N] Stage 2 proposed org-IDs are unique with respect to Stage 1. No org-ID appears in both lists. Stage 2 config additions will not create crawl duplicates. Section 5.1 criterion MET.

Sign with FORGE and session timestamp.

## Deliverable

File to: `Infrastructure/Stage 2 Org-ID Duplicate Cross-Check.md`

Update this task to `status: completed` when filed.

## Notes

This document is a required verification artifact for Section 5.1. SENTINEL will reference it when authorizing Stage 2. If any duplicates are found, SENTINEL must be notified immediately — do not silently remove the duplicate from the Stage 2 list without SENTINEL awareness.

Due April 29 — needed before the April 29 gate session so SENTINEL can confirm 5.1 status during the session.
