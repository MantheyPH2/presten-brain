---
title: Stage 2 Expansion — SENTINEL Authorization Gate
tags: [infrastructure, gotsport, org-expansion, stage2, sentinel, authorization, evo-draw, forge]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: pending — all 6 conditions unmet as of 2026-04-25
task: task-2026-04-25-stage2-authorization-gate-document
---

# Stage 2 Expansion — SENTINEL Authorization Gate

> **Purpose:** Single-document Stage 2 gate. FORGE updates Section 1 as conditions clear. SENTINEL reviews Section 2 and signs Section 4 when all conditions show ✅. Replaces multi-document synthesis.

---

## Section 1: Authorization Status Summary

```
Stage 2 Authorization Status: PENDING
Last updated: 2026-04-25
Conditions met: 0 / 6
Blocking conditions:
  - C1: Stage 1 TX crawl not yet executed
  - C2: Stage 1 results not yet available (no Results Report)
  - C3: Browser session not yet completed — all USYS state org IDs null
  - C4: May 1 Pre-Authorization Checklist not yet verified complete
  - C5: Org-ID Master Reference Section 1 has no confirmed entries
  - C6: No Stage 1 morning-after results (Stage 1 not yet run)
```

---

## Section 2: Authorization Conditions

FORGE updates the Status column when any condition changes. All 6 must show ✅ before SENTINEL reviews for authorization.

| # | Condition | Source Document | Status | How FORGE Confirms |
|---|-----------|-----------------|--------|-------------------|
| **C1** | Stage 1 TX crawl completed and Results Report filed | `Infrastructure/Stage 1 Backfill Results Report.md` | ❌ PENDING | FORGE confirms file exists with game count and duplicate check data (Sections B–E populated) |
| **C2** | Stage 1 results show GREEN: second TX crawl delta = 0, zero duplicates, Section E says "Stage 2 is authorized" | Stage 1 Results Report — Sections B, C, E | ❌ PENDING | Section E reads exactly: "Stage 1 exit conditions met. Stage 2 is authorized." AND duplicate check returns 0 rows AND second crawl delta = 0 |
| **C3** | Browser session completed — USYS state org IDs confirmed for ≥ 5 of 7 Priority 1 states (CA, FL, NY, NJ, WA, PA, OH) | `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` Section 2 | ❌ PENDING | ≥ 5 rows in Section 2 updated from `pending-browser-lookup` to `confirmed-[org_id]` for the P1 USYS states listed |
| **C4** | May 1 Pre-Authorization Checklist shows PASS on all Stage 1 gate items | `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` | ❌ PENDING | Checklist filed with all Stage 1 gate items checked ✅; no items marked ❌ or DEFERRED |
| **C5** | Org-ID Master Reference Section 1 updated with all browser-session-confirmed IDs (no null org_id values for confirmed entities) | `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` Section 1 | ❌ PENDING | Section 1 row count ≥ 1; all confirmed entities have actual org_id values (not `pending-lookup` or `null`) |
| **C6** | No active YELLOW or RED pipeline errors from the Stage 1 morning-after validation | `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` (Stage 1 run results) | ❌ PENDING | Morning-After Runbook filed for Stage 1 run with overall status GREEN; no unresolved YELLOW or RED items |

> [!warning] Do not remove any of the 6 conditions above without explicit SENTINEL authorization. Additional conditions may be added by SENTINEL at any time.

---

## Section 3: Stage 2 Scope — What Gets Added

Org IDs below enter the crawl config upon Stage 2 authorization. All P1 state IDs are pending browser session (C3). FORGE updates this table when C3 clears.

**Priority 1 States (C3 gate — browser session required)**

| Entity | Org-ID | Config Section | Auth Condition |
|--------|--------|----------------|----------------|
| Cal North Soccer (USYS CA-NorCal) | TBD — pending browser session | `// USYS State Associations` | C3 |
| Cal South Soccer (USYS CA-SoCal) | TBD — pending browser session | `// USYS State Associations` | C3 |
| Florida Youth Soccer Assoc (FYSA) | TBD — pending browser session | `// USYS State Associations` | C3 |
| Eastern NY Youth Soccer (ENYYSA) | TBD — pending browser session | `// USYS State Associations` | C3 |
| NJ Youth Soccer Assoc (NJYSA) | TBD — pending browser session | `// USYS State Associations` | C3 |
| Washington Youth Soccer (WSYSA) | TBD — pending browser session | `// USYS State Associations` | C3 |
| Eastern PA Youth Soccer (EPYSA) | TBD — pending browser session | `// USYS State Associations` | C3 |

**Priority 2 States (pending browser session + Stage 2 authorization)**

| Entity | Org-ID | Config Section | Auth Condition |
|--------|--------|----------------|----------------|
| Colorado Youth Soccer | TBD | `// USYS State Associations` | C3 + C1/C2 |
| Virginia Youth Soccer Assoc | TBD | `// USYS State Associations` | C3 + C1/C2 |
| Maryland Youth Soccer Assoc | TBD | `// USYS State Associations` | C3 + C1/C2 |
| NC Youth Soccer Assoc | TBD | `// USYS State Associations` | C3 + C1/C2 |
| Georgia Soccer (GYSA) | TBD | `// USYS State Associations` | C3 + C1/C2 |
| Arizona Youth Soccer Assoc | TBD | `// USYS State Associations` | C3 + C1/C2 |
| Massachusetts Youth Soccer Assoc | TBD | `// USYS State Associations` | C3 + C1/C2 |
| Tennessee Soccer Assoc | TBD | `// USYS State Associations` | C3 + C1/C2 |
| South Carolina Youth Soccer | TBD | `// USYS State Associations` | C3 + C1/C2 |

**League entities (pending browser session — may also be added at Stage 2)**

| Entity | Org-ID | Config Section | Auth Condition |
|--------|--------|----------------|----------------|
| USL Academy | TBD — pending browser session | `// USL Academy` | C3 (browser session confirms GotSport presence) |
| EDP Soccer (Eastern Development Program) | TBD — pending browser session | `// EDP Soccer` | C3 (browser session confirms GotSport presence) |
| Elite 64 | TBD — pending browser session | `// Elite 64` | C3 (conditional — only if browser session confirms GotSport org_id) |

> [!note] All TBD entries above require browser session completion (C3) before being added to the crawl config. FORGE updates this table from the Org-ID Master Reference after the session runs.

**Estimated crawl impact at Stage 2:**

| Estimate | Games |
|----------|-------|
| Conservative (P1 states, off-peak) | 30,000–60,000 new games |
| Expected (P1+P2 states, spring peak) | 60,000–120,000 new games |
| Crawl time at Option B backoff (600ms, 3 workers) | ~4–20 hours; plan for overnight run split across 2 nights if >10 hrs |

---

## Section 4: Authorization Trigger

When all 6 conditions in Section 2 show ✅, FORGE executes the following sequence:

1. FORGE updates Section 1 Status Summary to `AUTHORIZED` with the date
2. FORGE files a briefing note: "Stage 2 gate cleared. Conditions C1–C6 all ✅. Awaiting SENTINEL authorization to add org IDs to crawl config."
3. SENTINEL reviews Section 2, confirms all ✅, and completes the Authorization Block below
4. FORGE adds Stage 2 org IDs to `crawl-gotsport-api-parallel.js` per Section 3 and the `Infrastructure/GotSport Org-ID Config Edit Reference.md`
5. FORGE updates Section 1 after config changes are applied: `Stage 2 Authorization Status: AUTHORIZED AND DEPLOYED`

**SENTINEL Authorization Block** *(SENTINEL fills — do not pre-fill)*

```
SENTINEL authorization: AUTHORIZED / HELD
Authorization date: ___________
SENTINEL note: ___________
If HELD, blocking condition: ___________
```

---

## Section 5: Rollback Protocol

**If Stage 2 crawl must be halted mid-run:**

1. Identify the crawl process: `ps aux | grep crawl-gotsport | grep -v grep`
2. Send SIGTERM for graceful shutdown: `kill -15 <PID>`
3. Wait 30 seconds. If the process does not exit: `kill -9 <PID>`
4. Confirm process has stopped before running verification queries.

Partial Stage 2 ingestion is safe — games ingested before the halt are valid data. Resume from remaining org IDs once the issue is resolved.

**If duplicates appear mid-run** (run duplicate detection query from `Infrastructure/Source Health Monitoring Queries.md` Section 3):

```sql
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*) AS count
FROM games
WHERE created_at >= NOW() - INTERVAL '24 hours'
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1
ORDER BY count DESC;
```

If duplicates appear: halt the crawl, then run:

```sql
DELETE FROM games WHERE id IN (
  SELECT id FROM (
    SELECT id,
      ROW_NUMBER() OVER (
        PARTITION BY home_team_id, away_team_id, game_date
        ORDER BY created_at DESC
      ) as rn
    FROM games
    WHERE created_at >= NOW() - INTERVAL '48 hours'
  ) ranked
  WHERE rn > 1
);
```

After cleanup, confirm 0 duplicate rows, then investigate the source org_id before resuming.

**Full rollback is only required if:**
- Duplicates cannot be cleaned by the query above (schema mismatch or constraint error)
- A structural data integrity error is detected (games written with NULL team IDs)

Stage 2 failure does not roll back Stage 1. They are independently deployable.

---

## References

- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — source conditions (G1–G5 → translated to C1–C5 above; C6 added per FORGE)
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — org-ID status source of truth
- `Infrastructure/Stage 1 Backfill Results Report.md` — C1/C2 source (pending execution)
- `Infrastructure/May 1 Deployment Pre-Authorization Checklist.md` — C4 source
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — C6 source
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — how to add org IDs to config
- `Infrastructure/Source Health Monitoring Queries.md` — duplicate detection queries (Section 3)

*FORGE — 2026-04-25*
