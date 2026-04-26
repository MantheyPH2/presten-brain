---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Source Freshness Audit Template.md"
---

# Task: Source Freshness Audit Template

## Context

SENTINEL makes authorization decisions based on the assumption that ingested game data is current. The Stage 2 authorization gate, the May 1 deployment health check, and the DSS readiness review all depend on Evo Draw's data being fresh. But SENTINEL currently has no repeatable structured view of per-source data freshness.

The Pre-DSS Source Coverage Report (filed 2026-04-25) documents which sources are active and their coverage scope. It does not answer: how recently did each source ingest new games, and is that freshness within the expected window?

This template provides that answer. It is not a one-time document — it is designed to be re-run at each SENTINEL authorization decision point (before May 1 deployment, before Stage 2 authorization, at May 9 readiness gate).

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Source Freshness Audit Template.md`

This is a template with SQL and a structured table. FORGE populates it at each audit run. The header tracks when the audit was last run.

---

### Audit Header (FORGE fills each run)

```
Audit Run Date: ___________
Audit Run By: FORGE
Prior Audit Date: ___________
Purpose: [Stage 2 Auth / May 1 Deployment / May 9 DSS Readiness / Routine]
```

---

### Section 1: Per-Source Freshness Table

FORGE populates this table by running the SQL query below against the `games` table.

| Source | Org-ID / Feed | Most Recent Game Date | Games (Last 30d) | Games (Last 7d) | Expected Cadence | Freshness Status |
|--------|--------------|----------------------|-----------------|-----------------|-----------------|-----------------|
| GotSport — ECNL Girls | [org-id] | | | | Weekly+ | |
| GotSport — ECNL Boys | [org-id] | | | | Weekly+ | |
| GotSport — MLS NEXT | [org-id] | | | | Weekly+ | |
| GotSport — GA Girls | [org-id] | | | | Weekly+ | |
| GotSport — GA Boys | [org-id] | | | | Weekly+ | |
| GotSport — ECNL RL Girls | [org-id] | | | | Weekly+ | |
| GotSport — ECNL RL Boys | [org-id] | | | | Weekly+ | |
| GotSport — NPL | [org-id] | | | | Weekly+ | |
| GotSport — DPL | [org-id] | | | | Weekly+ | |
| TGS (ECNL supplemental) | TGS feed | | | | Per TGS schedule | |
| [Stage 2 org-IDs — fill after Stage 2 runs] | | | | | | |

**Freshness Status Values:**
- `GREEN` — most recent game within expected cadence window
- `YELLOW` — most recent game 1.5x expected cadence (may be offseason or scheduling gap)
- `RED` — most recent game > 2x expected cadence; or 0 games in last 30 days when season is active

---

### Section 2: SQL Query Pack

**Query 1 — Per-source most recent game and 30-day / 7-day counts**
```sql
SELECT
  source,
  org_id,
  MAX(game_date) AS most_recent_game,
  COUNT(*) FILTER (WHERE game_date >= CURRENT_DATE - INTERVAL '30 days') AS games_30d,
  COUNT(*) FILTER (WHERE game_date >= CURRENT_DATE - INTERVAL '7 days') AS games_7d
FROM games
WHERE game_date IS NOT NULL
GROUP BY source, org_id
ORDER BY most_recent_game DESC;
```

**Query 2 — Ingestion freshness (when was data last ADDED, not when was game last PLAYED)**
```sql
SELECT
  source,
  org_id,
  MAX(ingested_at) AS most_recent_ingestion,
  COUNT(*) FILTER (WHERE ingested_at >= NOW() - INTERVAL '30 days') AS ingested_30d
FROM games
WHERE ingested_at IS NOT NULL
GROUP BY source, org_id
ORDER BY most_recent_ingestion DESC;
```

Note: Query 1 shows recency of games played; Query 2 shows recency of data ingestion. Both are relevant: a source can have recent game dates from a past season's retroactive import but not be actively receiving new games.

**Query 3 — Null coverage check (data quality signal)**
```sql
SELECT
  source,
  COUNT(*) AS total_games,
  COUNT(*) FILTER (WHERE home_team_id IS NULL) AS null_home_team,
  COUNT(*) FILTER (WHERE away_team_id IS NULL) AS null_away_team,
  COUNT(*) FILTER (WHERE game_date IS NULL) AS null_date,
  COUNT(*) FILTER (WHERE score_home IS NULL AND score_away IS NULL) AS unscored
FROM games
WHERE ingested_at >= NOW() - INTERVAL '30 days'
GROUP BY source
ORDER BY total_games DESC;
```

High null rates in recent ingestion indicate a parsing regression, not a freshness problem. Flag to SENTINEL separately if null rate > 5% for any source.

---

### Section 3: Freshness Verdict

FORGE fills this section after completing the table in Section 1.

```
All active sources GREEN: YES / NO
Sources at YELLOW: [list]
Sources at RED: [list]
Sources in expected offseason (YELLOW acceptable): [list]

Overall freshness verdict: GREEN / YELLOW / RED
SENTINEL authorization impact:
  - May 1 deployment: [CLEARED / CONDITIONAL / BLOCKED]
  - Stage 2 authorization: [CLEARED / CONDITIONAL / BLOCKED]
  - DSS readiness: [CLEARED / CONDITIONAL / BLOCKED]

FORGE recommendation: ___________
```

---

### Section 4: Historical Freshness Log

Each time FORGE runs this audit, record the verdict summary here so SENTINEL can see the trend:

| Date | Purpose | Overall Verdict | Notable Flags |
|------|---------|----------------|---------------|
| [first run] | | | |

This section builds the freshness history without requiring SENTINEL to read old reports.

---

## Quality Standard

- All SQL queries must use actual table and column names. Confirm names against the database schema before filing.
- The freshness status thresholds (GREEN/YELLOW/RED) must be explicitly defined — not "seems fresh."
- The template must be re-runnable without modification. FORGE fills in the data cells; the structure is stable.
- If a column name in the template does not match the actual schema, document the correct name in a note at the top.

## Why This Matters

SENTINEL's authorization decisions currently rely on FORGE's briefing narrative for data freshness. A briefing that says "sources look healthy" is not the same as a table showing each source's most recent game date. With this template, SENTINEL's pre-authorization freshness check is a 60-second table scan: all GREEN → cleared. Any RED → hold. The first run of this template before May 1 deployment establishes the baseline; every subsequent run at a gate checkpoint is a comparison against that baseline. This is the data quality governance layer Evo Draw currently lacks.

## References

- `Infrastructure/Pre-DSS Source Coverage Report.md` — coverage scope that this template adds freshness tracking to
- `Infrastructure/Stage 2 Expansion — SENTINEL Authorization Gate.md` — authorization gate this audit feeds
- `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` — May 1 gate this audit feeds
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — per-run health checks this template supplements
