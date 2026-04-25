---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-25
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/May 1 Pipeline Monitoring Queries.md"
---

# Task: May 1 Pipeline Monitoring Queries

## Context

The May 1 Day-Of Runbook (Step 2) tells Presten to "run one command to check game ingestion is happening" and to "tail a log file." The runbook placeholders are appropriately marked `[TBD: confirm after Stage 1 run]` because FORGE does not yet have Stage 1 execution data.

However, the query structure is fully derivable from the schema now — FORGE knows the `games` table, `source_org_id`, `event_tier`, and `inserted_at` columns. FORGE can write the queries today, before Stage 1 runs. The expected numbers can be filled in from Stage 1 results when they arrive.

This task: write the monitoring query pack now. When Stage 1 runs, Presten fills in the threshold numbers from Stage 1 data and updates the document. FORGE does not have to write queries under time pressure on April 30.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/May 1 Pipeline Monitoring Queries.md`

---

### Section 1: Live Ingestion Check (Run During Crawl)

Presten runs this during Step 2 of the Day-Of Runbook — while the crawl is actively running — to confirm games are being inserted.

**Query A — New Games This Run:**
```sql
-- Returns: count of games inserted in the last N minutes
-- Presten runs this every ~15 minutes during the crawl
-- Pass: count is non-zero and increasing
-- Concern: count = 0 for 30+ minutes → check if crawl is hung
SELECT COUNT(*) AS new_games, MAX(inserted_at) AS last_insert
FROM games
WHERE inserted_at > NOW() - INTERVAL '30 minutes';
```

Confirm the exact column name for insertion timestamp (`inserted_at`, `created_at`, or other — pull from the actual schema).

---

### Section 2: Post-Run Health Check (Run After Crawl Completes)

Presten runs this at Step 3 of the Day-Of Runbook after the crawl exits.

**Query B — New Games by Source Org:**
```sql
-- Returns: game count per org_id for today's run
-- Allows FORGE to verify each active org_id returned data
-- Expected: all active Stage 1 org_ids have non-zero counts
SELECT source_org_id, COUNT(*) AS games_today
FROM games
WHERE inserted_at >= CURRENT_DATE
GROUP BY source_org_id
ORDER BY games_today DESC;
```

**Query C — New Games by League/Event Tier:**
```sql
-- Returns: game count per event_tier for today's run
-- Confirms coverage is distributed across expected tiers, not all from one source
SELECT event_tier, COUNT(*) AS games_today
FROM games
WHERE inserted_at >= CURRENT_DATE
GROUP BY event_tier
ORDER BY games_today DESC;
```

**Query D — Total New Games (GREEN/YELLOW/RED Threshold Check):**
```sql
-- Returns: total games ingested today
-- Compare against threshold: [fill from Stage 1 results]
-- GREEN: count >= [threshold from Stage 1]
-- YELLOW: count >= [50% of Stage 1 baseline]
-- RED: count < [50% of Stage 1 baseline] OR count = 0
SELECT COUNT(*) AS total_games_today
FROM games
WHERE inserted_at >= CURRENT_DATE;
```

**Fill after Stage 1:** When Stage 1 TX crawl runs, FORGE updates the threshold values in Query D from the actual Stage 1 results.

---

### Section 3: TX Cohort Spot Check (5 Org-IDs)

For the 5 highest-volume TX org_ids (pull from the full-gotsport-org-id-list or the master reference once Stage 1 runs), verify each returned games:

```sql
-- Replace the org_id list with the actual TX Stage 1 org_ids
SELECT source_org_id, COUNT(*) AS games_today
FROM games
WHERE inserted_at >= CURRENT_DATE
  AND source_org_id IN ([list of TX Stage 1 org_ids])
GROUP BY source_org_id;
```

If any TX org_id returns 0 games on May 1, that is a specific YELLOW/RED signal for FORGE to investigate — not a general health issue.

---

### Section 4: Threshold Placeholder Table

| Metric | Stage 1 Baseline (fill after Stage 1 runs) | GREEN | YELLOW | RED |
|--------|-------------------------------------------|-------|--------|-----|
| Total games ingested | TBD | ≥ TBD | ≥ 50% of baseline | < 50% or 0 |
| Active org_ids returning data | TBD | All | ≥ 80% | < 80% |
| No orgs with 0 games | TBD | All orgs > 0 | 1–2 orgs = 0 | > 2 orgs = 0 |

FORGE fills this table from Stage 1 results when they arrive. Until then, the queries are ready.

---

### Section 5: How to Use

One paragraph: Presten opens this document in a terminal session. Runs Query A periodically during Step 2. Runs Queries B–D at Step 3. Compares Query D total against the threshold table to determine GREEN/YELLOW/RED state for the Day-Of Runbook's Step 4 decision.

---

## Quality Standard

- All queries must reference actual column names from the `games` table schema — no placeholder column names
- Queries A and D are the critical path; B and C are diagnostic. Mark them accordingly
- The threshold table must be marked `[TBD: fill from Stage 1 results]` — do not invent threshold numbers

## Why This Matters

The Day-Of Runbook's Step 2 and Step 3 are currently underspecified for queries. Writing the SQL now (with TBD thresholds) means May 1 only requires Presten to open one file and copy-paste. Writing it after Stage 1 runs is a race condition — Stage 1 may run close to May 1, leaving no time to write and review queries before the deployment session.

## References

- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — Step 2 and Step 3 reference this query pack
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — GREEN/YELLOW/RED thresholds for post-session comparison
- `task-2026-04-25-stage1-backfill-results-report.md` — Stage 1 results that fill the threshold table
