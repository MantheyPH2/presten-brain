---
title: "ECNL Migration — Option B FORGE Pipeline Impact Brief"
tags: [infrastructure, ecnl, migration, pipeline, forge, sentinel, option-b]
created: 2026-04-28
updated: 2026-04-28
author: FORGE
status: filed — awaiting option decision
task: task-2026-04-28-ecnl-option-b-forge-pipeline-impact-brief
due: 2026-04-29
---

# ECNL Migration — Option B FORGE Pipeline Impact Brief

> [!info] Purpose
> ELO recommends Option A (preserve ratings, no reset). Presten decides by April 30. This brief documents what Option B requires from the pipeline side so FORGE can respond same-session if Option B is chosen. This document does not advocate for either option — it states pipeline impact only.

**Filed by:** FORGE — 2026-04-28  
**ELO recommendation:** Option A (filed April 28)  
**Option decision deadline:** April 30  
**Presten action:** Read this brief before selecting Option B, then authorize via Section 5 block.  

---

## Section 1: Option B Definition

**Source:** `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md` — Section 3

Option B is **Hard Entity Replacement**. All historical ECNL game records in the `games` table are updated to replace old TGS team IDs with new TGS team IDs, effective June 2, 2026. Old ECNL team entities are retired (`is_active = FALSE`) in the `teams` table. Forward ingestion (post-June-2 TGS scrape) is natively correct — new game records arrive with new team IDs.

Tables directly affected:
- `games` — UPDATE `home_team_id` / `away_team_id` for all ECNL source rows
- `team_merges` — UPDATE both `team_id` and `merged_team_id` for all rows referencing old ECNL IDs
- `teams` — UPDATE `is_active = FALSE` for all retiring ECNL team records

Cutover date: **June 2, 2026**

**What Option B is NOT:** Option B does not change the `ecnl_verified` flag logic, does not require schema DDL changes (no new tables, no new columns beyond `ecnl_verified` added April 28), and does not require TGS scraper code changes.

---

## Section 2: FORGE Pipeline Changes Required for Option B

| Question | Answer | Source |
|----------|--------|--------|
| Does Option B require schema changes beyond `ecnl_verified`? | **NO** — Option B is data updates only. No new tables, no new columns. `ecnl_verified` column added April 28 covers all options. | FORGE Pipeline Impact Spec, Section 3 |
| Are there pipeline game ingestion changes required? | **NO** — TGS scraper accepts new team IDs natively post-June-2. No scraper code changes required under Option B. | FORGE Pipeline Impact Spec, Section 3 — Ingestion Path Changes |
| Does the archive dry-run interact with the reset? (timing risk) | **YES — timing risk.** Option B marks old ECNL team records `is_active = FALSE`. If archive dry-run runs BEFORE migration, old ECNL teams become archive candidates prematurely. FORGE must sequence: migration first → archive dry-run after. | FORGE Pipeline Impact Spec, Section 3 + Archive SQL Fix Verification |
| Does Stage 1 launch on May 1 create any Option B conflicts? | **NO.** Stage 1 is TYSA GotSport crawl. TYSA is not ECNL. Option B affects TGS source records only. Stage 1 is ECNL-independent by design. | Config Final Review, Section 3 |
| Does the FM1/FM2 activation path change under Option B? | **NO.** FM2 fix (TGS scraper writes `ecnl_verified = TRUE` at ingestion time) applies regardless of option. Post-UPDATE verification confirms `ecnl_verified` count is unchanged. If any row loses the flag, Step 2b backfill SQL re-runs to restore it. | FORGE Pipeline Impact Spec, Section 3 — ecnl_verified Flag Behavior |

---

## Section 3: Option B vs Option A — FORGE Effort Delta

| Item | Option A FORGE Work | Option B FORGE Work | Delta |
|------|--------------------|--------------------|-------|
| Schema changes | 1 column add (`ecnl_pre_restructure_team_id`) | 0 new columns, 0 new tables | Option A slightly more DDL |
| Pipeline code changes | None | None | Equal |
| Data backfill | INSERT team_merges rows (one per restructured team); UPDATE ecnl_pre_restructure_team_id column | UPDATE all ECNL game records (home_team_id, away_team_id); UPDATE all team_merges referencing old ECNL IDs; UPDATE teams.is_active | **Option B significantly more write volume** |
| Coordination with ELO | ELO pre-migration snapshot (June 1) + post-migration verification | ELO pre-migration snapshot MUST complete BEFORE FORGE executes Step 4. Hard sequencing dependency — cannot proceed out of order. | **Option B has a hard sequencing gate that Option A does not** |
| Timeline to June 2 readiness | 2–3 days from authorization to first validated run | 4–5 days from authorization to first validated run | **Option B requires 2 more days** |
| Rollback | Low complexity — team_merges inserts can be reversed without touching game records | Higher complexity — requires re-running UPDATE in opposite direction; FORGE retains old→new mapping file for rollback | **Option B rollback is harder** |

---

## Section 4: FORGE Position on Option B Feasibility

Option B is **feasible within the June 2 deadline** if authorized by May 2. The 4–5 day implementation window is comfortable given 31 days remain until June 2. The critical path constraint is not FORGE's execution time — it is ELO's rating continuity pre-migration snapshot, which must complete before FORGE executes the `games` table UPDATE (Step 4 of Option B). This is a non-negotiable sequencing gate: executing Step 4 before ELO clears this gate will break rating lookups for all ECNL teams until ELO catches up. SENTINEL must confirm ELO gate is cleared before authorizing FORGE execution. If ELO decision is delayed past May 9 (DSS readiness check): FORGE flags to SENTINEL, but Option B's 4–5 day window remains feasible before June 2 even at that date.

One additional feasibility risk: The UPDATE volume across all ECNL game records may be large. FORGE will test with a single team pair first, confirm output, then execute the full batch. No estimate is possible without knowing total ECNL game count, but this is a standard mitigation for any bulk UPDATE.

---

## Section 5: FORGE Authorization Block (for SENTINEL — fill if Option B selected)

```
SENTINEL: If Presten selects Option B, FORGE requires:

[ ] ELO written confirmation of Option B scope
    (specific tables affected, reset logic, old→new team ID mapping delivery date)

[ ] SENTINEL authorization for execution of Option B Step 4
    (game record UPDATE — this is the high-consequence, partially-irreversible step)

[ ] ELO rating continuity pre-migration snapshot confirmed complete
    (FORGE will not execute Step 4 until ELO clears this gate in writing)

[ ] Revised June 2 execution sequence from ELO
    (ELO must file updated coordination timeline showing FORGE–ELO step ordering)

SENTINEL VERDICT ON OPTION B:
[ ] FORGE authorized to proceed — ELO gate must be confirmed before Step 4
[ ] HOLD — reason: ___________

Signed: SENTINEL
Date: ___________
```

> [!warning] FORGE Pre-Execution Gate (Option B only)
> FORGE will not execute Step 4 (UPDATE games table) under any circumstances until:
> 1. All four items above are checked by SENTINEL
> 2. ELO explicitly confirms rating continuity pre-migration snapshot is complete
> This gate is non-negotiable. The risk of out-of-order execution is rating system breakage for all ECNL teams — irreversible without a coordinated re-run.

---

## References

- `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md` — Option B detail (Section 3), shared prerequisites (Section 5), SENTINEL authorization gate (Section 6)
- `10 - Agents/ELO/Tasks/task-2026-04-28-ecnl-migration-option-recommendation.md` — ELO's Option A recommendation
- `10 - Agents/ELO/Tasks/task-2026-04-28-ecnl-option-a-implementation-prep-package.md` — Option A reference (contrasts with Option B)
- `10 - Agents/ELO/Tasks/task-2026-04-28-ecnl-migration-elo-forge-handoff-checklist.md` — ELO–FORGE coordination checklist
- `Infrastructure/Archive Inactive Teams — SQL Fix Verification and Dry-Run Readiness.md` — archive timing risk cross-reference

*FORGE — 2026-04-28*
