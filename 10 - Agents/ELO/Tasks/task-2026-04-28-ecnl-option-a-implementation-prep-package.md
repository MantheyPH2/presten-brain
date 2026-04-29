---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: pending
priority: high
due: 2026-04-30
trigger: File regardless of Presten's ECNL decision; activate prep steps only after GO
deliverable: "02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — Option A Implementation Prep Package.md"
---

# Task: ECNL Migration — Option A Implementation Prep Package

## Context

ELO filed its ECNL migration recommendation on April 28: **Option A (preserve existing ECNL ratings, no rating reset)**. Presten needs to decide by April 30.

If Presten selects Option A, ELO should be able to execute on the same day — not spend a session figuring out what needs to happen. Right now, Option A implementation steps are scattered across the rating continuity spec, CP1/CP2 checkpoint tasks, and the ELO/FORGE handoff checklist. No single document consolidates them into an execution-ready package.

This task pre-stages that package. ELO writes it before Presten's decision. When decision comes, ELO executes same-session.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — Option A Implementation Prep Package.md`

---

### Section 1: What Option A Means (ELO Plain-Language Summary)

**Option A:** When ECNL teams migrate from old ECNL league to new ECNL RL structure (effective June 2), their existing ELO ratings carry forward unchanged. No reset. The `ecnl_verified` flag on games from the old structure is retained. Games played under ECNL RL after June 2 begin accumulating normally from the carried-forward rating baseline.

**What does NOT happen:** No rating reset to 1000. No blending formula. No "migration bonus."

**Calibration implication:** Girls ECNL and ECNL RL ratings will be continuous. The June 2 league-structure change is invisible to the rankings — teams keep their spot.

---

### Section 2: Pre-Conditions (All Must Be Met Before Option A Activates)

| Pre-condition | Status | How to Verify |
|--------------|--------|---------------|
| `ecnl_verified BOOLEAN DEFAULT FALSE` column exists in production | Pending April 28 session | psql: `\d games` or `\d team_games` |
| ecnl_verified backfill SQL run: historical ECNL games flagged correctly | Pending April 28/29 | Query: `SELECT COUNT(*) FROM games WHERE league_id IN (ECNL ids) AND ecnl_verified = TRUE` |
| ECNL CP1 staleness check: no stale ECNL games blocking carry-forward | Pending April 29 ELO session | `task-2026-04-28-ecnl-cp1-cp2-checkpoint-results-document.md` |
| ECNL CP2 rating continuity check: no orphaned team ratings | Pending April 29 | Same checkpoint document |
| FORGE ECNL migration pipeline impact confirmed (Option A path) | Complete ✅ | `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md` |

ELO cannot finalize this package until CP1/CP2 run. Fill status column after April 29 session.

---

### Section 3: Implementation Steps — Option A

Ordered sequence. Each step has an owner and a verification check.

**Step 1: Confirm ecnl_verified backfill complete (ELO + FORGE)**

- Owner: FORGE runs backfill; ELO confirms counts
- SQL verification: `SELECT COUNT(*) FROM games WHERE ecnl_verified = TRUE` → expected range from CP1 results
- Must complete before June 2

**Step 2: Update League Hierarchy — ECNL RL entries (ELO)**

- ECNL RL leagues must have calibration values matching current ECNL values
- Source: `League Hierarchy` vault note — Section for ECNL leagues
- Action: Add or update `ecnl_rl` entries with same tier values as outgoing ECNL entries
- Verification: Spot check 5 ECNL RL leagues against ECNL counterparts

**Step 3: Confirm team_merges for ECNL → ECNL RL team identity (ELO)**

- Some clubs change their GotSport org-ID when migrating to ECNL RL
- Source: `Rankings/Team Merges Audit — Presten Execution Package.md` — run ECNL-specific merge check
- Action: Any team with old ECNL ID that maps to new ECNL RL ID must have a `team_merge` entry
- Deadline: Before June 2 first ECNL RL games ingest

**Step 4: Pre-migration rating snapshot (ELO)**

- Date: June 1 (day before migration)
- Action: ELO files `Rankings/ECNL Pre-Migration Rating Snapshot.md` with top-100 ECNL girls teams and their ratings
- Purpose: Audit trail; if post-June-2 ratings diverge unexpectedly, compare against snapshot

**Step 5: Post-migration verification (ELO, within 1 week of June 2)**

- Run 3 spot-check queries: 
  1. Confirm top-10 ECNL teams retained ratings ± 5 pts post-migration
  2. Confirm no ECNL team has rating reset to 1000
  3. Confirm ECNL RL games ingesting with correct league tier weight

---

### Section 4: FORGE Coordination Required

| Item | FORGE Action | Timing |
|------|-------------|--------|
| ecnl_verified column already confirmed | FORGE confirms from April 28 session output | April 28/29 |
| ECNL RL org-IDs in GotSport config | FORGE verifies ECNL RL org-IDs are in `crawl-gotsport-api-parallel.js` | Before June 2 |
| ECNL migration day pipeline check | FORGE monitors June 2 pipeline run for ECNL RL game ingestion | June 2 |

---

### Section 5: If Presten Selects Option B Instead

If Presten selects Option B (rating reset): this package is superseded. ELO has a separate spec (`task-2026-04-24-ecnl-rating-continuity-spec.md`) that covers Option B implications. FORGE impact spec covers both paths.

File a note: "Option B selected — this package voided as of [date]."

---

### Section 6: Open Questions (ELO answers before April 30)

1. Does ECNL RL use the same GotSport org-IDs as existing ECNL? (Check with FORGE — FORGE has org-ID master reference)
2. Are there ECNL RL leagues not yet in League Hierarchy that need calibration values before June 2?
3. Does ecnl_verified apply to both boys and girls ECNL games, or girls only?

---

## Definition of Done

- Sections 1–6 populated (Section 2 status column filled after April 29 CP1/CP2 checks)
- Implementation steps have owners, verification checks, and deadlines
- FORGE coordination items clearly enumerated
- Open questions answered or flagged
- Filed at deliverable path by April 30

## Why This Matters

The ECNL migration is June 2 — 35 days away. The implementation has 5 ordered steps, some with hard June 1 deadlines. If ELO waits for Presten's decision before scoping implementation, the planning session eats a week of sprint time. Pre-staging this now means execution starts within hours of the decision. This is the same approach FORGE used with the EDP build plan — and it's why FORGE can deploy EDP within 30 minutes of receiving an org-ID.

## References

- `task-2026-04-28-ecnl-migration-option-recommendation.md` — ELO's Option A recommendation
- `task-2026-04-24-ecnl-rating-continuity-spec.md` — rating continuity technical spec
- `task-2026-04-27-ecnl-migration-option-b-technical-risk-deep-dive.md` — Option B risks (confirms Option A is lower risk)
- `task-2026-04-28-ecnl-migration-elo-forge-handoff-checklist.md` — FORGE coordination checklist
- `task-2026-04-28-ecnl-cp1-cp2-checkpoint-results-document.md` — CP1/CP2 checkpoint template (fills April 29)
