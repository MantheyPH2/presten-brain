---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Source Health Monitoring Queries.md"
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Source Health Monitoring Queries.md"
---

# Task: Source Health Monitoring Queries

## Context

SENTINEL has no standardized tool to check data freshness across Evo Draw's active sources. Detecting a stale source today requires ad hoc SQL. The Source Gap Inventory is a one-time snapshot, not a recurring health check.

This task: write a reusable set of SQL queries that SENTINEL or Presten runs periodically (weekly recommended) to surface stale sources, volume drops, and ingestion failures before they affect ranking quality.

All queries are pure SQL documentation — vault work only, no execution access required to write them.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Infrastructure/Source Health Monitoring Queries.md`

### Section 1: Ingestion Recency Check

```sql
-- Last ingestion date per source — run weekly
SELECT source_org_id, source_name,
       MAX(created_at) AS last_ingested,
       NOW() - MAX(created_at) AS time_since_ingestion
FROM games
GROUP BY source_org_id, source_name
ORDER BY time_since_ingestion DESC;
```

Threshold definitions:
- `< 7 days` — Active. No action.
- `7–30 days` — Stale. Flag for SENTINEL review. FORGE checks crawl logs for errors.
- `> 30 days` — Critical. Escalate to Presten. Source may have broken or gone offline.

### Section 2: Weekly Game Volume Check

```sql
-- Games ingested in last 7 days per source
SELECT source_org_id, source_name,
       COUNT(*) AS games_last_7_days
FROM games
WHERE created_at >= NOW() - INTERVAL '7 days'
GROUP BY source_org_id, source_name
ORDER BY games_last_7_days ASC;
```

Expected ranges by source type (during season, September–May):
- ECNL: 50+ games/week
- GA / GA ASPIRE: 30+ games/week
- State cup leagues: near 0 outside state cup event windows — not an alert
- Off-season (June–August): all sources may show near 0 — not an alert

Flag: any in-season source showing 0 games for 2+ consecutive weekly checks.

### Section 3: Duplicate Detection

```sql
-- Run after any new ingestion batch
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*) AS count
FROM games
WHERE created_at >= NOW() - INTERVAL '7 days'
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1
ORDER BY count DESC;
```

Expected: 0 rows. If duplicates appear: log the org_id of the source in the FORGE briefing and flag to SENTINEL. Do not run calibration recomputes until duplicates are resolved.

### Section 4: New Source Coverage Confirmation

```sql
-- Confirm recently added org_ids are producing games
-- Update [NEW_ORG_IDS] after each org-ID expansion session
SELECT source_org_id, COUNT(*) AS total_games,
       MAX(created_at) AS most_recent_game,
       MIN(game_date) AS earliest_game_date
FROM games
WHERE source_org_id IN ([NEW_ORG_IDS])
GROUP BY source_org_id
ORDER BY total_games DESC;
```

Instructions: replace `[NEW_ORG_IDS]` with the actual org IDs added in the most recent expansion session. Run this query 24 hours after the expansion crawl to confirm new sources are active. Expected: game count > 0 for each new org_id within 48 hours of addition.

After TX cohort expansion: insert TX cohort org IDs here. After USL Academy / EDP addition: update again.

### Section 5: Source Count Baseline

```sql
-- Full source inventory with game counts and recency
SELECT source_name, source_org_id,
       COUNT(*) AS total_games,
       MIN(game_date) AS earliest_game,
       MAX(game_date) AS most_recent_game,
       NOW() - MAX(created_at) AS staleness
FROM games
GROUP BY source_name, source_org_id
ORDER BY total_games DESC;
```

Run monthly or after any major expansion. Compare to the previous run. Flag any source where:
- `total_games` has not increased since last month AND
- `most_recent_game` is more than 14 days ago AND
- It is currently in-season for that source

### Section 6: Run Schedule

| Query | Frequency | Who Runs | Notes |
|-------|-----------|----------|-------|
| Ingestion Recency (§1) | Weekly | Presten (psql) | Monday morning recommended |
| Weekly Volume (§2) | Weekly | Presten (psql) | Same session as §1 |
| Duplicate Detection (§3) | After each ingestion batch | FORGE (post-crawl) | Automated when pipeline runs daily |
| New Source Coverage (§4) | After each org-ID expansion | Presten (psql) | Run 24h post-expansion |
| Source Count Baseline (§5) | Monthly | SENTINEL review session | Update thresholds as baselines stabilize |

## Definition of Done

- Document written at `Infrastructure/Source Health Monitoring Queries.md`
- All five query sections present, copy-paste ready
- Threshold definitions included for each section
- `[NEW_ORG_IDS]` placeholder in Section 4 documented with instructions for how to fill it
- Run schedule table included
- Summary in briefing: "Source health monitoring queries written. 5 queries covering ingestion recency, volume, duplicates, new source confirmation, and monthly baseline. Run schedule defined."

## Why This Matters

SENTINEL's core responsibility is product completeness — which requires detecting when sources break. These queries convert ad hoc troubleshooting into a repeatable 5-minute weekly check. When Presten runs the daily pipeline (May 1 deadline), source health monitoring becomes part of the operational cadence, not a reaction to problems.

## References

- `Infrastructure/Source Gap Inventory — April 2026.md` — source list this monitoring covers
- `Infrastructure/Daily Pipeline Log Format Spec.md` — pipeline logging that feeds duplicate detection
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — validation pattern this extends to ongoing ops
