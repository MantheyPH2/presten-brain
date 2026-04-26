---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: high
due: 2026-05-01
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/ECNL RL Staleness Verification — Presten Execution Package.md"
---

# Task: ECNL RL Staleness Verification — Presten Execution Package

## Context

`Infrastructure/ECNL RL Staleness Verification Query — April 2026.md` has been written and is sitting at Score 9 in the Source Coverage Priority Matrix. It has been deferred from every prior FORGE plan for 7+ days. It is the highest-priority unverified source item heading into DSS.

The verification query file exists. This task converts it into a complete Presten execution package — same format and standard as every other execution package in the vault (`Boys Option A Spot Check — Presten Execution Package`, `Pre-April-28 Baseline Snapshot — Presten Execution Package`, etc.).

The gap: the staleness query as written is a query. A Presten execution package is a document Presten opens, runs top-to-bottom in a single psql session, and knows exactly what to do with the result. The current file may not have: PASS/YELLOW/FAIL thresholds, an output file template, specific downstream actions, or a run timing recommendation tied to the April 28-May 1 window.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/ECNL RL Staleness Verification — Presten Execution Package.md`

Do NOT create a new document from scratch. Pull the query from the existing staleness verification file and structure it as follows.

---

### Pre-Run Gate

One condition to confirm before running: no ECNL RL data exists before the TGS AthleteOne ingestion path was active. If Presten is uncertain when that path became active, note the earliest expected game date and use it as a sanity check.

**Run timing:** Run at the next available psql session — ideally April 29 after the GA ASPIRE recompute, at the same time as the April 29 Decision Tree queries. ECNL RL staleness is independent of April 28 — it does not need to wait for the recompute. If the April 29 session runs long, this can be run at the May 1 pre-deployment check.

---

### Query 1 — ECNL RL Ingestion Recency

Pull this from the existing staleness verification file. Ensure it returns:
- `event_tier`, `source`, `MAX(created_at) AS last_ingested`, `COUNT(*) AS game_count`
- For `event_tier = 'ecnl_rl'` across all sources

**PASS:** `days_since_most_recent` ≤ 14 AND `game_count` ≥ 500 for current season.
**YELLOW:** 15–30 days since most recent, OR 200–499 games → log in SENTINEL briefing. ECNL RL is lower-volume than full ECNL; confirm whether this reflects low play volume or an ingestion gap.
**FAIL:** > 30 days since most recent OR < 200 games → escalate to SENTINEL. If ECNL RL ingestion is stale, ECNL migration rating continuity analysis will be based on incomplete data.

Save output as: `02 - Tiger Tournaments/Projects/Reports/ecnl-rl-staleness-check-[date].md`

---

### Query 2 — ECNL RL vs Full ECNL Volume Comparison

A comparative check: how does ECNL RL game volume compare to full ECNL game volume?

```sql
SELECT event_tier, COUNT(*) AS game_count, MAX(game_date) AS most_recent_game
FROM games
WHERE event_tier IN ('ecnl', 'ecnl_rl')
  AND season = '2025-2026'
GROUP BY event_tier;
```

**Expected:** ECNL RL game count is lower than full ECNL (ECNL RL is a separate sub-league). If ECNL RL count > ECNL count, investigate — this is unexpected given relative league sizes.

---

### Decision Table

| Query 1 Result | Query 2 Result | Action |
|----------------|----------------|--------|
| PASS | Expected ratio | No action. File GREEN status in next FORGE briefing. |
| PASS | ECNL RL > ECNL | Flag to SENTINEL. Investigate whether ECNL RL is being over-counted or full ECNL under-counted. |
| YELLOW | Any | Log in FORGE briefing. ELO notes low ECNL RL volume in ECNL migration rating continuity analysis. |
| FAIL | Any | Escalate to SENTINEL immediately. Do not proceed with ECNL migration planning until resolved. |

---

### Downstream Dependencies

This verification result feeds:
1. **ECNL Migration Rating Continuity Spec (ELO)** — if ECNL RL is stale or low-volume, the migration spec's rating continuity model needs a caveat
2. **ecnl_verified Column Accuracy** — the `ecnl_verified` backfill targets `event_tier = 'ecnl'` games, not `ecnl_rl`. If the two tiers have different sources with different reliability, note this for ELO
3. **May 9 DSS Readiness Assessment** — ECNL data quality is a background assumption in Evo Draw's competitive claim. If ECNL RL is stale, that claim weakens

---

## Quality Standard

- Match the exact structure and tone of `Rankings/Boys Option A Spot Check — Presten Execution Package.md`
- Every query must be copy-paste ready
- Every threshold must be a specific number
- The decision table must have an explicit action for every outcome combination
- Total estimated run time for Presten: ≤ 10 minutes in psql

## Why This Matters

ECNL RL staleness has been in FORGE's plan since April 22. It is Score 9 — the highest priority item in the Source Coverage Matrix that is not an active pipeline fix. It has been deferred because other URGENT tasks took priority. Those urgent tasks are now complete. This is the window (April 26-29) to run it before the May 9 DSS readiness check adds time pressure. If ECNL RL is stale, ELO needs to know before finalizing the ECNL migration continuity spec (due May 10).

## References

- `Infrastructure/ECNL RL Staleness Verification Query — April 2026.md` — source query (pull verbatim)
- `Infrastructure/Source Coverage Priority Matrix.md` — Score 9 ranking
- `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` — downstream dependency
- `task-2026-04-26-ecnl-verified-ingestion-path-audit.md` — related audit covering ecnl_verified accuracy
