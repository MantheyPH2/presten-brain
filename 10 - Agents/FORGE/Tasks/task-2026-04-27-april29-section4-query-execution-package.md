---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/April 29 — Section 4 Source Baseline Query Package.md"
---

# Task: April 29 — Section 4 Source Baseline Query Package

## Context

FORGE's `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` has Section 4 as a placeholder that fills in April 29 after the April 28 DB session completes. The April 29 FORGE session checklist references this fill-in step. What does not exist: the copy-paste SQL that FORGE runs to produce Section 4 numbers.

On April 29, after Presten runs the April 28 DB session, FORGE has a narrow window to run these queries, record the baseline, and update the document before the May 1 deployment session begins. That window cannot start with a "what queries do I run?" design step — FORGE needs to enter April 29 with the queries already written and confirmed correct.

This task: write the complete, copy-paste-ready query package for Section 4.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/April 29 — Section 4 Source Baseline Query Package.md`

---

### Section 1: Pre-Run Checklist

Two-item gate before running queries:

- [ ] Presten has confirmed the April 28 DB session completed (GA ASPIRE UPDATE ran, ecnl_verified column altered, compute-rankings.js reran)
- [ ] April 29 psql session is open

If either is not confirmed: **do not run Section 4 queries.** Wait for Presten confirmation.

---

### Section 2: Source Ingestion Baseline Queries

The following queries populate Section 4 of `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md`. Run them in order. Record the output exactly — copy the numbers into the baseline document before closing psql.

**Query S4-1: Total game count by source**
```sql
-- Total games per source, sorted by volume
SELECT source, COUNT(*) AS game_count
FROM games
GROUP BY source
ORDER BY game_count DESC;
```
Record: source name, count. This establishes the pre-May-1 game volume baseline per source.

**Query S4-2: Active org-IDs in crawl config (GotSport)**
```sql
-- Count distinct org_ids that have contributed games in the last 90 days
SELECT COUNT(DISTINCT org_id) AS active_org_ids
FROM games
WHERE source = 'gotsport'
  AND game_date >= NOW() - INTERVAL '90 days';
```
Record: count. Cross-reference against the Org-ID Master Reference Active section count — should match.

**Query S4-3: Game recency by source**
```sql
-- Most recent game date per source
SELECT source, MAX(game_date) AS latest_game_date
FROM games
GROUP BY source
ORDER BY latest_game_date DESC;
```
Record: source, latest date. Flag any source where latest_game_date is > 30 days ago — potential stale source.

**Query S4-4: ecnl_verified column population status**
```sql
-- After April 28 ALTER TABLE: check column exists and population rate
SELECT
  COUNT(*) AS total_games,
  COUNT(ecnl_verified) AS ecnl_verified_populated,
  ROUND(100.0 * COUNT(ecnl_verified) / COUNT(*), 1) AS population_pct
FROM games
WHERE source = 'ecnl';
```
Record: total ECNL games, populated count, population %. If population % is 0 after April 28 ALTER, flag immediately — the ecnl_verified column exists but is not populated. This gates ELO's CP2 checkpoint.

**Query S4-5: Post-fix GA ASPIRE event_tier count**
```sql
-- Confirm GA ASPIRE event_tier fix applied
SELECT event_tier, COUNT(*) AS game_count
FROM event_tiers
WHERE event_tier IN ('ga', 'ga_aspire')
ORDER BY event_tier;
```
Record: ga count, ga_aspire count. After April 28 fix, ga_aspire should have games; any ASPIRE-tagged events previously in 'ga' should now be in 'ga_aspire'. If ga_aspire count = 0 after the fix session, escalate to SENTINEL immediately.

---

### Section 3: Output Filing Instructions

After running all 5 queries:

1. Open `02 - Tiger Tournaments/Projects/Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md`
2. Navigate to Section 4 (placeholder block)
3. Fill in each row with the query output
4. Record run date and time at the top of Section 4: `Filled: 2026-04-29 [HH:MM]`
5. File next FORGE briefing with: "Section 4 Source Baseline filled. GA ASPIRE fix confirmed [yes/no]. ecnl_verified population: [%]. Stale sources flagged: [list or 'none']."

---

### Section 4: Escalation Triggers

| Signal | Escalation |
|--------|-----------|
| GA ASPIRE count = 0 post-fix | Flag to SENTINEL immediately. April 28 DB session may not have run correctly. |
| ecnl_verified population = 0% | Flag to SENTINEL. CP2 is blocked; ELO April 29 checklist will fail at CP2. |
| Any source with latest_game_date > 60 days | Note in briefing as stale-source candidate. Do not escalate unless source was expected to be active. |
| Active org-ID count differs from Org-ID Master Reference | Reconcile discrepancy. File the delta in briefing. |

---

## Why This Matters

Section 4 of the Source Baseline is the only pre-May-1 snapshot of actual ingestion state after the April 28 fixes. ELO's G1–G4 gate framework references this baseline for calibration decisions. FORGE's Stage 2 authorization trigger document also requires Section 4 data (game volume pre-May-1 to compare against post-May-1). If FORGE enters April 29 without this query package, the Section 4 fill-in becomes an improvised step under session time pressure. Writing it now ensures April 29 is an execution session, not a design session.

## Definition of Done

- Query package written at the deliverable path
- All 5 queries are copy-paste ready (no pseudocode)
- Output filing instructions are unambiguous
- Escalation triggers are explicit
- FORGE can open this document on April 29 and follow it top to bottom without any additional preparation

## References

- `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` — document these queries populate
- `Infrastructure/April 29 FORGE Session Checklist.md` — this task is step 2 of that checklist
- `task-2026-04-26-stage2-authorization-trigger-document.md` — depends on Section 4 data
- ELO CP2 checkpoint (`task-2026-04-26-ecnl-migration-evidence-collection.md`) — depends on ecnl_verified query result
