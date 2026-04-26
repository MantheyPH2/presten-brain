---
title: Source Ingestion Baseline — Pre-Launch Snapshot
tags: [infrastructure, pipeline, monitoring, baseline, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: ready — Presten runs at April 29–30 psql session
pipeline-deploy-date: 2026-05-01
---

# Source Ingestion Baseline — Pre-Launch Snapshot

```
Snapshot Date:        ___________  (fill when queries run; target April 29–30)
Run By:               Presten
Pipeline Deploy Date: May 1, 2026
Purpose:              Pre-deployment baseline for first-week pipeline health comparison
```

> Run all queries in a single psql session. Paste output into tables below. After May 1, compare daily against this baseline using `Infrastructure/Source Health Monitoring Queries.md`.

---

## Section 1: Game Count Baseline

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
| | | | | |
| | | | | |
| | | | | |
| | | | | |

*(Add rows as needed. Do not summarize — paste all rows.)*

---

## Section 2: Total Games Baseline

```sql
SELECT COUNT(*) AS total_games FROM games;
```

**Total at snapshot time:** ___________

This is the comparison count. After May 1 pipeline runs, this count must increase. If flat or decreasing after 48 hours of pipeline operation: escalate to SENTINEL immediately.

---

## Section 3: Source Freshness Status

For each source, classify freshness at snapshot time. Use `MAX(game_date)` from Section 1 to calculate days since last game.

**Freshness thresholds** (from `Infrastructure/Source Health Monitoring Queries.md`):
- **FRESH** = last game ≤ 14 days ago
- **STALE** = last game > 14 days ago
- **UNKNOWN** = no games exist for this source

| Source | Days Since Last Game | Status | Notes |
|--------|---------------------|--------|-------|
| gotsport_api | | FRESH / STALE / UNKNOWN | |
| gotsport_html | | FRESH / STALE / UNKNOWN | |
| tgs | | FRESH / STALE / UNKNOWN | |
| tgs_athleteone | | FRESH / STALE / UNKNOWN | |
| tgs_rl | | FRESH / STALE / UNKNOWN | |

*(Add rows for any additional sources visible in Section 1 output.)*

---

## Section 4: Post-April-28 Verification

**Complete this section the day after the April 28 DB session**, using the `April 28 Execution Log` as reference.

| Check | Expected | Actual | Status |
|-------|----------|--------|--------|
| GA ASPIRE games reclassified | `event_tier = 'ga_aspire'` for qualifying events | | ✅ / ❌ |
| ecnl_verified backfill (Step 2b) | `ecnl_verified = TRUE` for TGS/TGS-AthleteOne ECNL games | | ✅ / ❌ |
| Total games count delta (pre vs post April 28) | 0 (only event_tier and ecnl_verified changed; no rows added/removed) | | ✅ / ❌ |

**Verification query for GA ASPIRE reclassification:**

```sql
SELECT COUNT(*) AS ga_aspire_count
FROM games
WHERE event_tier = 'ga_aspire';
```

Expected: > 0 rows. If 0: April 28 GA ASPIRE UPDATE did not apply or did not commit.

**Verification query for ecnl_verified backfill:**

```sql
SELECT COUNT(*) AS verified_count
FROM games
WHERE ecnl_verified = TRUE;
```

Expected: > 0 rows. If 0: Step 2b backfill did not apply.

---

## Section 5: Known Pre-Existing Staleness

List any sources that are **STALE at snapshot time** (from Section 3). These were stale before the pipeline launched — first-week monitoring should not flag them as pipeline failures.

**Pre-existing stale sources at snapshot time:**

- *(fill from Section 3 — list each STALE source and days since last game)*
- ___________

If no sources are STALE: write "None — all sources FRESH at baseline."

---

## Section 6: First-Week Monitoring Thresholds

After May 1, FORGE runs `Infrastructure/Source Health Monitoring Queries.md` daily for the first 7 days. Compare all results against this baseline.

| Signal | Condition | Action |
|--------|-----------|--------|
| **GREEN** | Total game count increases > 0 within 48 hours of first pipeline run | No action. Log in daily briefing. |
| **YELLOW** | No new games after 48 hours; sources that were FRESH at baseline are still FRESH | Monitor. Check crawl log. Report to SENTINEL in next briefing. |
| **RED** | Sources that were FRESH at baseline are now STALE; OR total game count decreased | Escalate to SENTINEL immediately. Do not wait for next briefing cycle. |

**Baseline comparison query** (run daily May 1–7):

```sql
SELECT 
  source,
  COUNT(*) AS current_count,
  MAX(created_at) AS last_ingested
FROM games
GROUP BY source
ORDER BY source;
```

Compare `current_count` to Section 1 baseline counts. Any source showing zero new rows 48 hours after pipeline launch is a YELLOW signal. Any source where `last_ingested` is earlier than the Section 1 baseline is a RED signal.

---

## References

- `[[Source Health Monitoring Queries]]` — daily monitoring queries for first week
- `[[Weekly Pipeline Health Review — Template]]` — the weekly review this baseline feeds (May 8 first review)
- `[[May 1 Deployment — Day-Of Runbook]]` — deploy timeline
- `[[April 28 Execution Log]]` — Section 4 of this document requires April 28 results
- `[[Post-Fix Calibration Stability Monitoring Spec]]` — ELO's monitoring uses the same baseline starting point

*FORGE — 2026-04-26*
