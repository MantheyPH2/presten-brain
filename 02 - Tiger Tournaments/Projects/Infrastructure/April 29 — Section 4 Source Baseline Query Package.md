---
title: April 29 — Section 4 Source Baseline Query Package
tags: [infrastructure, pipeline, baseline, sql, april29, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: ready — execute April 29 after April 28 DB session confirms
task: task-2026-04-27-april29-section4-query-execution-package
---

# April 29 — Section 4 Source Baseline Query Package

> Copy-paste-ready SQL for populating Section 4 of `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md`. Run on April 29 after confirming the April 28 DB session completed. This is an execution document — open, run top to bottom, record output.

---

## Section 1: Pre-Run Checklist

**Gate before running any queries below. Both must be confirmed.**

- [ ] Presten has confirmed the April 28 DB session completed (GA ASPIRE UPDATE ran, `ecnl_verified` column altered, `compute-rankings.js` reran, April 28 Execution Log filled)
- [ ] April 29 psql session is open and connected to the production database

If either is not confirmed: **do not run Section 2 queries.** Wait for Presten confirmation. Running queries before April 28 completes will produce a pre-fix baseline that does not reflect the actual post-fix state.

---

## Section 2: Source Ingestion Baseline Queries

Run in order. Record output exactly. Copy numbers into `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` Section 4 before closing psql.

---

### Query S4-1: Total Game Count by Source

```sql
-- Total games per source, sorted by volume
SELECT source, COUNT(*) AS game_count
FROM games
GROUP BY source
ORDER BY game_count DESC;
```

**Record:** source name + count for every row. This establishes the pre-May-1 game volume baseline per source.

**Flag if:** Any source with zero games that was expected to be active. Any source name that looks like a test or duplicate entry.

---

### Query S4-2: Active Org-IDs in Crawl Config (GotSport)

```sql
-- Count distinct org_ids that have contributed games in the last 90 days
SELECT COUNT(DISTINCT org_id) AS active_org_ids
FROM games
WHERE source = 'gotsport'
  AND game_date >= NOW() - INTERVAL '90 days';
```

**Record:** count. Cross-reference against the Org-ID Master Reference Active section count — should match.

**Flag if:** Active org-ID count differs from the Org-ID Master Reference Section 1 row count. Reconcile the discrepancy and file the delta in the briefing.

---

### Query S4-3: Game Recency by Source

```sql
-- Most recent game date per source
SELECT source, MAX(game_date) AS latest_game_date
FROM games
GROUP BY source
ORDER BY latest_game_date DESC;
```

**Record:** source + latest date for every row.

**Flag if:** Any source where `latest_game_date` is > 30 days ago — potential stale source. Note in briefing as stale-source candidate. Do not escalate unless source was expected to be active.

---

### Query S4-4: ecnl_verified Column Population Status

```sql
-- After April 28 ALTER TABLE: check column exists and population rate
SELECT
  COUNT(*) AS total_games,
  COUNT(ecnl_verified) AS ecnl_verified_populated,
  ROUND(100.0 * COUNT(ecnl_verified) / COUNT(*), 1) AS population_pct
FROM games
WHERE source IN ('tgs', 'tgs_athleteone');
```

**Record:** total TGS/AthleteOne games, populated count, population %.

**Flag if:** Population % is 0 after April 28 ALTER TABLE + Step 2b backfill UPDATE — the `ecnl_verified` column exists but is not populated. This gates ELO's CP2 checkpoint. Escalate to SENTINEL immediately if population is 0%.

> [!note] Step 2b dependency
> Query S4-4 only reflects correct data if Step 2b (ecnl_verified backfill UPDATE) ran during the April 28 session. Check April 28 Execution Log Step 2b row before interpreting this result.

---

### Query S4-5: Post-Fix GA ASPIRE Event Tier Count

```sql
-- Confirm GA ASPIRE event_tier fix applied
SELECT event_tier, COUNT(*) AS game_count
FROM games
WHERE event_tier IN ('ga', 'ga_aspire')
   OR event_name ILIKE '%ga aspire%'
   OR event_name ILIKE '%ga_aspire%'
GROUP BY event_tier
ORDER BY event_tier;
```

**Record:** event_tier + game_count for every row returned.

**Flag if:** `ga_aspire` count = 0 after the April 28 GA ASPIRE UPDATE. If zero, escalate to SENTINEL immediately — the April 28 UPDATE may not have run. Additionally, flag if any games with `event_name ILIKE '%ga aspire%'` still show `event_tier = 'ga'` (indicates the fix was incomplete).

---

## Section 3: Output Filing Instructions

After running all 5 queries:

1. Open `02 - Tiger Tournaments/Projects/Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md`
2. Navigate to Section 4 (placeholder block)
3. Fill in each table row with the query output — copy numbers directly, do not round or estimate
4. At the top of Section 4, record: `Filled: 2026-04-29 [HH:MM]`
5. File the Section 4 Summary Block:

```
Section 4 Status: [All checks PASS / Query S4-N returned unexpected result — see notes]
Post-April-28 baseline confirmed: [YES / NO]
GA ASPIRE fix confirmed: [YES — ga_aspire rows present / NO — escalated to SENTINEL]
ecnl_verified population: [N%]
Stale sources flagged: [list or "none"]
Filed by FORGE: 2026-04-29 [HH:MM]
```

6. File the next FORGE briefing with: "Section 4 Source Baseline filled. GA ASPIRE fix confirmed [yes/no]. ecnl_verified population: [%]. Stale sources flagged: [list or 'none']."

---

## Section 4: Escalation Triggers

| Signal | Escalation |
|--------|-----------|
| GA ASPIRE count = 0 post-fix (S4-5) | Flag to SENTINEL immediately. April 28 DB session may not have applied the UPDATE. Do not proceed to May 1 launch without SENTINEL confirmation. |
| ecnl_verified population = 0% (S4-4) | Flag to SENTINEL immediately. CP2 is blocked; ELO April 29 checklist will fail at CP2. |
| Any source with `latest_game_date` > 60 days (S4-3) | Note in briefing as stale-source candidate. Do not escalate unless source was expected to be active in the last 60 days. |
| Active org-ID count differs from Org-ID Master Reference (S4-2) | Reconcile discrepancy. File the delta in briefing. Do not escalate unless delta is > 5 org-IDs. |
| Query returns an error or unexpected schema error | Stop. Do not proceed to next query. File error text in briefing and flag to SENTINEL. |

---

## References

- `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` — document these queries populate (Section 4)
- `Infrastructure/April 29 FORGE Session Checklist.md` — this query package is Step 2 of that checklist
- `Infrastructure/April 28 Execution Log.md` — confirms April 28 steps ran; cross-check before running S4-4 and S4-5
- `Infrastructure/Stage 2 Crawl Expansion — Authorization Trigger Document.md` — depends on Section 4 game volume data
- ELO CP2 checkpoint — depends on ecnl_verified query result (S4-4)

*FORGE — 2026-04-26*
