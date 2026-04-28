---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: open
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/May 1 Pipeline Launch — ELO Rankings Monitoring Plan.md"
---

# Task: May 1 Pipeline Launch — ELO Rankings Monitoring Plan

## Background

May 1 is the daily pipeline launch date. FORGE has three coverage documents for the infrastructure side: Day-Of Runbook, Morning-After Runbook, and Incident Response Playbook. ELO has no equivalent for the rankings side.

When new games from Stage 2 GotSport org-IDs arrive for the first time, ratings will shift. Expected: top-tier teams gain game count, ratings stabilize slightly. Possible anomalies: teams with no history before Stage 2 receiving outsized rating shifts, source distribution skewing (too many games from one region), or rating variance collapsing (new games all clustered at similar strength). FORGE catches infrastructure failures. ELO catches output quality failures. Without this plan, ELO has no structured signal from May 1 through May 7.

## Deliverable

`02 - Tiger Tournaments/Projects/Infrastructure/May 1 Pipeline Launch — ELO Rankings Monitoring Plan.md`

## Structure

### Section 1: Monitoring Schedule

| Date | Trigger | ELO Action |
|------|---------|------------|
| May 1 | First pipeline batch completes | Run all 5 post-run checks (Section 2) |
| May 2 | Second batch completes | Run all 5 checks; compare to May 1 baseline |
| May 3 | Third batch completes | Run all 5 checks; confirm rolling trend |
| May 7 | End of Week 1 window | File Week 1 Rankings Summary to FORGE monitoring log |

### Section 2: Post-Run Rankings Checks

Run after each batch. All SQL uses `[last_run_timestamp]` as the pipeline run timestamp.

**R1 — New Game Volume**
```sql
SELECT COUNT(*) AS new_games
FROM games
WHERE created_at > '[last_run_timestamp]';
```
| GREEN | YELLOW | RED |
|-------|--------|-----|
| ≥ 100 new games | 20–99 | < 20 |

RED action: notify FORGE immediately — Stage 2 org-IDs may not have fired.

---

**R2 — Average Rating Shift**
```sql
SELECT ROUND(AVG(ABS(rating - prev_rating)), 1) AS avg_delta
FROM teams
WHERE updated_at > '[last_run_timestamp]'
  AND prev_rating IS NOT NULL;
```
| GREEN | YELLOW | RED |
|-------|--------|-----|
| < 10 pts avg | 10–25 pts | > 25 pts |

RED action: identify top-10 largest movers; check whether new game source is plausible for each.

---

**R3 — Large Mover Count**
```sql
SELECT COUNT(*) AS large_movers
FROM teams
WHERE ABS(rating - prev_rating) > 50
  AND updated_at > '[last_run_timestamp]';
```
| GREEN | YELLOW | RED |
|-------|--------|-----|
| 0–5 teams | 6–20 teams | > 20 teams |

RED action: pause recompute recommendation; file SENTINEL alert before next run.

---

**R4 — Source Distribution**
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

RED action: notify FORGE — Stage 2 config may not have activated.

---

**R5 — Rating Variance**
```sql
SELECT ROUND(STDDEV(rating), 1) AS rating_stddev
FROM teams
WHERE rating IS NOT NULL;
```
Baseline: capture pre-May-1 stddev value before first run. Compare each subsequent run.

| GREEN | YELLOW | RED |
|-------|--------|-----|
| Within 5% of baseline | 5–15% shift | > 15% shift |

RED action: variance collapse (too many similar-strength games) or variance spike (low-K outlier); file anomaly report before next run.

### Section 3: Alert Conditions and Escalation

Any RED condition → file SENTINEL queue alert immediately; do not wait for next briefing.

Alert format:
```
type: alert
priority: high
condition: [R1/R2/R3/R4/R5] RED — [description]
data: [query output]
recommended action: [what ELO suggests]
```

### Section 4: Week 1 Rankings Summary

File as a section in FORGE's Week 1 Monitoring Log by May 7.

Format:
- **Games added (May 1–7):** [total new games]
- **Avg rating shift (per run):** [avg across all runs]
- **Large mover events:** [count of R3 triggers, if any]
- **Source distribution:** [new sources confirmed active]
- **Alert events:** [none / describe]
- **ELO assessment:** [stable / monitoring / needs-attention + one sentence]

---

## Instructions

1. Write this plan before April 30 (before May 1 launch)
2. Capture pre-launch rating stddev baseline before May 1 first run
3. Execute all 5 checks after each May 1–3 pipeline batch
4. File any RED alerts immediately to SENTINEL queue
5. File Week 1 Rankings Summary in FORGE monitoring log by end of May 7

## Due

2026-04-30 — plan written and pre-launch baseline captured before May 1 first run.
