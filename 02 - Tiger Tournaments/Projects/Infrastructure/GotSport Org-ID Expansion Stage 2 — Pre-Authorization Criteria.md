---
title: GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria
tags: [infrastructure, gotsport, org-expansion, stage2, deployment, evo-draw, forge]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: ready — awaiting Stage 1 execution
task: task-2026-04-25-stage2-expansion-preauth-criteria
companion: "Infrastructure/Stale Team Backfill Runbook — April 2026.md"
---

# GotSport Org-ID Expansion — Stage 2 Pre-Authorization Criteria

**Prepared by:** FORGE  
**Date:** 2026-04-25  
**Purpose:** Define explicit authorization criteria for Stage 2 before Stage 1 runs, so SENTINEL can review the Stage 1 Results Report against measurable pass/fail conditions and authorize immediately — or hold off with a specific reason.

> [!warning] Gate Status
> Stage 2 is NOT authorized until SENTINEL explicitly states authorization in a briefing following Stage 1 Results Report review. This document defines what SENTINEL is reviewing and what authorization looks like.

---

## Section 1: Stage 2 Scope

### Org IDs in Stage 2 Cohort

Stage 2 adds all Priority 1 and Priority 2 USYS state association org IDs beyond the TX test cohort. Source: `Data Sources/GotSport USYS Org-ID Master List.md`.

| Priority | States | Estimated Org Entries | Notes |
|----------|--------|:---:|-------|
| P1 | CA (Cal North + Cal South), FL, NY (ENYYSA + WNYSA), NJ, WA, PA (EPYSA + WPYSA), OH (South + North) | ~11 | Large-state associations; multiple sub-state splits in CA, NY, PA, OH |
| P2 | CO, VA, MD, NC, GA, AZ, MA, TN, SC | ~9 | Medium-state associations; single org each |
| **Total P1+P2** | **~12 states** | **~20–25** | All require browser confirmation before adding to config — org IDs not yet confirmed |

**Note:** Org IDs in the Master List are currently `null` (lookup method documented, actual IDs pending Presten browser session). Stage 2 cannot begin until org IDs are confirmed AND Stage 1 gate conditions are met.

### Estimated Game Volume

| Estimate | Games |
|----------|:---:|
| Conservative (P1 states, off-peak) | 30,000–60,000 new games |
| Expected (P1+P2 states, spring peak season) | 60,000–120,000 new games |
| Maximum (all P1+P2 states at full coverage) | Up to 150,000 new games |

**Basis:** USYS State Cup/League unranked division gap estimated at 20,000–35,000 games total (Source Gap Inventory). Stage 2 covers ~10× the states in the TX test cohort; TX alone represents approximately 15–20% of total state volume due to player population size.

### Estimated Crawl Time

At Option B backoff rates (600ms/request, 3 workers):

| Phase | Estimate |
|-------|---------|
| Per-org discovery (team_ranking_data calls) | ~2–5 min per org × 25 orgs = ~1–2 hours |
| Game ingestion for newly discovered teams | 1,000–5,000 new teams × 600ms ÷ 3 workers = ~3–17 hours |
| **Total Stage 2 crawl time** | **~4–20 hours** (run overnight; split across 2 nights if >10 hours estimated) |

**Geographic coverage added:** CA, FL, NY, NJ, WA, PA, OH, CO, VA, MD, NC, GA, AZ, MA, TN, SC. Combined, these states account for the majority of US youth soccer registration outside Texas.

### Uncertainty

Estimated game volume and crawl time are uncertain until Stage 1 results confirm the game-per-org-ID ratio. The Stage 1 Results Report (filed after TX execution) will provide an empirical games/org-ID figure that enables a tighter Stage 2 estimate.

---

## Section 2: Stage 2 Authorization Gate

All five conditions below must be TRUE before Stage 2 runs. SENTINEL reviews each against the Stage 1 Results Report.

| # | Condition | Source Doc | Pass Criteria | Fail Action |
|---|-----------|------------|---------------|-------------|
| **G1** | Stage 1 game count stabilized | Stale Team Backfill Runbook — Stage 1 exit conditions | Second TX crawl returns 0 new games (delta = 0). First crawl established baseline; second crawl confirms no more data available from those org IDs. | Do NOT proceed. Run the second TX crawl. Re-evaluate after second crawl confirms stabilization. |
| **G2** | Zero duplicate games from Stage 1 | Pipeline Deployment Validation Spec — Section 2, Step 2 | The duplicate check query (`SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*) FROM games WHERE created_at > NOW() - INTERVAL '24 hours' GROUP BY ... HAVING COUNT(*) > 1`) returns **0 rows**. | Stop. File duplicate game IDs at `Reports/gotsport-org-expansion-test-[date].md`. Do not add any state org IDs until FORGE identifies root cause and confirms fix. |
| **G3** | No org-ID errors in Stage 1 | Stage 1 Results Report Section D | Zero TX org IDs returned 403, 404, or persistent empty payload. A single 429 handled by backoff is acceptable. | For each error org ID: investigate whether the org_id itself is wrong (confirm via browser lookup) or whether GotSport rejected the request pattern. Fix root cause before Stage 2. |
| **G4** | Option B backoff performed as designed | Reports/backoff-implementation-2026-04-24.md | Log shows `API_RETRY` events for any 429s, with `delay_ms` increasing on successive retries. SYNC_RUN_SUMMARY shows `teams_dlq` ≤ 2% of total teams attempted. No team hit max retries without recovery except known-invalid IDs. | If >2% DLQ rate: do NOT run Stage 2 at scale. The backoff implementation has a parameter problem. File for FORGE review. Correct backoff config before Stage 2. |
| **G5** | FORGE issued explicit Stage 2 authorization in Stage 1 Results Report | Stage 1 Results Report Section E | Section E reads: "Stage 1 exit conditions met. Stage 2 is authorized." | If Section E says "Stage 1 exit conditions NOT met": Stage 2 is blocked. Resolution step is stated in Section E. Complete resolution before returning to SENTINEL. |

---

## Section 3: Stage 2 Rollback Plan

### Halting a Running Stage 2 Crawl

If an issue is detected while Stage 2 is running:

1. Identify the crawl process: `ps aux | grep crawl-gotsport | grep -v grep`
2. Send SIGTERM to allow graceful shutdown: `kill -15 <PID>`
3. Wait 30 seconds. If the process does not exit: `kill -9 <PID>`
4. Confirm process has stopped before running any verification queries.

Partial Stage 2 ingestion is **acceptable** — games ingested before the halt are valid data. The crawl can be resumed from the point of interruption (excluding already-crawled orgs) once the issue is resolved.

### Detecting Duplicates Mid-Run

Run the duplicate detection query from `Infrastructure/Source Health Monitoring Queries.md` Section 3, scoped to the last 24 hours:

```sql
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*) AS count
FROM games
WHERE created_at >= NOW() - INTERVAL '24 hours'
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1
ORDER BY count DESC;
```

Run this query:
- Before Stage 2 starts (baseline should be 0 rows)
- 1 hour into Stage 2 (first sanity check)
- After each org-ID batch completes (if running in batches)
- Immediately if SYNC_RUN_SUMMARY shows unexpected DLQ growth

**If duplicates appear:** halt the crawl per the steps above, then run:

```sql
-- Remove duplicates introduced during Stage 2, keeping the most recently created record
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

After cleanup, re-run the duplicate check query to confirm 0 rows, then investigate the source org_id that produced duplicates before re-running Stage 2.

### Whether Partial Stage 2 Ingestion Requires Rollback

**Partial ingestion is safe.** The dedup pipeline (`dedup-and-link-teams.js`) runs globally and will handle any overlap between previously discovered teams and newly discovered teams. A partial Stage 2 run that processes 10 of 25 org IDs does not need to be rolled back — resume from the remaining org IDs on the next session.

**Full rollback is only required if:**
- Duplicates were created AND the cleanup query above fails to remove them cleanly (schema mismatch or index constraint error)
- A structural data integrity error is detected (e.g., games written with NULL team IDs)

In those cases: file a FORGE queue item with the exact error. Do not attempt to fix pipeline code without FORGE analysis.

---

## Section 4: SENTINEL Authorization Protocol

### What SENTINEL Reviews

SENTINEL reads the Stage 1 Results Report (`Reports/gotsport-org-expansion-stage1-results-[date].md`) and checks:

1. **Section E** — does it say "Stage 2 is authorized"? If yes, proceed. If no, read the reason and follow the stated resolution step.
2. **Section B (Duplicate Check)** — confirms the value is "0 rows — pass."
3. **Section C (Stage 1 Exit Condition)** — confirms second crawl delta = 0.
4. **Section D (Issues/Escalations)** — confirms no unresolved errors.

SENTINEL does not need to re-verify the underlying SQL queries. FORGE is responsible for accuracy of the Results Report. SENTINEL's role is to review the report and authorize or hold.

### What Authorization Looks Like

SENTINEL authorization is a single statement in SENTINEL's briefing, following the Stage 1 Results Report review:

> "Stage 1 Results Report reviewed. Gate conditions G1–G5: all pass. Stage 2 is authorized. Presten may proceed with full org-ID list per `Data Sources/GotSport USYS Org-ID Master List.md`. Org IDs must be confirmed via browser session before crawl runs."

If any gate condition fails, SENTINEL states:

> "Stage 1 gate condition [G#] not met: [reason]. Stage 2 is NOT authorized. Resolution required: [specific step from Section 2 Fail Action]. Re-evaluate after resolution."

### Authorization Timeline

| Scenario | Timeline |
|----------|----------|
| All gate conditions pass | SENTINEL authorizes in the same briefing session that reviews the Stage 1 Results Report. No waiting period. |
| One condition requires follow-up (e.g., second TX crawl not yet run) | Authorization deferred until the follow-up is complete and FORGE files an amended Results Report. |
| Duplicate check fails | Authorization blocked until root cause is identified, fix is confirmed, and FORGE re-files a clean Results Report. No timeline — fix must be complete first. |

---

## References

- `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — Stage exit conditions (G1 basis)
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — duplicate check query (G2 basis)
- `Data Sources/GotSport USYS Org-ID Master List.md` — Stage 2 org-ID scope
- `Infrastructure/Source Health Monitoring Queries.md` — duplicate detection query (Section 3)
- `task-2026-04-25-stage1-backfill-results-report.md` — Stage 1 Results Report that this document gates
- `10 - Agents/FORGE/Tasks/task-2026-04-24-gotsport-org-id-expansion.md` — org-ID expansion design

*FORGE — 2026-04-25*
