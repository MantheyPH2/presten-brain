---
title: May 1 Pipeline Monitoring Queries
tags: [infrastructure, pipeline, monitoring, gotsport, sql, evo-draw, forge]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: ready — thresholds pending Stage 1 results
task: task-2026-04-25-may1-pipeline-monitoring-queries
---

# May 1 Pipeline Monitoring Queries

> [!info] How to use
> Open this document in a terminal session alongside the [[May 1 Deployment — Day-Of Runbook]]. Run Query A periodically during Step 2 (while the crawl is running). Run Queries B–D at Step 3 (after the crawl exits). Compare Query D total against the threshold table to determine GREEN/YELLOW/RED state.

> [!warning] Thresholds pending Stage 1
> Query D's GREEN/YELLOW/RED threshold values are marked `[TBD: fill from Stage 1 results]`. When the TX Stage 1 crawl runs, FORGE updates this document immediately from the `gotsport-org-expansion-stage1-results-[date].md` report. Until then, all queries are structurally complete and copy-paste ready.

---

## Section 1: Live Ingestion Check (Run During Crawl)

**Query A — New Games This Run** ⭐ *Critical path*

Run every ~15 minutes while `run-daily.sh` is executing. Confirms games are being inserted and ingestion is progressing.

```sql
SELECT 
  COUNT(*) AS new_games,
  MAX(created_at) AS last_insert
FROM games
WHERE created_at > NOW() - INTERVAL '30 minutes';
```

**Pass:** `new_games` is non-zero and increasing across successive checks.  
**Concern:** `new_games = 0` for 30+ consecutive minutes → the crawl may be hung on a rate limit or network issue. Check the crawl log.  
**Column note:** Uses `created_at` — confirmed in Source Coverage Priority Matrix and Source Gap Inventory monitoring queries. If the production DB uses `inserted_at` instead, update the column name before running.

---

## Section 2: Post-Run Health Check (Run After Crawl Completes)

Run these after `run-daily.sh` exits. Confirms distribution, coverage, and total volume.

**Query B — New Games by Source Org** *(Diagnostic)*

```sql
SELECT 
  source,
  COUNT(*) AS games_today
FROM games
WHERE created_at >= CURRENT_DATE
GROUP BY source
ORDER BY games_today DESC;
```

**Use:** Confirms which sources are contributing games. All active sources should show non-zero counts. If `gotsport` shows 0: the crawl config may not be pointing to active org IDs.

**Query C — New Games by Event Tier** *(Diagnostic)*

```sql
SELECT 
  event_tier,
  COUNT(*) AS games_today
FROM games
WHERE created_at >= CURRENT_DATE
  AND is_hidden = false
GROUP BY event_tier
ORDER BY games_today DESC;
```

**Use:** Confirms coverage across expected tiers. A healthy run distributes games across multiple tiers. A single tier dominating (e.g., 95% from one event_tier) is a yellow signal worth investigating.

**Query D — Total New Games** ⭐ *Critical path — GREEN/YELLOW/RED gate*

```sql
SELECT COUNT(*) AS total_games_today
FROM games
WHERE created_at >= CURRENT_DATE
  AND is_hidden = false;
```

**Decision (fill thresholds from Stage 1 results):**

| Status | Threshold | Action |
|--------|-----------|--------|
| **GREEN** | `total_games_today >= [TBD: Stage 1 baseline]` | Deployment confirmed successful. File closure note to FORGE briefing. |
| **YELLOW** | `total_games_today >= [TBD: 50% of Stage 1 baseline]` | Pipeline ran but underperformed. Do NOT escalate immediately. Run Query B and C to identify which source/tier is low. Contact FORGE with findings. |
| **RED** | `total_games_today < [TBD: 50% of Stage 1 baseline]` OR `= 0` | Pipeline failed or produced no output. Execute RED-state remediation from Day-Of Runbook. Do NOT proceed to Stage 2. Contact SENTINEL. |

---

## Section 3: TX Cohort Spot Check

For the 5 highest-volume TX org IDs (fill from `Infrastructure/full-gotsport-org-id-list.md` after Stage 1 runs), verify each returned games on May 1:

```sql
SELECT 
  source,
  COUNT(*) AS games_today
FROM games
WHERE created_at >= CURRENT_DATE
  AND source_id IN (
    -- Replace with actual TX Stage 1 org IDs from Stage 1 Results Report
    '[TX_ORG_ID_1]',
    '[TX_ORG_ID_2]',
    '[TX_ORG_ID_3]',
    '[TX_ORG_ID_4]',
    '[TX_ORG_ID_5]'
  )
GROUP BY source
ORDER BY games_today DESC;
```

> [!note] Placeholder values
> Replace `[TX_ORG_ID_N]` with actual org IDs from the Stage 1 Results Report once Stage 1 executes. If any TX org ID returns 0 games on May 1, this is a **specific YELLOW/RED signal for that org** — not a general health issue. Log it and forward to FORGE with the org ID value.

---

## Section 4: Threshold Placeholder Table

FORGE fills this table from Stage 1 results when they arrive. Until then, the queries are structurally complete.

| Metric | Stage 1 Baseline | GREEN | YELLOW | RED |
|--------|-----------------|-------|--------|-----|
| Total games ingested | `[TBD after Stage 1]` | ≥ baseline | ≥ 50% of baseline | < 50% of baseline, or 0 |
| Active org IDs returning data | `[TBD after Stage 1]` | All active orgs > 0 | ≥ 80% of orgs > 0 | < 80% of orgs, or 0 |
| No orgs with 0-game result | `[TBD after Stage 1]` | All orgs > 0 | 1–2 orgs = 0 | > 2 orgs = 0 |

**When Stage 1 runs:** FORGE opens this document and populates the Stage 1 Baseline column from `Reports/gotsport-org-expansion-stage1-results-[date].md` Section A (Game Ingestion Summary). GREEN and YELLOW thresholds are computed from that baseline.

---

## Section 5: How to Use

1. **Open this document** in a terminal or second tab before starting the May 1 deployment.
2. **During Step 2** of the Day-Of Runbook (while crawl is running): copy-paste Query A into `psql -d youth_soccer`. Run it every ~15 minutes. Confirm `new_games` is non-zero and rising.
3. **At Step 3** of the Day-Of Runbook (after crawl exits): run Queries B, C, and D in sequence.
4. **Compare Query D result** to the Threshold Table (Section 4) to determine GREEN/YELLOW/RED state.
5. **Report to FORGE** the Query D total and the GREEN/YELLOW/RED determination. FORGE updates the Stage 1 Results Report and the May 1 Deployment records.

---

## Related

- [[May 1 Deployment — Day-Of Runbook]] — references these queries in Step 2 and Step 3
- [[Daily Pipeline Morning-After Validation Runbook]] — uses similar thresholds for May 2 morning check
- `Infrastructure/gotsport-org-expansion-stage1-results-[date].md` — fills Section 4 thresholds when available

*FORGE — 2026-04-25*
