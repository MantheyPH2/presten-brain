---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: completed
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/May 1 — Per-Source Game Volume Forecast.md"
---

# Task: May 1 Per-Source Game Volume Forecast

## Context

The May 1 Day-Of Runbook has a "Section: Expected First-Run Numbers" that currently reads: "Expected game count: pending Stage 1 results." The Stage 1 TX crawl has not yet run. But SENTINEL cannot enter May 1 with no volume expectations — if the first pipeline run produces 0 games, SENTINEL needs to know whether that is catastrophically bad or plausibly explained by source lag.

The Morning-After Runbook has aggregate GREEN/YELLOW/RED thresholds. What does not exist: per-source expected volumes. If the pipeline ingests 8,000 games total but all from one source and zero from three others, that is a different situation than 8,000 evenly distributed. An aggregate pass can mask individual source failures.

FORGE has the knowledge needed to estimate per-source volumes before May 1 runs:
- Current active org-ID set (from the Master Reference and Org-ID List)
- Historical game frequency patterns (from the Source Ingestion Baseline docs)
- Known season-window context (April = mid-season for most leagues)

This document fills the May 1 Runbook placeholder and gives SENTINEL and Presten a source-level diagnostic framework for the first-run validation.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/May 1 — Per-Source Game Volume Forecast.md`

---

### Header

```
Written by: FORGE
Date: 2026-04-29 (or earlier if Stage 1 results are available)
Coverage: Active org-IDs confirmed in crawl config as of this writing
Excludes: Stage 1 TX (TYSA) — pending browser lookup; Stage 2 entities — not yet in config
Note: These are pre-crawl estimates. Actual results may vary based on season timing and GotSport data availability.
```

---

### Section 1: Active Source Summary

List all org-IDs currently confirmed in the active crawl config (pull from Master Reference Section 1). For each:

| Org-ID | Entity Name | Type | Est. Games in DB | Est. New Games on May 1 | Confidence |
|--------|-------------|------|-----------------|------------------------|-----------|

**Confidence levels:**
- High: Entity has prior crawl history in DB; game frequency is known
- Medium: Entity is confirmed on GotSport with expected game volume but no prior crawl
- Low: Org-ID confirmed, but league size/activity is uncertain

---

### Section 2: Per-Source GREEN/YELLOW/RED Thresholds

For each active source, define what constitutes a healthy first run:

| Entity | GREEN (healthy) | YELLOW (investigate) | RED (likely failed) |
|--------|----------------|---------------------|-------------------|

Example logic: 
- GREEN: new game count ≥ 50% of estimated volume  
- YELLOW: new game count = 10–49% of estimated volume (partial crawl, not necessarily broken)  
- RED: 0–9% of estimated volume (crawl likely failed for this source)

FORGE sets these thresholds based on its knowledge of each source. Do not use a single uniform threshold — different sources have different expected volume ranges.

---

### Section 3: Aggregate Totals

Sum the per-source estimates into an aggregate forecast:

```
Total estimated new games (May 1 first run):
  Conservative estimate (70% of expected): [N]
  Expected estimate (100%): [N]
  Optimistic estimate (120%): [N]

GREEN floor (aggregate): [N] games (maps to Morning-After Runbook GREEN threshold)
RED ceiling (aggregate): [N] games (if below this, run FAILED)
```

Note: If Stage 1 TX results arrive before this document is filed, incorporate Stage 1 volume (TYSA expected count from `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md`) into the aggregate.

---

### Section 4: Source Failure Diagnostic Tree

If the first run is aggregate YELLOW (partial), this tree guides Presten toward the failing source:

```
Aggregate YELLOW — Diagnostic Steps:
1. Run per-source counts (query in Pipeline Validation Spec)
2. Identify which sources are individually RED
3. Check log output for those sources: connection error? auth failure? rate limit?
4. Escalate RED sources to FORGE briefing; GREEN sources confirm partial success
5. File per-source breakdown in May 1 monitoring log
```

---

### Section 5: Stage 1 Placeholder

If Stage 1 TX (TYSA) results are available before May 1:
- Incorporate TYSA into Section 1 with actual org-ID and observed game count from the Stage 1 run
- Update Section 3 aggregate totals accordingly
- Note: "Stage 1 results incorporated from `Infrastructure/Stage 1 Backfill Results Report.md`"

If Stage 1 has not run by the time this document is filed:
- Leave TYSA as: "TYSA (TYSA org-ID pending browser lookup) — not included in May 1 forecast. If org-ID is confirmed and config staged before May 1, add [N] games to aggregate per Stage 1 Output Spec GREEN floor."

---

## Quality Standard

- Section 1 must list every active org-ID from the Master Reference. Do not skip an org-ID because the estimate is uncertain — write the estimate with Low confidence.
- Section 2 thresholds must be numeric. "Some games" is not a threshold.
- Section 4 diagnostic tree must be executable by Presten without FORGE present.
- File by April 29 at the latest — this document must be available before May 1 begins.

## Why This Matters

The Morning-After Runbook GREEN threshold covers aggregate volume. SENTINEL's May 1 review covers aggregate health. But individual source failures can hide inside aggregate GREEN. If SENTINEL sees "8,000 games — GREEN" and reports it to Presten as a successful first run, but three sources produced 0 games because their org-IDs were misconfigured, the pipeline appears healthy until the next review. Per-source thresholds surface that failure immediately.

May 1 is Stage 3 of the crawl expansion. Getting Stage 3 health assessment right on day one prevents 1–2 weeks of incorrect monitoring.

## References

- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — active org-ID list for Section 1
- `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` — Stage 1 TYSA volume estimates for Section 5
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — aggregate GREEN threshold this doc supplements
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — per-source query patterns
- `Infrastructure/Source Ingestion Baseline — April 2026.md` — historical volume context
