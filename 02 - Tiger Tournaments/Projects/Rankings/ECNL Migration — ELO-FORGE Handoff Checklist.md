---
title: ECNL Migration — ELO-FORGE Handoff Checklist
tags: [ecnl, migration, elo, forge, handoff, june2, sentinel-gate]
created: 2026-04-28
author: ELO
status: active — Section 1 fills after April 30 option decision
---

# ECNL Migration — ELO-FORGE Handoff Checklist

> The ECNL migration is June 2. This checklist defines exactly what ELO must deliver to FORGE before FORGE can touch the schema, and what FORGE must deliver to ELO before ELO validates. SENTINEL gatekeeps at three points.

---

## Section 1: Migration Option Baseline

| Field | Value |
|-------|-------|
| Selected option | _[fill after April 30 decision: Option A / B / C]_ |
| Decision filed date | _[fill after April 29 session + April 30 Presten authorization]_ |
| Decision document | `Infrastructure/ECNL Migration — Option Selection Filing Protocol.md` |
| FORGE pipeline spec section | `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md` (Section _[A/B/C]_ marked ACTIVE) |
| ELO recommendation (pre-decision) | Option A — filed 2026-04-28 (`Rankings/ECNL Migration — ELO Option Recommendation.md`) |

---

## Section 2: ELO Pre-Handoff Deliverables (ELO → FORGE)

FORGE does not proceed to any schema change until all items are checked.

| # | Deliverable | Due | Filed At | Status |
|---|-------------|-----|----------|--------|
| 1 | ECNL team identity mapping table (old entity → new entity, with rating continuity plan) | May 10 | `Rankings/ECNL Migration — Team Identity Map.md` | ☐ TODO |
| 2 | Pre-migration rating snapshot for all ECNL-tagged teams | May 15 | `Rankings/ECNL Migration — Pre-Migration Rating Snapshot.md` | ☐ TODO |
| 3 | ecnl_verified backfill confirmed complete (CP1 PASS) | April 29 | `Rankings/ECNL Migration — CP1 CP2 Checkpoint Results.md` | ☐ PENDING April 29 |
| 4 | Rating continuity validation spec for chosen option | May 5 | `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md` (existing) | ☐ TODO — confirm against chosen option |
| 5 | Brier score baseline for ECNL cohort (girls + boys) | May 7 | Within `Rankings/DSS Demo — Ranking Accuracy Evidence Package.md` or standalone | ☐ TODO |

ELO files each item to the specified path. FORGE reads each and confirms receipt in the FORGE Pipeline Impact Spec (Section 6 authorization gate).

---

## Section 3: FORGE Pre-Schema-Change Deliverables (FORGE → ELO)

After ELO completes Section 2, FORGE completes these before touching production schema. ELO confirms each before approving FORGE to proceed.

| # | Deliverable | Due | Filed At | Status |
|---|-------------|-----|----------|--------|
| 1 | FORGE Pipeline Impact Spec with chosen option marked ACTIVE | Within 24h of ELO option decision | `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md` | ☐ TODO |
| 2 | Schema change DDL reviewed by SENTINEL | May 12 | `Infrastructure/ECNL Migration — Schema Change DDL.md` | ☐ TODO |
| 3 | Staging environment test: one ECNL game ingested under new schema | May 14 | FORGE briefing + test result note | ☐ TODO |
| 4 | Rollback plan documented and confirmed executable | May 14 | `Infrastructure/ECNL Migration — Rollback Plan.md` | ☐ TODO |

---

## Section 4: Migration Execution Sequence

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

**Any step failure:** The agent who owns that step files a SENTINEL queue item immediately. SENTINEL decides proceed/abort within 4 hours.

---

## Section 5: Abort Criteria

Migration is aborted (rolled back to pre-migration state) if:

1. ELO continuity verification report (Step 7) shows > 15% of ECNL teams with rating delta > 100 pts with no explainable cause
2. FORGE ingestion path produces duplicate game rows after re-enable (Step 8)
3. ELO 24h stability check (Step 9) shows ongoing anomalous rating movements

**Rollback:** FORGE executes rollback plan (Section 3 item 4). ELO restores pre-migration snapshot (Step 1). SENTINEL confirms rollback complete.

---

## Section 6: SENTINEL Oversight Gates

SENTINEL authorization required before:

- ☐ FORGE applies schema change (Step 3) — SENTINEL confirms Section 2 and Section 3 complete
- ☐ FORGE re-enables ingestion (Step 8) — SENTINEL confirms ELO continuity report PASS
- ☐ Migration declared complete (Step 10) — SENTINEL confirms ELO stability check PASS

SENTINEL will not authorize any gate if the previous gate has an unresolved FAIL.

---

*ELO — 2026-04-28*

## References

- `Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md`
- `Rankings/ECNL Migration Rating Continuity Spec — June 2026.md`
- `Infrastructure/ECNL Migration — FORGE Pipeline Impact Spec.md`
- `Infrastructure/ECNL Migration — Option Selection Filing Protocol.md`
- `Rankings/ECNL Migration — CP1 CP2 Checkpoint Results.md`
- `Rankings/ECNL Migration — Option Comparison Matrix.md`
