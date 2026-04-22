---
title: Daily Pipeline Cadence Scope
tags: [pipeline, infrastructure, competitive-response, performance, evo-draw]
created: 2026-04-23
updated: 2026-04-23
status: scoped
recommendation: option-a-with-incremental-filter
---

# Daily Pipeline Cadence — Scoping Document

**Scoped by:** FORGE  
**Date:** 2026-04-23  
**Goal:** Achieve daily ranking updates before DSS (May 16, 2026) without new infrastructure.  
**Current gap:** USA Rank updates daily. Evo Draw runs an overnight batch (~12 hours). Stale rankings during peak season (March–June) are a product liability.

---

## 1. Current Crawl Time Breakdown

### Math Verification

```
107,528 team IDs
÷ 3 workers (parallel)
= 35,843 IDs per worker

35,843 IDs × 1,200ms per request
= 43,011,600ms per worker
= 43,011 seconds
= ~716 minutes
= ~11.9 hours ✓
```

**Confirmed: ~12 hours for a full crawl.** The published estimate is accurate.

### Bottleneck Identification

| Factor | Analysis |
|--------|----------|
| Rate limit delay (1200ms) | **Primary bottleneck.** 1200ms/request × 107K IDs is the dominant cost. |
| Worker count (3) | Contributes 3× speedup. Adding more workers reduces time proportionally — but only helps if GotSport allows it. |
| DB write speed | Not the bottleneck. Upserts on `(source, source_id)` are indexed; writes are fast relative to the network delay. |
| Team discovery step | One-time API call to `/team_ranking_data` to get 107,528 IDs. Negligible overhead. |

**The 1200ms delay is self-imposed** — it was set conservatively to avoid rate-limit blocks from GotSport. No documented GotSport rate limit exists. The question is whether 600ms or 300ms triggers blocking.

**Recommendation:** Do not reduce the delay without testing. Instead, reduce the NUMBER of IDs crawled (incremental filter). This is safer and more effective.

### What Fraction of IDs are Active?

Based on youth soccer seasonality:
- Peak season: March–June (spring showcases, leagues, state cups)
- Off-season: July–August (minimal games)
- Fall season: September–November
- Winter: December–February (light)

Conservative estimate: in any 90-day rolling window, **15–25% of teams have new games.** Call it 20% as a working estimate:

```
107,528 × 20% = ~21,500 active teams
21,500 × 1200ms ÷ 3 workers = ~2.87 hours
```

During peak season (March–June), active team fraction may be 30–40%:
```
107,528 × 35% = ~37,635 teams → ~5 hours at current settings
```

**Daily crawl of active teams only: 2.5–5 hours. Full pipeline including steps 1–9: ~3–6 hours total.**

---

## 2. Active Team Analysis

### Definition of "Active"
A team is **active** if it has played a game in the last 90 days OR has a scheduled game in the next 30 days.

### SQL Query for Active Team IDs

```sql
-- Active team IDs: played in last 90 days or scheduled in next 30
SELECT DISTINCT unnest(ARRAY[home_team_id, away_team_id]) AS gotsport_id
FROM games
WHERE source IN ('gotsport_api', 'gotsport')
  AND is_hidden = false
  AND (
    (game_status = 'completed' AND match_date > NOW() - INTERVAL '90 days')
    OR
    (game_status = 'scheduled' AND match_date BETWEEN NOW() AND NOW() + INTERVAL '30 days')
  )
  AND home_team_id IS NOT NULL
  AND away_team_id IS NOT NULL;
```

**Estimated output:** 15,000–40,000 team IDs depending on season. Save to a temp file or pass directly as `GSAPI_TEAM_IDS` environment variable:

```bash
# Generate active IDs file before daily crawl
psql youth_soccer -t -A -F',' -c "
  SELECT DISTINCT unnest(ARRAY[home_team_id, away_team_id])
  FROM games
  WHERE source IN ('gotsport_api','gotsport')
    AND is_hidden = false
    AND (
      (game_status = 'completed' AND match_date > NOW() - INTERVAL '90 days')
      OR (game_status = 'scheduled' AND match_date BETWEEN NOW() AND NOW() + INTERVAL '30 days')
    )
    AND home_team_id IS NOT NULL AND away_team_id IS NOT NULL
" > /tmp/active_team_ids.txt

# Feed to scraper (GotSport API scraper supports GSAPI_TEAM_IDS env var)
GSAPI_TEAM_IDS=$(cat /tmp/active_team_ids.txt) node crawl-gotsport-api-parallel.js
```

---

## 3. Pipeline Step Dependency Analysis

### Which Steps Must Run in Full?

| Step | Script | Incremental? | Notes |
|------|--------|-------------|-------|
| Step 1 | `cleanup-and-normalize.js` | **Full** | Operates on all games; normalization is idempotent but must cover all records |
| Step 2 | `backfill-age-gender.js` | **Incremental possible** | Only needs to run on games with null age_group/gender — add `WHERE age_group IS NULL` |
| Step 3 | `final-backfill.js` | **Incremental possible** | Same — target null fields only |
| Step 4 | `backfill-all-games.js` | **Full** | Sibling inference requires seeing all games in an event together |
| Step 5 | `fix-api-gender.js` | **Incremental possible** | Target null-gender API games only |
| Step 6 | `dedup-and-link-teams.js` | **Full** | Dedup requires comparing across all sources — can't be partial |
| Step 7 | `match-teams-cross-platform.js` | **Full** | Team merge graph requires global view |
| Step 8 | `compute-rankings.js` | **Full** | Rankings depend on all games and merges |
| Step 9 | `audit-data.js` | **Full** | Summary report — but non-blocking; can be skipped in daily run |

**Critical insight:** Steps 6–9 (dedup + ranking) MUST run in full every time rankings are updated. They cannot be made truly incremental without a major rewrite. Steps 1–5 (normalization) are largely idempotent and fast.

**Can ranking recompute independently of data ingestion?**  
Yes. `run-pipeline.sh` (Steps 1–9, no scraping) is already separate from `run-overnight.sh` (scraping + pipeline). Rankings can be recomputed at any time against existing data. This is the key enabler for Option A.

### CPU-bound vs IO-bound

| Steps | Bound by |
|-------|----------|
| 1–5 (normalization) | IO (DB reads/writes) — fast |
| 6 (dedup) | IO (large table scans + updates) — moderate |
| 7 (team matching) | CPU (fuzzy matching) + IO — moderate |
| 8 (rankings) | CPU (Elo computation, 9+1 phases) — heaviest step |
| 9 (audit) | IO — fast |

Steps 1–6: ~15–30 minutes total. Step 7: ~20–45 minutes. Step 8: varies with game volume, likely 30–60 minutes. Step 9: ~5 minutes.

**Full pipeline (no scraping): ~1–2 hours.** This fits comfortably after a 2.5–3 hour incremental crawl.

---

## 4. Three-Option Comparison

| | Option A: Smart Overnight Batch | Option B: Segmented Daily Runs | Option C: Event-Triggered Incremental |
|--|--|--|--|
| **Description** | Filter to active teams only (~20% of IDs); run daily at 2 AM | Split 107K IDs into 7 segments; rotate daily; recompute rankings daily | Monitor GotSport events endpoint; crawl only teams from newly completed events |
| **Daily latency** | 18–24 hours (runs once at 2 AM) | 18–24 hours (rankings recompute daily) | 2–6 hours post-event (near real-time) |
| **Staleness worst case** | 24 hours | 7 days for any given team | Hours (for active events) |
| **Implementation effort** | **Low — 4–8 hours** | Low-Medium — 6–10 hours | **High — 20–40 hours** |
| **Code changes** | Add active ID query + update cron schedule | Add segment rotation logic + update cron | New event monitoring script; routing logic |
| **Infrastructure required** | None — existing machine + cron | None — existing machine + cron | None, but significantly more complex logic |
| **Risk to data integrity** | Very low — same pipeline, smaller input | Low — partial coverage per day; dedup handles overlap | Medium — partial updates, stale ranking windows if event monitor misses events |
| **Can ship before DSS?** | **Yes (May 16, 2026)** | Yes | No — too much build time |
| **Ranking freshness** | Good (daily, active teams) | Acceptable (daily recompute, 7-day team staleness) | Excellent (near real-time) |

---

## 5. Recommendation

**Option A: Smart Overnight Batch — with Active Team Filter.**

**Justification:**
1. **Meets the goal.** Daily ranking updates, which matches USA Rank's cadence.
2. **Minimal risk.** The pipeline logic is unchanged. Only the input ID set shrinks. All existing normalization, dedup, and ranking steps run identically.
3. **Ships before DSS.** Can be implemented in a single 4–6 hour session.
4. **No new infrastructure.** Single machine, existing cron, existing PostgreSQL.
5. **Option C is better long-term** but requires 3–5× more development time and introduces monitoring complexity we can't safely ship before May 16.

Option B is a reasonable fallback but introduces 7-day team staleness as a structural property — unacceptable during peak showcase season when teams play multiple events per week.

---

## 6. Implementation Plan for Option A

**Target:** Daily pipeline runs, active teams only, shipping before May 16, 2026.

### Step 1: Write the Active ID Query Script (1–2 hours)

Create `get-active-team-ids.sh`:

```bash
#!/bin/bash
# get-active-team-ids.sh
# Outputs comma-separated active GotSport team IDs to stdout

psql youth_soccer -t -A -c "
SELECT string_agg(DISTINCT gotsport_id, ',')
FROM (
  SELECT unnest(ARRAY[home_team_id, away_team_id]) AS gotsport_id
  FROM games
  WHERE source IN ('gotsport_api','gotsport')
    AND is_hidden = false
    AND (
      (game_status = 'completed' AND match_date > NOW() - INTERVAL '90 days')
      OR (game_status = 'scheduled' AND match_date BETWEEN NOW() AND NOW() + INTERVAL '30 days')
    )
    AND home_team_id IS NOT NULL
    AND away_team_id IS NOT NULL
) t
"
```

### Step 2: Create `run-daily.sh` (1 hour)

```bash
#!/bin/bash
# run-daily.sh
# Daily pipeline: active teams only + full normalization + ranking recompute
# Replaces run-overnight.sh for nightly execution
# run-overnight.sh preserved as fallback (weekly full crawl)

set -e
LOG_FILE="/var/log/evo-draw/pipeline-$(date +%Y%m%d).log"
exec >> "$LOG_FILE" 2>&1

echo "=== Evo Draw Daily Pipeline: $(date) ==="

# Step 0a: Get active team IDs
echo "Fetching active team IDs..."
ACTIVE_IDS=$(bash get-active-team-ids.sh)
ID_COUNT=$(echo "$ACTIVE_IDS" | tr ',' '\n' | wc -l)
echo "Active teams: $ID_COUNT"

# Step 0b: Crawl active teams only
echo "Starting GotSport API crawl (active teams only)..."
GSAPI_TEAM_IDS="$ACTIVE_IDS" GSAPI_SKIP_RANKINGS=true \
  node crawl-gotsport-api-parallel.js

echo "GotSport crawl complete."

# Steps 1–9: Full pipeline
echo "Running pipeline..."
bash run-pipeline.sh

echo "=== Daily pipeline complete: $(date) ==="
```

### Step 3: Update Cron Schedule (30 min)

```bash
# Edit crontab: crontab -e

# Daily pipeline — active teams only, 2 AM
0 2 * * * cd /path/to/evo-draw && bash run-daily.sh

# Weekly full crawl — all 107K IDs, Sunday 11 PM
# (Ensures no teams fall through the active filter permanently)
0 23 * * 0 cd /path/to/evo-draw && bash run-overnight.sh
```

### Step 4: Add Run Monitoring (1–2 hours)

Create `check-pipeline-health.sh` to detect failed runs:

```bash
#!/bin/bash
# check-pipeline-health.sh — called by cron after daily run
# Alerts if pipeline log shows ERROR or if last run was >26 hours ago

LOG_FILE="/var/log/evo-draw/pipeline-$(date +%Y%m%d).log"

if grep -q "ERROR\|FATAL\|Cannot connect" "$LOG_FILE" 2>/dev/null; then
  echo "PIPELINE ERROR detected — check $LOG_FILE" | mail -s "Evo Draw Pipeline FAILED" presten@email.com
fi

# Check last successful rankings write
LAST_UPDATE=$(psql youth_soccer -t -A -c \
  "SELECT MAX(updated_at) FROM rankings")
echo "Last rankings update: $LAST_UPDATE"
```

```bash
# Add to crontab — run 30 min after daily pipeline expected to complete
30 8 * * * bash /path/to/check-pipeline-health.sh
```

### Step 5: Test on 3 Days (1–2 hours)

- Day 1: Run `run-daily.sh` manually, observe ID count and crawl duration
- Verify rankings table updated (`SELECT MAX(updated_at) FROM rankings`)
- Confirm game counts are reasonable (new games since yesterday)
- Day 2–3: Let cron run unattended, check logs

**Total implementation: 4–7 hours.** Well within reach before DSS.

---

## 7. Cron Schedule

```crontab
# Evo Draw — Daily Pipeline Cadence
# Implemented: 2026-04-23

# Daily: active teams crawl + full pipeline (Mon–Sat)
0 2 * * 1-6 cd /path/to/evo-draw && bash run-daily.sh

# Weekly: full 107K ID crawl + full pipeline (Sunday, catches all inactive teams)
0 23 * * 0 cd /path/to/evo-draw && bash run-overnight.sh

# Health check: runs at 8:30 AM, after daily pipeline expected to finish
30 8 * * * bash /path/to/check-pipeline-health.sh
```

**Expected completion times:**
- Daily run start: 2:00 AM
- Crawl done (active teams, ~21K IDs): ~4:30–5:00 AM
- Pipeline steps 1–9: 5:00–7:00 AM
- Rankings updated: ~7:00–8:00 AM
- Health check: 8:30 AM

Rankings are fresh every morning by 8 AM — same effective cadence as USA Rank.

---

## 8. Rollback Plan

If the incremental pipeline causes data integrity issues (unexpected ranking shifts, missing teams, stale data complaints), revert immediately:

### Immediate Rollback (5 minutes)

```bash
# 1. Disable daily cron — comment out the Mon-Sat line
crontab -e
# Comment: # 0 2 * * 1-6 cd /path/to/evo-draw && bash run-daily.sh

# 2. Re-enable overnight batch manually
bash run-overnight.sh &

# Rankings will be refreshed with full 107K ID crawl within 12 hours
```

### Rollback Triggers (when to execute)

- Rankings for known active teams are not updating day-over-day
- User reports that a recently played game is not showing in rankings
- Active ID query returns fewer than 5,000 IDs (likely query bug)
- Pipeline log shows completion without errors but `MAX(updated_at)` in rankings is >36 hours old

### Post-Rollback Diagnosis

```sql
-- Check if active ID query returned reasonable results
-- (run manually before re-enabling incremental)
SELECT COUNT(DISTINCT unnest(ARRAY[home_team_id, away_team_id]))
FROM games
WHERE source IN ('gotsport_api','gotsport')
  AND is_hidden = false
  AND match_date > NOW() - INTERVAL '90 days';

-- Should return 15,000–40,000 during season
-- If <5,000: query bug or DB issue
-- If >60,000: normal; full crawl may be warranted
```

`run-overnight.sh` is preserved and unchanged — it remains the tested fallback at all times.

---

## Summary

| Item | Value |
|------|-------|
| Current crawl time | ~12 hours (verified) |
| Bottleneck | 1200ms delay × 107K IDs |
| Active team estimate (90 days) | ~15,000–40,000 (20–35% of total) |
| Daily crawl time (active only) | **2.5–5 hours** |
| Full pipeline (Steps 1–9) | ~1–2 hours |
| Total daily runtime | **~3.5–7 hours** (done before 9 AM) |
| Recommendation | **Option A: Smart Overnight Batch** |
| Implementation effort | **4–7 hours** |
| Ships before DSS? | **Yes** |
| New infrastructure required | **None** |
| Cron | `0 2 * * 1-6` (daily) + `0 23 * * 0` (weekly full) |

---

## Related Notes
- [[Data Pipeline]] — 9 steps detailed
- [[GotSport API Scraper]] — `GSAPI_TEAM_IDS` env var, 3-worker config
- [[Database Schema]] — `games` table, `rankings` table
- [[Server and Frontend]] — rankings served via API
- [[USA Rank Competitive Intel]] — daily update frequency gap

---

*Scoped by FORGE — 2026-04-23*
