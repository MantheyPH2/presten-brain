---
title: Source Freshness Audit Template
tags: [infrastructure, data-quality, freshness, sources, monitoring, authorization, evo-draw, forge]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: active — re-run at each SENTINEL authorization gate
task: task-2026-04-25-source-freshness-audit-template
---

# Source Freshness Audit Template

> **Purpose:** Provide SENTINEL with a structured per-source freshness table at each authorization decision point. FORGE runs the SQL queries, fills in the table, and files a freshness verdict. SENTINEL's authorization check becomes a 60-second table scan: all GREEN → cleared. Any RED → hold.

> **Re-run at:** May 1 deployment pre-authorization, Stage 2 authorization, May 9 DSS readiness gate, and any post-incident health check.

> **Column name notes:** Queries use `source_org_id`, `source_name`, `created_at`, and `game_date` per the confirmed schema in `Source Health Monitoring Queries.md`. If your schema differs, update accordingly.

---

## Audit Header

```
Audit Run Date:   ___________
Audit Run By:     FORGE
Prior Audit Date: ___________
Purpose: [ ] Stage 2 Auth  [ ] May 1 Deployment  [ ] May 9 DSS Readiness  [ ] Routine
```

---

## Section 1: Per-Source Freshness Table

FORGE populates this table by running the SQL in Section 2 and applying the freshness status rules below.

| Source | source_org_id | Most Recent Game Date | Games (Last 30d) | Games (Last 7d) | Expected Cadence | Freshness Status |
|--------|--------------|----------------------|-----------------|-----------------|-----------------|-----------------|
| GotSport — ECNL Girls | TBD | | | | Weekly+ | |
| GotSport — ECNL Boys | TBD | | | | Weekly+ | |
| GotSport — MLS NEXT | TBD | | | | Weekly+ | |
| GotSport — GA Girls | TBD | | | | Weekly+ | |
| GotSport — GA Boys | TBD | | | | Weekly+ | |
| GotSport — GA ASPIRE | TBD | | | | Weekly+ | |
| GotSport — ECNL RL Girls | TBD | | | | Weekly+ | |
| GotSport — ECNL RL Boys | TBD | | | | Weekly+ | |
| GotSport — NPL | TBD | | | | Weekly+ | |
| GotSport — DPL | TBD | | | | Weekly+ | |
| TGS (ECNL supplemental) | TGS feed | | | | Per TGS schedule | |
| *[Stage 2 org-IDs — add rows after Stage 2 runs]* | | | | | | |

**Freshness Status Rules:**

| Status | Condition |
|--------|-----------|
| `GREEN` | Most recent game within the expected cadence window; consistent volume over last 30d |
| `YELLOW` | Most recent game at 1.5× expected cadence (e.g., 10–14 days for a weekly source), OR volume noticeably below seasonal baseline but > 0 |
| `RED` | Most recent game > 2× expected cadence during active season, OR 0 games in last 30 days during active season |
| `OFFSEASON` | 0 or near-0 games expected based on Section 1 of `Infrastructure/GotSport Org-ID Inactive Detection Protocol.md`; not a health concern |

> [!note] YELLOW is not a blocker unless two consecutive audits show YELLOW for the same source. One YELLOW = flag and monitor. Two consecutive YELLOW = investigate before authorization.

---

## Section 2: SQL Query Pack

Run all three queries from a `psql youth_soccer` session. Copy results into Section 1.

**Query 1 — Per-source most recent game date and volume counts**

```sql
SELECT
  source_name,
  source_org_id,
  MAX(game_date) AS most_recent_game,
  COUNT(*) FILTER (WHERE game_date >= CURRENT_DATE - INTERVAL '30 days') AS games_30d,
  COUNT(*) FILTER (WHERE game_date >= CURRENT_DATE - INTERVAL '7 days')  AS games_7d
FROM games
WHERE game_date IS NOT NULL
GROUP BY source_name, source_org_id
ORDER BY most_recent_game DESC NULLS LAST;
```

*Answers: when were the most recent games played per source, and is that recency within a healthy window?*

---

**Query 2 — Ingestion freshness (when was data last added to the DB)**

```sql
SELECT
  source_name,
  source_org_id,
  MAX(created_at) AS most_recent_ingestion,
  COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '30 days') AS ingested_30d
FROM games
WHERE created_at IS NOT NULL
GROUP BY source_name, source_org_id
ORDER BY most_recent_ingestion DESC NULLS LAST;
```

*Answers: is the pipeline actively writing new rows for this source? A source can have recent `game_date` values from a prior retroactive import while no longer receiving new games. Compare this result against Query 1.*

> [!important] Both queries matter. A source with recent game_date but stale created_at is not actively ingesting. Flag this to SENTINEL.

---

**Query 3 — Null coverage check (data quality signal for recent ingestion)**

```sql
SELECT
  source_name,
  COUNT(*)                                                            AS total_games,
  COUNT(*) FILTER (WHERE home_team_id IS NULL)                       AS null_home_team,
  COUNT(*) FILTER (WHERE away_team_id IS NULL)                       AS null_away_team,
  COUNT(*) FILTER (WHERE game_date IS NULL)                          AS null_game_date,
  COUNT(*) FILTER (WHERE home_score IS NULL AND away_score IS NULL)  AS unscored
FROM games
WHERE created_at >= NOW() - INTERVAL '30 days'
GROUP BY source_name
ORDER BY total_games DESC;
```

*Answers: is recent data well-formed? High null rates in recent ingestion indicate a parsing regression, not a freshness problem. Flag separately if null rate > 5% for any column in any source.*

---

## Section 3: Freshness Verdict

FORGE fills this section after completing Section 1.

```
All active sources GREEN:       YES / NO
Sources at YELLOW:              [list, or "none"]
Sources at RED:                 [list, or "none"]
Sources in expected OFFSEASON:  [list, or "none"]
Sources with stale ingestion
  despite recent game dates:    [list, or "none"]

Overall freshness verdict:      GREEN / YELLOW / RED

SENTINEL authorization impact:
  May 1 deployment:       [ CLEARED ]  [ CONDITIONAL ]  [ BLOCKED ]
  Stage 2 authorization:  [ CLEARED ]  [ CONDITIONAL ]  [ BLOCKED ]
  DSS readiness:          [ CLEARED ]  [ CONDITIONAL ]  [ BLOCKED ]

FORGE recommendation:
___________
```

**Verdict rules:**
- All active sources GREEN → Overall: GREEN
- Any YELLOW (not offseason) → Overall: YELLOW; authorization CONDITIONAL pending investigation
- Any RED during active season → Overall: RED; authorization BLOCKED until resolved
- Any source with null rate > 5% → flag separately regardless of overall verdict

---

## Section 4: Historical Freshness Log

Each time FORGE runs this audit, add a summary row so SENTINEL can track the trend without reading full reports.

| Date | Purpose | Overall Verdict | Sources at YELLOW/RED | Notable Flags |
|------|---------|----------------|----------------------|---------------|
| *(first run — fill in date)* | | | | |

---

## References

- `Infrastructure/Pre-DSS Source Coverage Report — May 2026.md` — coverage scope this template adds freshness tracking to
- `Infrastructure/Stage 2 Expansion — SENTINEL Authorization Gate.md` — authorization gate this audit feeds; freshness is one of the gate conditions
- `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` — May 1 gate this audit feeds
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — per-run health checks this template supplements at authorization gates
- `Infrastructure/GotSport Org-ID Inactive Detection Protocol.md` — companion document for diagnosing individual 0-game org-IDs detected by this audit
- `Infrastructure/Source Health Monitoring Queries.md` — ongoing monitoring queries this template extends to authorization-gate checkpoints

*FORGE — 2026-04-25*
