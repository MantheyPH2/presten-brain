---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: medium
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Weekly Pipeline Health Review — Template.md"
---

# Task: Weekly Pipeline Health Review Template

## Context

The pipeline launch infrastructure is now comprehensive at the session level:
- **Morning-After Runbook** — Day 1 post-deployment health check
- **Daily Pipeline Log Spec** — Per-run output format
- **Source Freshness Audit Template** — Per-session source recency check
- **Deployment Validation Spec** — One-time deployment verification

What does not exist: a weekly health review cadence. After May 1, the pipeline runs daily. SENTINEL reviews FORGE briefings daily, but daily briefings will naturally focus on the most recent run's results. A broader view — 7-day game count trends, source drift over time, Stage 2 expansion progress — requires a weekly summary that no daily briefing will naturally produce.

Without a weekly cadence, calibration and coverage regressions can go unnoticed for 5–6 days before SENTINEL detects them. With one, SENTINEL has a structured weekly checkpoint to catch slow deterioration before it reaches DSS.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Weekly Pipeline Health Review — Template.md`

This is a reusable template. FORGE fills it out and files it to a new file each week: `Infrastructure/Weekly Pipeline Health Review — [YYYY-MM-DD].md`. First review due May 8 (7 days post-May-1 deployment). Cadence: every Friday or the first pipeline session of the week.

---

### Header (FORGE fills each week)

```
Week ending: [YYYY-MM-DD]
Review number: [1, 2, 3...]
FORGE filing date: [YYYY-MM-DD]
Pipeline deployment date: 2026-05-01
Days since deployment: [N]
```

---

### Section 1: 7-Day Game Count Summary

| Source | Games Ingested (7d) | Avg per Run | Prior Week Avg | Delta | Status |
|--------|--------------------|--------------|--------------|----|--------|
| ECNL Girls | | | | | |
| ECNL Boys | | | | | |
| MLS NEXT | | | | | |
| GA Girls | | | | | |
| GA Boys | | | | | |
| GA ASPIRE | | | | | |
| ECNL RL Girls | | | | | |
| ECNL RL Boys | | | | | |
| NPL | | | | | |
| DPL | | | | | |
| TGS | | | | | |
| **Stage 2 sources** | | | | | |
| **TOTAL** | | | | | |

Status options: `GREEN` (within 10% of prior week) / `YELLOW` (10–25% drop) / `RED` (>25% drop or 0) / `OFFSEASON` (expected 0)

FORGE writes the SQL to generate this table (one query against `games.created_at` grouped by `source_name` filtered to the last 7 days). Include the query in Section 6 so it is copy-paste ready each week.

---

### Section 2: Source Freshness Status

Quick status for each active source: is the most recent game date within the expected freshness window?

| Source | Most Recent Game Date | Days Since Last Ingest | Freshness Status |
|--------|----------------------|----------------------|-----------------|
| | | | |

Pull from the Source Freshness Audit Template results if a freshness audit was run this week. If not, run Query 1 from that template and record results here.

Freshness Status: `FRESH` / `STALE-WARNING` / `STALE-CRITICAL` per the Source Freshness Audit thresholds.

---

### Section 3: Stage 2 Expansion Progress

| Stage 2 Gate Condition | Status | Date Cleared |
|------------------------|--------|-------------|
| C1 — Stage 1 crawl results confirmed | | |
| C2 — Stage 1 game volume validated | | |
| C3 — USL Academy org-IDs confirmed | | |
| C4 — ecnl_verified column live | | |
| C5 — EDP org-IDs confirmed | | |
| C6 — Baseline snapshot complete | | |
| **SENTINEL authorization** | | |

Stage 2 expansion started: [date, once applicable]
Stage 2 new sources active: [list, once applicable]

---

### Section 4: Error and Anomaly Log

Any errors or unexpected behavior observed in the past 7 days:

| Date | Source | Error Description | Resolution | Status |
|------|--------|------------------|------------|--------|
| | | | | |

If no errors in the 7-day window: "No errors this week."

Include any YELLOW or RED rows from the GotSport Org-ID Inactive Detection Protocol that were triggered this week.

---

### Section 5: SENTINEL Weekly Flag Summary

Three-line SENTINEL summary FORGE writes at the end of each review:

```
OVERALL PIPELINE STATUS: GREEN / YELLOW / RED
REASON (if not GREEN): [one sentence]
ACTION REQUIRED FROM SENTINEL: YES / NO — [if YES, what]
```

GREEN: All sources FRESH, no RED game count drops, no unresolved errors.
YELLOW: One or more STALE-WARNING sources or one YELLOW game count drop. SENTINEL informed; no immediate action.
RED: Any STALE-CRITICAL, RED game count drop, or unresolved error from prior week. SENTINEL action required.

---

### Section 6: Standing SQL Queries

Three queries FORGE runs to generate Sections 1–2 each week. Write these once in the template; copy-paste into psql each week.

**Query A — 7-Day Game Count by Source**
```sql
-- FORGE writes the actual query
-- Returns: source_name, game_count_7d, avg_per_run
-- Filter: created_at >= NOW() - INTERVAL '7 days'
```

**Query B — Most Recent Game Date per Source**
```sql
-- FORGE writes the actual query
-- Returns: source_name, max(game_date), max(created_at)
-- Used for freshness status
```

**Query C — 7-Day vs Prior-7-Day Delta**
```sql
-- FORGE writes the actual query
-- Returns: source_name, count_current_7d, count_prior_7d, pct_delta
-- Used for "Delta" column in Section 1
```

Column names must use confirmed schema: `source_name` (or `source_org_id`), `game_date`, `created_at`.

---

## Cadence

| Review # | Week Ending | Due |
|----------|------------|-----|
| 1 | 2026-05-08 | 2026-05-08 |
| 2 | 2026-05-15 | 2026-05-15 |
| 3 | 2026-05-22 | 2026-05-22 |

FORGE adds future weeks to this table as the pipeline runs. SENTINEL adds this review to its weekly briefing as the canonical pipeline health source.

---

## Quality Standard

- All active sources must appear in Section 1 at each filing. Never omit a source because it was OFFSEASON — include it with status OFFSEASON.
- The three SQL queries must be complete and copy-paste ready at template creation time. FORGE tests each query on the live database before filing the template.
- The Section 5 SENTINEL flag must be the last thing FORGE writes, after completing all other sections. It is a synthesis, not a starting point.
- "GREEN overall with one YELLOW source" = YELLOW overall, not GREEN. Be honest about the status.

## Why This Matters

Daily briefings track individual runs. Weekly reviews track the system. A source that slowly drops from 120 games/day to 85 games/day over 6 days would appear as a series of "slightly lower than yesterday" daily notes — each individually unremarkable. The 7-day view shows a 29% drop: YELLOW trending to RED. SENTINEL cannot detect that pattern from daily briefings alone. The weekly review is the instrument that catches slow deterioration.

## References

- `Infrastructure/Daily Pipeline Log Spec.md` — per-run output this weekly review summarizes
- `Infrastructure/Source Freshness Audit Template.md` — freshness data for Section 2
- `Infrastructure/GotSport Org-ID Inactive Detection Protocol.md` — anomaly inputs for Section 4
- `Infrastructure/Stage 2 Expansion — Authorization Gate Document.md` — gate conditions for Section 3
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — Day 1 baseline that Section 1's "Prior Week Avg" is compared against
