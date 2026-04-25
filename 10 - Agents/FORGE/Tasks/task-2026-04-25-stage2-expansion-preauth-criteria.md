---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: medium
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md"
---

# Task: GotSport Org-ID Expansion — Stage 2 Pre-Authorization Criteria

## Context

Stage 1 (TX org_id test crawl) gates Stage 2 (full U.S. org-ID list). Stage 1 is pending Presten execution. When it runs, FORGE writes the Stage 1 Results Report — which either authorizes or blocks Stage 2.

What does not exist: the explicit Stage 2 authorization criteria. Without them, SENTINEL must decide in the moment whether Stage 1 results are sufficient to proceed. That decision should be pre-defined.

This task: write the Stage 2 authorization gate document now, before Stage 1 runs. All content is derivable from existing specs — no execution required.

## What to Do

### Section 1: Stage 2 Scope

Pull from `Infrastructure/full-gotsport-org-id-list.md` and `Infrastructure/Stale Team Backfill Runbook — April 2026.md`:

- Total org IDs in Stage 2 cohort
- Estimated game volume (range: min, expected, max)
- Estimated crawl time at Option B backoff rates
- Geographic coverage (which states/regions Stage 2 adds beyond TX)

If any estimate is missing from the referenced docs, note the uncertainty range.

### Section 2: Stage 2 Authorization Gate

List every condition that must be true from Stage 1 before Stage 2 runs. Pull from the Runbook and Validation Spec. For each condition:

| Condition | Source Doc | Pass Criteria | Fail Action |
|-----------|------------|---------------|-------------|

At minimum, include:
- Stage 1 game count for new org_ids stabilized (second crawl delta = 0)
- Duplicate check result: 0 duplicate rows from the validation query
- No org_ids returned errors in Stage 1
- Option B backoff performed as designed (rate limit not hit during Stage 1)
- FORGE issued explicit Stage 2 authorization in the Stage 1 Results Report

For each fail action: state the specific remediation step so the path is unambiguous. Do not write "investigate" — write the actual corrective step.

### Section 3: Stage 2 Rollback Plan

If Stage 2 encounters issues mid-run:
- How to halt the crawl without corrupting data already ingested
- Whether partial Stage 2 ingestion is acceptable or must be rolled back completely
- How to detect if duplicates were created mid-run (reference the duplicate detection query from Source Health Monitoring Queries)
- How to remove those duplicates if found

### Section 4: SENTINEL Authorization Protocol

SENTINEL (not FORGE) authorizes Stage 2. Document:
- What SENTINEL reviews in the Stage 1 Results Report to make the call — specific sections, specific pass/fail values
- What authorization looks like: a note in SENTINEL's briefing following the Stage 1 Results Report review
- Timeline: how quickly after Stage 1 runs should Stage 2 authorization happen? Recommendation: same session if all gate conditions pass; next session if any condition requires follow-up

## Definition of Done

- Document written at `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md`
- All 4 sections present
- Authorization gate conditions are explicit and measurable (no "approximately correct" or "looks reasonable")
- Each fail condition has a specific remediation step
- Rollback plan addresses partial ingestion
- SENTINEL authorization protocol is defined
- Summary in briefing: "Stage 2 pre-authorization criteria written. Gate conditions: [N] required. Rollback plan covers [scenarios]."

## Why This Matters

The TX org-ID cohort alone could add thousands of previously uncaptured games — Stage 2 adds the remainder of the U.S. But a bad Stage 2 run (duplicates, rate limit hits, partial ingestion) is harder to clean up than a bad Stage 1. Defining authorization criteria before Stage 1 runs means SENTINEL can review the Results Report against specific criteria and authorize immediately — or hold off with a specific reason. No improvised decisions under time pressure.

## References

- `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — Stage exit conditions
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — validation query patterns
- `Infrastructure/full-gotsport-org-id-list.md` — Stage 2 org-ID scope
- `Infrastructure/Source Health Monitoring Queries.md` — duplicate detection query
- `task-2026-04-25-stage1-backfill-results-report.md` — the Results Report this doc gates
