---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: completed
completed: 2026-04-28
priority: high
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — ELO-FORGE Handoff Checklist.md"
---

# Task: ECNL Migration — ELO-FORGE Handoff Checklist

## Context

The ECNL migration is June 2. ELO decides the migration option (by April 30). FORGE implements the pipeline side. The June 1 / June 2 sequencing risk register and ELO's rating continuity spec both exist. What does not exist: a shared, ordered checklist defining exactly what ELO must deliver to FORGE before FORGE can touch the schema, and what FORGE must deliver back to ELO before ELO can validate.

Without this handoff checklist, the migration will be executed across two agents with no formal artifact exchange protocol. Either agent could begin their phase before the other is ready, or the handoff could happen with missing artifacts that only become apparent mid-migration.

The June 2 date is hard. The DSS demo is May 9. There is a narrow safe window for the migration between May 17 and June 1. This checklist ensures both agents are working to the same sequence and that SENTINEL can track the handoff at each gate.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — ELO-FORGE Handoff Checklist.md`

---

### Section 1: Migration Option Baseline

| Field | Value |
|-------|-------|
| Selected option | _[fill after April 30 decision: Option A / B / C]_ |
| Decision filed date | _[fill]_ |
| Decision document | `Infrastructure/ECNL Migration — Option Selection Filing Protocol.md` |
| FORGE pipeline impact spec (chosen option) | `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md` (Section _[A/B/C]_ marked ACTIVE) |

---

### Section 2: ELO Pre-Handoff Deliverables (ELO → FORGE)

ELO must complete and file these artifacts before FORGE makes any schema change. FORGE does not proceed until all items are checked.

| # | Deliverable | Due | Filed At | Status |
|---|-------------|-----|----------|--------|
| 1 | ECNL team identity mapping table (old entity → new entity, with rating continuity plan) | May 10 | `Rankings/ECNL Migration — Team Identity Map.md` | _[ ] TODO_ |
| 2 | Pre-migration rating snapshot for all ECNL-tagged teams | May 15 | `Rankings/ECNL Migration — Pre-Migration Rating Snapshot.md` | _[ ] TODO_ |
| 3 | ecnl_verified backfill confirmed complete (file count > 0) | April 29 (CP1) | Confirmed in `Rankings/ECNL Migration — CP1 CP2 Checkpoint Results.md` | _[ ] TODO_ |
| 4 | Rating continuity validation spec for chosen option | May 5 | `Rankings/ECNL Migration — Rating Continuity Validation Spec.md` (existing task) | _[ ] TODO_ |
| 5 | Brier score baseline for ECNL cohort (girls + boys) | May 7 | Within DSS accuracy evidence package OR standalone | _[ ] TODO_ |

ELO files each item to the specified path. FORGE reads each and confirms receipt in the FORGE Pipeline Impact Spec (Section 6 authorization gate).

---

### Section 3: FORGE Pre-Schema-Change Deliverables (FORGE → ELO)

After ELO completes Section 2, FORGE completes these before touching production schema. ELO confirms each before approving FORGE to proceed.

| # | Deliverable | Due | Filed At | Status |
|---|-------------|-----|----------|--------|
| 1 | FORGE Pipeline Impact Spec with chosen option marked ACTIVE | Within 24h of ELO option decision | `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md` | _[ ] TODO_ |
| 2 | Schema change DDL reviewed by SENTINEL | May 12 | `Infrastructure/ECNL Migration — Schema Change DDL.md` | _[ ] TODO_ |
| 3 | Staging environment test: one ECNL game ingested under new schema | May 14 | FORGE briefing + test result note | _[ ] TODO_ |
| 4 | Rollback plan documented and confirmed executable | May 14 | `Infrastructure/ECNL Migration — Rollback Plan.md` | _[ ] TODO_ |

---

### Section 4: Migration Execution Sequence

The exact order of operations on June 1–2:

| Step | Date | Owner | Action | Prerequisite |
|------|------|-------|--------|--------------|
| 1 | May 31 | ELO | Final pre-migration rating snapshot | ecnl_verified confirmed complete |
| 2 | June 1 | FORGE | Disable ECNL ingestion in pipeline | SENTINEL authorization |
| 3 | June 1 | FORGE | Apply schema change (DDL) | Section 3 all checked |
| 4 | June 1 | FORGE | Update ingestion path to new mapping | Schema change confirmed applied |
| 5 | June 1 | ELO | Apply team identity mapping table to ratings | FORGE ingestion path confirmed |
| 6 | June 1 | ELO | Run ratings recompute for ECNL cohort | Identity map applied |
| 7 | June 1 | ELO | File ratings continuity verification report | Recompute complete |
| 8 | June 2 | FORGE | Re-enable ECNL ingestion | ELO continuity report = PASS |
| 9 | June 2 | ELO | 24h post-migration ratings stability check | First post-migration games ingested |
| 10 | June 2 | SENTINEL | Authorization to declare migration complete | ELO stability check = PASS |

Any step failure: the agent who owns that step files a SENTINEL queue item immediately. SENTINEL decides proceed/abort within 4 hours.

---

### Section 5: Abort Criteria

Migration is aborted (rolled back to pre-migration state) if:

1. ELO continuity verification report (Step 7) shows > 15% of ECNL teams with rating delta > 100 pts with no explainable cause
2. FORGE ingestion path produces duplicate game rows after re-enable (Step 8)
3. ELO 24h stability check (Step 9) shows ongoing anomalous rating movements

Rollback: FORGE executes rollback plan (Section 3 item 4). ELO restores pre-migration snapshot (Step 1). SENTINEL confirms rollback complete.

---

### Section 6: SENTINEL Oversight Gates

SENTINEL authorization required before:
- [ ] FORGE applies schema change (Step 3) — SENTINEL confirms Section 2 and Section 3 complete
- [ ] FORGE re-enables ingestion (Step 8) — SENTINEL confirms ELO continuity report PASS
- [ ] Migration declared complete (Step 10) — SENTINEL confirms ELO stability check PASS

SENTINEL will not authorize any gate if the previous gate has an unresolved FAIL.

---

## Definition of Done

- Document filed at deliverable path by May 5
- Section 1 filled after April 30 ELO option decision
- Section 2 items have specific due dates (not relative)
- Section 3 items have specific due dates
- Section 4 execution sequence has calendar dates (June 1–2) confirmed
- ELO briefing May 5 states: "ELO-FORGE handoff checklist filed; Section 2 deliverable 3 (ecnl_verified CP1) already confirmed"

## Why This Matters

The ECNL migration is the highest-risk schema change in the Evo Draw timeline. It touches team identity for the most-watched league in the platform. ELO and FORGE have never executed a joint schema migration. Without a checklist, the migration happens by verbal agreement across briefing documents — which is not auditable, not reliable, and not SENTINEL-authorized. This checklist makes the migration a structured handoff that SENTINEL can review and gatekeep at each step.

## References

- `task-2026-04-27-june1-june2-sequencing-risk-register.md` — risk register (complement this)
- `task-2026-04-24-ecnl-rating-continuity-spec.md` — ELO rating continuity requirement
- `task-2026-04-28-ecnl-migration-pipeline-impact-spec.md` — FORGE pipeline impact spec (completed today)
- `task-2026-04-28-ecnl-migration-option-filing-protocol.md` — ELO option decision filing (assigned 10:36)
- `task-2026-04-28-ecnl-cp1-cp2-checkpoint-results-document.md` — CP1/CP2 checkpoint tracking
- `task-2026-04-26-ecnl-migration-option-comparison-matrix.md` — option analysis
