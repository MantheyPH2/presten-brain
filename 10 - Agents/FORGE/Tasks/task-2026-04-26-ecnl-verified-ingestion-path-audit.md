---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md"
---

# Task: ecnl_verified Ingestion Path Audit

## Context

The Step 2b backfill SQL (`UPDATE games SET ecnl_verified = TRUE WHERE source IN ('tgs', 'tgs_athleteone') AND event_tier = 'ecnl'`) fixes all ECNL games currently in the database. This runs on April 28.

Starting May 1, the daily pipeline ingests new games every day. When a new ECNL game is written to the `games` table via the pipeline, does it receive `ecnl_verified = TRUE` automatically?

If not: every new ECNL game ingested after April 28 will have `ecnl_verified = FALSE` (the column default). The historical backfill is complete, but the column is permanently inaccurate for all future data. ECNL migration logic built on `ecnl_verified` will have a growing body of ECNL games that are silently excluded.

This audit is the forward-looking counterpart to the Step 2b backfill.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md`

---

### Section 1: Pipeline Write Path for ECNL Games

- Where in the pipeline code does a game row get written to the `games` table?
- Which file contains the INSERT or upsert logic?
- What columns does the write include? Is `ecnl_verified` in the column list?

---

### Section 2: ecnl_verified Assignment Logic

Check whether the pipeline explicitly sets `ecnl_verified` at write time:

- **If `ecnl_verified` is set at write time:** Quote the exact logic. Confirm it uses the same conditions as the Step 2b backfill (`source IN ('tgs', 'tgs_athleteone') AND event_tier = 'ecnl'`). If the conditions differ, note the discrepancy.
- **If `ecnl_verified` is NOT set at write time:** The column relies entirely on the one-time backfill. All new ECNL games will default to `FALSE`. Document this as a gap.

---

### Section 3: Audit Verdict — One of Three Outcomes

**Outcome A: Pipeline sets `ecnl_verified = TRUE` for new ECNL games**
- Quote the code.
- Confirm it is consistent with the backfill conditions.
- No action needed. SENTINEL clears this risk.

**Outcome B: Pipeline does not set `ecnl_verified`**
- Document the gap.
- Propose the minimum code addition: at game write time, set `ecnl_verified = (source IN ('tgs', 'tgs_athleteone') AND event_tier = 'ecnl')` 
- Estimate severity: how many new ECNL games are expected per week from May 1 onward? If game volume is low, impact is minor short-term. If ECNL has active play in May, the unset field grows rapidly.
- Recommend this fix be included in the May 1 deployment, not deferred.

**Outcome C: Code not accessible or location unknown**
- Flag to SENTINEL immediately.
- Write: "Pipeline write path not audited. SENTINEL must request Presten confirm ecnl_verified assignment before May 1 deployment."

---

### Section 4: Interaction with ECNL Migration

The ECNL Migration Rating Continuity Spec (ELO, filed April 25) examines how ECNL team ratings behave post-June 1 migration. That spec depends on `ecnl_verified` accurately reflecting historical ECNL games. If the column is only accurate for pre-April-28 games (not new games), the migration spec's Option 2 (re-tag historical games) becomes less useful, because the distinction between "verified historical ECNL" and "unverified new ECNL" is muddy.

Note in this section whether the migration spec's assumptions are affected by the audit outcome.

---

## Quality Standard

- Section 3 must have a single clear verdict (A, B, or C).
- If Outcome B: the code addition must be specific enough for Presten to implement in one session.
- File by April 29 so SENTINEL can decide whether to flag this as a May 1 deployment blocker.

## Why This Matters

`ecnl_verified` is the deduplication anchor for ECNL game overlap between TGS AthleteOne and TGS RL. If new games from May 1 onward are not being marked correctly, the dedup logic will gradually lose accuracy. This is not visible in April 29 analysis (which uses only pre-April-28 data). It becomes visible in May 8 first weekly health review — at which point fixing it requires another backfill of a larger dataset.

## References

- `Infrastructure/Step 2b — ecnl_verified Backfill SQL Execution Package.md` — the one-time backfill this audit complements
- `Infrastructure/April 28 Execution Log.md` — Step 2b is now logged; this audit closes the forward-looking gap
- `task-2026-04-26-ga-aspire-pipeline-classifier-audit.md` — parallel audit for GA ASPIRE classification
- `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` — ELO's migration spec that depends on ecnl_verified accuracy
