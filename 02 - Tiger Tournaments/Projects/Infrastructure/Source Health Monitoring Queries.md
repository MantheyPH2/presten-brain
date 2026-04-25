---
type: reference
author: FORGE
date: 2026-04-24
status: ready
applies_to: All active Evo Draw data sources
run_schedule: See Section 6
---

# Source Health Monitoring Queries

Reusable SQL queries for detecting stale sources, volume drops, and ingestion failures before they affect ranking quality. Run on the `youth_soccer` database at `localhost:5432` (user `p`).

---

## Section 1: Ingestion Recency Check

```sql
-- Last ingestion date per source — run weekly
SELECT source_org_id, source_name,
       MAX(created_at) AS last_ingested,
       NOW() - MAX(created_at) AS time_since_ingestion
FROM games
GROUP BY source_org_id, source_name
ORDER BY time_since_ingestion DESC;
```

**Threshold definitions:**
- `< 7 days` — **Active.** No action.
- `7–30 days` — **Stale.** Flag for SENTINEL review. FORGE checks crawl logs for errors.
- `> 30 days` — **Critical.** Escalate to Presten. Source may have broken or gone offline.

---

## Section 2: Weekly Game Volume Check

```sql
-- Games ingested in last 7 days per source
SELECT source_org_id, source_name,
       COUNT(*) AS games_last_7_days
FROM games
WHERE created_at >= NOW() - INTERVAL '7 days'
GROUP BY source_org_id, source_name
ORDER BY games_last_7_days ASC;
```

**Expected ranges by source type (during season, September–May):**
- ECNL: 50+ games/week
- GA / GA ASPIRE: 30+ games/week
- State cup leagues: near 0 outside state cup event windows — not an alert
- Off-season (June–August): all sources may show near 0 — not an alert

**Flag:** Any in-season source showing 0 games for 2+ consecutive weekly checks.

---

## Section 3: Duplicate Detection

```sql
-- Run after any new ingestion batch
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*) AS count
FROM games
WHERE created_at >= NOW() - INTERVAL '7 days'
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1
ORDER BY count DESC;
```

**Expected:** 0 rows.
**If duplicates appear:** Log the `org_id` of the source in the FORGE briefing and flag to SENTINEL. Do not run calibration recomputes until duplicates are resolved.

---

## Section 4: New Source Coverage Confirmation

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

**Instructions:** Replace `[NEW_ORG_IDS]` with the actual org IDs added in the most recent expansion session. Run 24 hours after the expansion crawl to confirm new sources are active.
**Expected:** `game count > 0` for each new `org_id` within 48 hours of addition.

**Update log:**
- After TX cohort expansion: insert TX cohort org IDs here.
- After USL Academy / EDP addition: update again.
- After each new org_id batch: update and re-run.

---

## Section 5: Source Count Baseline

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

**Run:** Monthly or after any major expansion. Compare to previous run.
**Flag any source where ALL of the following are true:**
- `total_games` has not increased since last month, AND
- `most_recent_game` is more than 14 days ago, AND
- It is currently in-season for that source.

---

## Section 6: Run Schedule

| Query | Frequency | Who Runs | Notes |
|-------|-----------|----------|-------|
| Ingestion Recency (§1) | Weekly | Presten (psql) | Monday morning recommended |
| Weekly Volume (§2) | Weekly | Presten (psql) | Same session as §1 |
| Duplicate Detection (§3) | After each ingestion batch | FORGE (post-crawl) | Automated when pipeline runs daily |
| New Source Coverage (§4) | After each org-ID expansion | Presten (psql) | Run 24h post-expansion |
| Source Count Baseline (§5) | Monthly | SENTINEL review session | Update thresholds as baselines stabilize |

---

## References

- `Infrastructure/Source Gap Inventory — April 2026.md` — source list this monitoring covers
- `Infrastructure/Daily Pipeline Log Format Spec.md` — pipeline logging that feeds duplicate detection
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — validation pattern this extends to ongoing ops
