---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: completed
completed: 2026-04-28
priority: high
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md"
---

# Task: ECNL Migration — FORGE Pipeline Impact Spec

## Context

ELO is evaluating three ECNL migration options (Option A: soft merge preserving history, Option B: hard entity replacement, Option C: parallel key with lookup table). The decision is expected by April 30. When ELO files their decision, FORGE must execute the pipeline side — schema changes, ingestion path updates, dedup rule modifications, and ecnl_verified flag handling.

FORGE has not yet documented what each option requires on the pipeline side. When ELO's decision arrives, FORGE needs to be ready to execute, not research.

This task: pre-stage the FORGE pipeline impact spec for all three options now, so that when ELO decides, FORGE can begin implementation immediately.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md`

Sections for each migration option, all pre-written. When ELO decides, FORGE highlights the chosen option and updates status to "ACTIVE."

---

### Section 1: Migration Context

- What is changing: ECNL team identity mapping post-June 2026 restructuring
- What FORGE owns: schema, ingestion, dedup, ecnl_verified flag logic
- What ELO owns: rating continuity, team_merges, calibration validation
- Decision authority: ELO selects option; FORGE implements; SENTINEL authorizes before any production schema change

---

### Section 2: Option A — Soft Merge (History Preserved)

**Schema changes required:**
- [FORGE documents: any new columns, indexes, foreign keys]
- Expected DDL complexity: LOW / MEDIUM / HIGH

**Ingestion path changes:**
- [FORGE documents: changes to how ECNL games are ingested and mapped to team IDs]

**Dedup rule changes:**
- [FORGE documents: any changes to `team_merges` handling or dedup logic]

**ecnl_verified flag behavior:**
- [FORGE documents: does the flag apply to old ECNL records, new records, or both?]

**Implementation steps:**
1.
2.
3.

**Time estimate:** [days from authorization to first validated run]

**Risk:** [specific failure modes unique to Option A]

---

### Section 3: Option B — Hard Entity Replacement

**Schema changes required:**

**Ingestion path changes:**

**Dedup rule changes:**

**ecnl_verified flag behavior:**

**Implementation steps:**

**Time estimate:**

**Risk:** [specific concern: historical rating continuity depends on ELO's continuity spec being executed first — FORGE cannot proceed until ELO clears that gate]

---

### Section 4: Option C — Parallel Key with Lookup Table

**Schema changes required:**
- New lookup table required: `ecnl_team_key_map`
- [FORGE documents: exact schema]

**Ingestion path changes:**

**Dedup rule changes:**

**ecnl_verified flag behavior:**

**Implementation steps:**

**Time estimate:** [likely highest — new table design, migration, testing]

**Risk:** [specific failure modes unique to Option C]

---

### Section 5: Cross-Option Shared Requirements

Regardless of option chosen, FORGE must:
- Coordinate timing with ELO's rating continuity execution
- Run a pre-migration snapshot of affected team records
- Confirm `ecnl_verified` backfill is complete before migration executes
- Provide SENTINEL a post-migration verification query result before DSS

---

### Section 6: SENTINEL Authorization Gate

Before any production schema change executes (any option):
- [ ] ELO decision filed and acknowledged
- [ ] FORGE pipeline impact spec for chosen option reviewed by SENTINEL
- [ ] ELO rating continuity pre-migration snapshot confirmed
- [ ] Presten authorization given

FORGE does not touch the production schema until all 4 boxes are checked.

---

### Section 7: Post-Migration Validation

Regardless of option chosen, FORGE files a post-migration validation report covering:
- Row counts: affected teams before and after
- ecnl_verified flag counts: newly set, unchanged
- Ingestion test: one ECNL game ingested correctly under new path
- Any anomalies

---

## Definition of Done

- Spec filed at deliverable path by May 5
- All 3 options have complete pipeline impact sections with specific schema changes, steps, and time estimates
- Option C includes the `ecnl_team_key_map` table DDL
- Section 6 authorization gate is present and accurate
- When ELO files their decision: FORGE updates this doc within 24 hours to mark the chosen option ACTIVE and update status to in-progress

## Why This Matters

ECNL migration is June 2. With a May 9 DSS readiness check and May 16 demo, FORGE has a narrow window to implement migration safely. Waiting until ELO decides to start thinking about the pipeline impact loses 3–5 days of prep. Pre-staging all three options means the day ELO decides, FORGE is ready to start — not ready to research.

## References

- `task-2026-04-26-ecnl-migration-option-comparison-matrix.md` — ELO's option analysis
- `task-2026-04-26-ecnl-migration-evidence-collection.md` — decision evidence
- `task-2026-04-26-ecnl-migration-option-selection-decision-gate.md` — ELO decision gate
- `task-2026-04-26-ecnl-verified-ingestion-path-audit.md` — current ingestion path (FORGE)
- `task-2026-04-24-june2-ecnl-migration-verification-spec.md` — ELO verification side
- `task-2026-04-24-ecnl-rating-continuity-spec.md` — ELO continuity requirement FORGE must coordinate with
