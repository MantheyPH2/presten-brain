---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: completed
priority: high
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/ECNL RL Staleness Verification Query — April 2026.md"
---

# Task: ECNL RL Staleness Verification Query

## Context

The Source Coverage Priority Matrix (filed this session) ranks ECNL RL as the highest-priority gap: score 9 across Volume (3) + Impact (3) + Effort (3). The matrix notes that a 10-minute DB query either confirms ECNL RL is current or surfaces a silent staleness problem.

ECNL RL represents ~700 clubs at Tier 2. If ingestion has drifted, it's the largest undetected ranking distortion currently possible in Evo Draw. The Source Health Monitoring Queries doc (`Infrastructure/Source Health Monitoring Queries.md`) provides a weekly monitoring template, but it does not isolate ECNL RL specifically. Presten needs a purpose-built verification query to run at the next psql session.

This task: write the verification document. One query, copy-paste ready, with expected output and pass/fail thresholds.

## What to Do

### Step 1: Design the Verification Query

Write a SQL query that checks ECNL RL's ingestion health. The query should:

1. Filter teams affiliated with ECNL RL (by `league_id`, `event_type`, or equivalent identifier — pull from existing source-to-league mapping in the vault)
2. Return: `COUNT(games)`, `MAX(game_date)` (most recent game ingested), `MIN(game_date)` (oldest game), and `DATEDIFF` or equivalent to show days since most recent ingestion
3. Group by season if the schema supports it, so staleness is visible per-season rather than masked by historical data

Adapt exact column names to match the Evo Draw schema as documented in the vault. If the exact column names are uncertain, write the query with a comment noting which column to confirm.

### Step 2: Define Pass/Fail Thresholds

After the query block, define:

- **PASS:** Most recent game ingested within 30 days of today AND game count is within 20% of expected season volume (use League Hierarchy tier estimates for ECNL RL — Tier 2, national-level)
- **YELLOW:** Most recent game 30–60 days old OR game count 20–40% below expected
- **FAIL:** Most recent game >60 days old OR game count >40% below expected

For each status: write one line on what FORGE does next (PASS: no action; YELLOW: flag in next briefing; FAIL: escalate to SENTINEL immediately, halt Stage 2 planning for ECNL RL-affected leagues).

### Step 3: Write the Delivery Document

Deliverable: `02 - Tiger Tournaments/Projects/Infrastructure/ECNL RL Staleness Verification Query — April 2026.md`

Format:

```
## Context
[One paragraph: what ECNL RL is, why staleness matters, when to run this]

## Query
[SQL block — copy-paste ready]

## Expected Output Columns
[Table: column name | what it means]

## Pass/Fail Thresholds
[PASS / YELLOW / FAIL with criteria and next action]

## When to Run
[Recommended: next psql session after April 28. Takes ~2 minutes to run.]
```

## Definition of Done

- Query is copy-paste ready (no placeholders for schema — confirm against vault or note column to verify)
- Pass/Fail thresholds are explicit and numerical
- Document is self-contained: Presten can open it and run the query without reading any other doc
- Summary in briefing: "ECNL RL Staleness Query filed. PASS threshold: [N] days. Recommended run: next psql session."

## Why This Matters

ECNL RL is the highest-ranked gap in the Source Coverage Priority Matrix. Every session that passes without running this query is a session where SENTINEL may be authorizing decisions (Stage 2, Event Strength calibration) on top of an undetected ingestion failure. The query takes 2 minutes to run. The document exists to make sure it happens at the next opportunity, not the next time someone remembers to check.

## References

- `02 - Tiger Tournaments/Projects/Infrastructure/Source Coverage Priority Matrix — April 2026.md`
- `02 - Tiger Tournaments/Projects/Infrastructure/Source Health Monitoring Queries.md`
- `02 - Tiger Tournaments/Projects/Leagues/League Hierarchy.md`
