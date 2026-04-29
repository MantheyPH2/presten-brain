---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-29
priority: medium
due: 2026-05-01 EOD
status: pending
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Expansion Authorization Trigger Document.md (all sections except post-May-1 actuals)"
---

# Task: Stage 2 Trigger Document Pre-Population

## Context

The Stage 2 Expansion Authorization Trigger Document (`Infrastructure/Stage 2 Expansion Authorization Trigger Document.md`) is the gate document SENTINEL uses to authorize expanding from Stage 1 to Stage 2 (additional org-IDs / sources). Section 1 requires Week 1 pipeline run data (May 1–3 actuals) and cannot be filled until May 4.

However, every other section — criteria definitions, source configurations, expansion scope, monitoring rubric references, risk assessment — can be pre-populated now with vault-available information. The goal: on May 4, FORGE fills Section 1 actuals and the document is immediately ready for SENTINEL review without further prep work.

## What to Do

Open: `02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Expansion Authorization Trigger Document.md`

Identify all sections that do not require post-May-1 pipeline data. Pre-populate all of them completely. Mark each section's completion status in the document.

## Minimum Sections to Pre-Populate

At minimum, the following must be filled:

**Section: Stage 2 Expansion Scope**
- List all org-IDs pre-staged for Stage 2 (from `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md`)
- List all sources pending Stage 2 activation
- Note any org-IDs still awaiting Presten input (e.g., TYSA if still pending)

**Section: Stage 1 Baseline Criteria (Threshold Definitions)**
- What Week 1 pipeline performance metrics must Stage 1 meet before Stage 2 is authorized?
- Minimum: games-per-day threshold, error rate threshold, pipeline stability requirement (consecutive successful runs)
- Pull from `Infrastructure/Stage 2 Monitoring Rubric Definition.md` if filed; define here if not

**Section: Risk Assessment**
- What are the known risks of Stage 2 expansion?
- What monitoring will FORGE run during Stage 2 expansion week?
- What is the rollback path if Stage 2 expansion causes pipeline instability?

**Section: FORGE Certification (Pre-Stage)**
- Pre-populate all fields FORGE can certify now (config pre-staged, org-IDs reviewed, SENTINEL notification plan in place)
- Leave blank: "Week 1 actuals confirmed [DATE]" and "SENTINEL authorization received [DATE]"

**Section: Data Placeholder for May 4**
Explicitly mark the fields that require post-May-1 data with: `[TO FILL: May 4 — after May 1–3 pipeline runs]`

Fields to leave blank (do NOT guess):
- Total games ingested May 1–3
- Per-source game volume actuals
- Pipeline error rate actuals
- Any metrics that require live pipeline output

## Definition of Done

- All pre-population-eligible sections are fully filled
- All post-May-1 data fields are explicitly marked with `[TO FILL: May 4]`
- Document header shows `status: pre-populated-pending-may4-actuals`
- FORGE confirms pre-population complete in next briefing
- On May 4: FORGE fills actuals and updates status to `ready-for-sentinel-review`

## Why This Matters

SENTINEL's Stage 2 authorization decision is targeted for May 4–5. If the trigger document is not pre-staged, FORGE will need to build the entire document from scratch on May 4 under time pressure. Pre-staging now removes that risk and keeps the Stage 2 authorization on schedule.

## References

- `Infrastructure/Stage 2 Authorization Trigger Document.md` — document to pre-populate
- `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` — org-ID list
- `Infrastructure/Stage 2 Monitoring Rubric Definition.md` — monitoring criteria (if filed)
- `Infrastructure/Stage 2 Authorization Gate Document.md` — prior gate criteria reference
- `Infrastructure/May 1 Per-Source Game Volume Forecast.md` — expected baseline for threshold definitions
