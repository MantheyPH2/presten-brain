---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-29
priority: medium
due: 2026-04-30 EOD
status: completed
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/ECNL Migration — Option 1 Implementation Card.md"
completed: 2026-04-29
topic: ecnl-option1-forge-implementation-card
---

# Task: ECNL Option 1 — FORGE Implementation Card

## Context

ELO has filed the `Rankings/ECNL Migration Option — Decision Brief.md` recommending **Option 1 — No Re-tag**. Presten must authorize by April 30 EOD. SENTINEL will review and authorize or override on April 30.

If Option 1 is authorized (likely), FORGE needs to know exactly what to do and when. The existing pipeline impact documents (`task-2026-04-28-ecnl-migration-pipeline-impact-spec.md`, `task-2026-04-28-ecnl-option-b-forge-pipeline-impact-brief.md`) cover Option B (Option 2) and general pipeline impact. There is no clean, specific, ready-to-execute card for **Option 1**.

This task fixes that.

---

## What to Build

Create `Infrastructure/ECNL Migration — Option 1 Implementation Card.md`

One page. Action-card format. Covers exactly what FORGE does if SENTINEL authorizes Option 1 (No Re-tag).

---

## Required Content

### What Option 1 Means for FORGE

In 3–5 bullets: what FORGE does NOT need to do under Option 1, and what FORGE DOES need to do. No re-derivation of ELO's analysis — FORGE's scope only.

Expected content:
- No schema changes (no ALTER TABLE for ecnl_legacy tier)
- No UPDATE SQL for team tier re-labeling
- Pipeline ingestion continues unchanged on migration date
- FORGE's role is monitoring: confirm game ingestion from ECNL migrated teams is uninterrupted post-June 1
- Any config adjustments required (if ECNL migrated teams shift org IDs or source identifiers)

### Implementation Timeline

| Date | FORGE Action | Trigger |
|------|-------------|---------|
| April 30 EOD | Authorization received | SENTINEL/Presten decision |
| [date] | Config verification (confirm ECNL team identifiers unchanged in GotSport post-migration) | [FORGE fills] |
| June 1 | Migration day monitoring | ECNL migration executes |
| June 1 + 24 hrs | File ECNL migration ingestion confirmation | Post-migration first run |
| June 1 + 72 hrs | File 3-day post-migration source health check | Pipeline stability |

### What FORGE Monitors on June 1

List the specific metrics FORGE checks on migration day to confirm Option 1 is working:
- Game count continuity for ECNL teams (before vs. after migration)
- No orphaned team records in source data
- Source identifier integrity (org IDs still resolve correctly)

### If Migration Fails (Option 1 Fallback)

1-paragraph contingency: what FORGE does if ECNL migration causes ingestion failures under Option 1. Who FORGE notifies, what the escalation path looks like.

### Handoff to ELO

What FORGE signals to ELO when migration ingestion is confirmed:
- The specific briefing language FORGE uses
- What ELO needs from FORGE to run CP1 and CP2

---

## Deliverable

`Infrastructure/ECNL Migration — Option 1 Implementation Card.md`

Ready to execute the moment SENTINEL authorizes. One-page card format — no research needed, just FORGE's action steps. Confirm delivery in briefing.
