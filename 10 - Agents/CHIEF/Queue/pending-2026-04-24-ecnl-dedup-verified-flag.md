---
type: agent-queue
agent: CHIEF
category: decision
priority: urgent
date: 2026-04-24
status: pending
answer: ""
recommendation: "CHIEF recommends assigning this P0 task to FORGE immediately. ELO identified the bug; SENTINEL confirmed severity (500-2k teams at risk); FORGE has the code expertise. Five-minute fix with June 1 deadline."
---

# Task Assignment: ECNL Dedup `verified = true` Flag — P0 BUG

## Context

ELO's June 1 Risk Assessment identified a critical code bug in `ecnl-transition-dedup.js`. On June 1, when this script inserts team merge records to map TGS team IDs → GotSport team IDs, it must include `verified = true` in the INSERT statement.

**Why it matters:** The Ranking Engine Phase 1 only loads verified merges. Without this flag, every ECNL team whose platform ID changes on June 1 silently loses its entire game history and drops to ~1000 rating overnight.

**Scope:** 500–2,000 ECNL teams. **Timing:** June 1, 2026 (37 days out).

## The Bug

Currently, `ecnl-transition-dedup.js` INSERT statement is missing `verified = true`:

```sql
-- CURRENT (BROKEN)
INSERT INTO team_merges (tgs_id, gotsport_id, dedup_method)
VALUES ($1, $2, 'ecnl-transition');

-- SHOULD BE
INSERT INTO team_merges (tgs_id, gotsport_id, dedup_method, verified)
VALUES ($1, $2, 'ecnl-transition', true);
```

## Assignment

This is a FORGE task. FORGE has access to the pipeline codebase and handled all other dedup/migration work. The fix is five lines of code.

## Your Decision

**Assign to FORGE?** Yes / No / Other

If yes, CHIEF will create the formal task file immediately.

---

*CHIEF — escalated from ELO's June 1 Risk Assessment*
