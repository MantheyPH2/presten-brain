---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md"
---

# Task: Source Ingestion Baseline — Pre-Launch Snapshot

## Context

The daily pipeline deploys May 1. Starting May 1, the pipeline runs every day and ingests new games from all sources. FORGE has monitoring queries (`Infrastructure/Source Health Monitoring Queries.md`) and a weekly health review template (`Infrastructure/Weekly Pipeline Health Review — Template.md`). What does not exist: a pre-deployment baseline document that records source health state *before* the pipeline goes live.

Without a baseline, FORGE cannot answer the first-week question: "Is the pipeline working, or was this source already stale before we launched?" If GotSport has 0 new games on May 2, FORGE cannot tell whether that is a pipeline failure or a scheduling gap. The baseline provides the comparison point.

This document must be written before May 1 — ideally April 29-30, after the April 28 DB session completes and the post-fix data state is stable.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md`

---

### Header

```
Snapshot Date: ___________  (fill when queries run; target April 29–30)
Run By: Presten (or FORGE if DB access available)
Pipeline Deploy Date: May 1, 2026
Purpose: Pre-deployment baseline for first-week pipeline health comparison
```

---

### Section 1: Game Count Baseline

```sql
SELECT 
  source,
  event_tier,
  COUNT(*) AS game_count,
  MAX(game_date) AS most_recent_game,
  MAX(created_at) AS last_ingested
FROM games
GROUP BY source, event_tier
ORDER BY source, event_tier;
```

**Paste full output below.** This is the reference state before any pipeline-generated rows exist.

| Source | Event Tier | Game Count | Most Recent Game | Last Ingested |
|--------|------------|------------|-----------------|---------------|
| | | | | |

---

### Section 2: Total Games Baseline

```sql
SELECT COUNT(*) AS total_games FROM games;
```

Total at snapshot time: ___________

This is the baseline count. After May 1 pipeline runs, the count should increase. If the count is flat or decreasing after 48 hours of pipeline operation, escalate to SENTINEL.

---

### Section 3: Source Freshness Status

For each source, classify freshness at snapshot time:

| Source | Days Since Last Game | Status | Notes |
|--------|---------------------|--------|-------|
| gotsport_api | | FRESH / STALE / UNKNOWN | |
| gotsport_html | | FRESH / STALE / UNKNOWN | |
| tgs | | FRESH / STALE / UNKNOWN | |
| tgs_athleteone | | FRESH / STALE / UNKNOWN | |
| tgs_rl | | FRESH / STALE / UNKNOWN | |

Status thresholds (from Source Health Monitoring Queries): FRESH = last game ≤ 14 days. STALE = > 14 days. UNKNOWN = no games for this source.

---

### Section 4: Post-April-28 Verification

This section is added after the April 28 DB session completes. Confirm the following were applied:

| Check | Expected | Actual | Notes |
|-------|----------|--------|-------|
| GA ASPIRE games reclassified | `event_tier = 'ga_aspire'` for qualifying games | | |
| ecnl_verified backfill (Step 2b) | `ecnl_verified = TRUE` for TGS/TGS-AthleteOne ECNL games | | |
| Total games count delta (pre vs post April 28) | 0 (only event_tier and ecnl_verified changed) | | |

---

### Section 5: Known Pre-Existing Staleness

Any sources that are STALE at snapshot time were stale *before* the pipeline launched. FORGE lists them here so first-week monitoring does not flag them as pipeline failures.

Pre-existing stale sources (at snapshot time):
- ___________

---

### Section 6: First-Week Monitoring Thresholds

After May 1, FORGE runs `Infrastructure/Source Health Monitoring Queries.md` daily for the first 7 days. Compare against this baseline.

**Green:** Total game count increases by > 0 within 48 hours of first pipeline run.
**Yellow:** No new games after 48 hours; sources that were FRESH at snapshot are still FRESH.
**Red:** Sources that were FRESH at snapshot are now STALE; OR total game count decreased.

Red triggers immediate SENTINEL escalation.

---

## Quality Standard

- Section 1 query must be copy-paste ready
- Section 3 freshness status must be filled in (not left blank) at the time of snapshot
- Section 4 must be updated the day after April 28, not left empty
- Section 5 must explicitly list any pre-existing stale sources so first-week monitoring has clear context

## Why This Matters

The May 8 first weekly health review (per `Infrastructure/Weekly Pipeline Health Review — Template.md`) needs a comparison baseline. Without it, FORGE cannot distinguish "pipeline failed to ingest" from "no games were scheduled this week." The pre-launch snapshot also documents the post-April-28 data state as the authoritative starting point for all first-month pipeline health analysis — including ELO's calibration stability monitoring.

## References

- `Infrastructure/Source Health Monitoring Queries.md` — daily monitoring queries for first week
- `Infrastructure/Weekly Pipeline Health Review — Template.md` — the weekly review this baseline feeds
- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — deploy timeline context
- `Infrastructure/April 28 Execution Log.md` — Section 4 requires April 28 results
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — ELO's monitoring uses same baseline assumption
