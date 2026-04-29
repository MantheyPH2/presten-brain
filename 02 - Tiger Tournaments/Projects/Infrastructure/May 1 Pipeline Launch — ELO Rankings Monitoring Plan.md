---
title: May 1 Pipeline Launch — ELO Rankings Monitoring Plan
tags: [pipeline, rankings, monitoring, may1, elo, infrastructure]
created: 2026-04-28
author: ELO
status: active — monitoring begins May 1
---

# May 1 Pipeline Launch — ELO Rankings Monitoring Plan

## Purpose

When new games from Stage 2 GotSport org-IDs arrive for the first time on May 1, ratings will shift. FORGE catches infrastructure failures. ELO catches output quality failures. This plan defines ELO's structured signal from May 1 through May 7.

---

## Section 1: Monitoring Schedule

| Date | Trigger | ELO Action |
|------|---------|------------|
| May 1 | First pipeline batch completes | Run all 5 post-run checks (Section 2) |
| May 2 | Second batch completes | Run all 5 checks; compare to May 1 baseline |
| May 3 | Third batch completes | Run all 5 checks; confirm rolling trend |
| May 7 | End of Week 1 window | File Week 1 Rankings Summary to FORGE monitoring log |

---

## Section 2: Post-Run Rankings Checks

*Run after each batch. All SQL uses `[last_run_timestamp]` as the pipeline run timestamp.*

### R1 — New Game Volume

```sql
SELECT COUNT(*) AS new_games
FROM games
WHERE created_at > '[last_run_timestamp]';
```

| GREEN | YELLOW | RED |
|-------|--------|-----|
| ≥ 100 new games | 20–99 | < 20 |

**RED action:** Notify FORGE immediately — Stage 2 org-IDs may not have fired.

---

### R2 — Average Rating Shift

```sql
SELECT ROUND(AVG(ABS(rating - prev_rating)), 1) AS avg_delta
FROM teams
WHERE updated_at > '[last_run_timestamp]'
  AND prev_rating IS NOT NULL;
```

| GREEN | YELLOW | RED |
|-------|--------|-----|
| < 10 pts avg | 10–25 pts | > 25 pts |

**RED action:** Identify top-10 largest movers; check whether new game source is plausible for each.

---

### R3 — Large Mover Count

```sql
SELECT COUNT(*) AS large_movers
FROM teams
WHERE ABS(rating - prev_rating) > 50
  AND updated_at > '[last_run_timestamp]';
```

| GREEN | YELLOW | RED |
|-------|--------|-----|
| 0–5 teams | 6–20 teams | > 20 teams |

**RED action:** Pause recompute recommendation; file SENTINEL alert before next run.

---

### R4 — Source Distribution

```sql
SELECT source, COUNT(*) AS game_count
FROM games
WHERE created_at > '[last_run_timestamp]'
GROUP BY source
ORDER BY game_count DESC;
```

Expected: new GotSport org-IDs appear as distinct source entries. Compare to pre-launch baseline.

| GREEN | YELLOW | RED |
|-------|--------|-----|
| ≥ 3 new org-ID sources visible | 1–2 new sources | No new sources |

**RED action:** Notify FORGE — Stage 2 config may not have activated.

---

### R5 — Rating Variance

```sql
SELECT ROUND(STDDEV(rating), 1) AS rating_stddev
FROM teams
WHERE rating IS NOT NULL;
```

Capture pre-May-1 stddev baseline before first run. Compare each subsequent run.

| GREEN | YELLOW | RED |
|-------|--------|-----|
| Within 5% of baseline | 5–15% shift | > 15% shift |

**RED action:** Variance collapse (too many similar-strength games) or variance spike (low-K outlier); file anomaly report before next run.

**Pre-launch baseline capture:** ELO runs R5 SQL before May 1 first run and records: `pre_launch_stddev = [value]`.

---

## Section 3: Alert Conditions and Escalation

Any RED condition → file SENTINEL queue alert immediately; do not wait for next briefing.

Alert format:
```
type: alert
priority: high
condition: [R1/R2/R3/R4/R5] RED — [description]
data: [query output]
recommended action: [what ELO suggests]
```

---

## Section 4: Week 1 Rankings Summary

File as a section in FORGE's Week 1 Monitoring Log by May 7.

| Metric | Value |
|--------|-------|
| Games added (May 1–7) | _[fill]_ |
| Avg rating shift per run | _[fill]_ |
| Large mover events (R3 triggers) | _[fill]_ |
| Source distribution — new sources confirmed | _[fill]_ |
| Alert events | _[none / describe]_ |
| ELO assessment | _[stable / monitoring / needs-attention + one sentence]_ |

---

## Instructions

1. Capture pre-launch rating stddev baseline before May 1 first run
2. Execute all 5 checks after each May 1–3 pipeline batch
3. File any RED alerts immediately to SENTINEL queue
4. File Week 1 Rankings Summary in FORGE monitoring log by end of May 7

---

*ELO — 2026-04-28*

## References

- `Infrastructure/May 1 Pipeline Launch — FORGE Day-Of Runbook.md`
- `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` — pre-May-1 game count reference
