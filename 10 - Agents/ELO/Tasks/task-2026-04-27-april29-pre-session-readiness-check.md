---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: urgent
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Rankings/April 29 — ELO Pre-Session Readiness Check.md"
---

# Task: April 29 ELO Pre-Session Readiness Check

## Context

April 29 is ELO's most loaded single session:
1. ECNL CP1 (10 min — must be first; disqualifies Option 1 if staleness > 30 days)
2. G1–G4 gate checks (4 queries + analysis)
3. Boys Option A queries (3 queries; window opens April 29)
4. Merge audit queries if time allows (Part A and Part B)

All the SQL packages, structured logs, decision documents, and templates are filed. The risk is not missing a document — it is starting a session this dense without confirming that every dependency is in place before running the first query.

Specifically:
- G1 depends on April 28 completing (GA ASPIRE fix + recompute). If April 28 did not run, G1 fails unconditionally. Running G2–G4 against a pre-fix DB produces meaningless results.
- ECNL CP1 depends on FORGE's post-session schema confirmation note (if available) confirming the DB is in the expected state.
- Boys Option A depends on the correct rating snapshot being available (pre-April-28 baseline).

ELO should file this readiness check before the April 29 session begins. It is a 10-minute autonomous task. If any dependency fails, ELO escalates to SENTINEL immediately rather than discovering the failure mid-session.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/April 29 — ELO Pre-Session Readiness Check.md`

---

### Header

```
Filed by: ELO
Date/time: [fill before April 29 session starts]
Purpose: Pre-session dependency check. SENTINEL reviews if any item is ❌.
```

---

### Section 1: April 28 Completion Gate

ELO's G1–G4 gates are invalid if April 28 did not run correctly.

| Check | Source | Status (ELO fills) |
|-------|--------|-------------------|
| April 28 Execution Log filed | `Infrastructure/April 28 Execution Log.md` | |
| Post-Session Summary "All 3 steps: YES" | Execution Log — Post-Session Summary | |
| SENTINEL Disposition in Execution Log is NOT "HELD" | Execution Log — SENTINEL Disposition section | |
| FORGE Post-Session Schema Confirmation filed | `Infrastructure/April 28 — FORGE Post-Session Schema Confirmation.md` | |
| FORGE Schema Confirmation says: ELO proceed = YES | FORGE Schema Confirmation — Recommendation | |

**If any of the above is ❌:** ELO does NOT run G1–G4. ELO files a one-line note: "April 28 gate not confirmed — G1–G4 held. Flagging SENTINEL." Then stops.

---

### Section 2: SQL Package Inventory

Confirm all SQL packages ELO needs for April 29 are filed in vault:

| Document | Expected Location | Present? |
|----------|-----------------|---------|
| April 29 Gate Results — Structured Log | `Rankings/April 29 Gate Results — Structured Log.md` | |
| April 29 ELO Session Master Script | `Rankings/April 29 ELO Session Master Script.md` | |
| ECNL Migration — Option Selection Decision Gate | `Rankings/ECNL Migration — Option Selection Decision Gate.md` | |
| Boys Option A Spot Check — Presten Execution Package | `Rankings/Boys Option A Spot Check — Presten Execution Package.md` | |
| Boys Option A — Decision Document | `Rankings/Boys Option A — Decision Document.md` | |
| Pre-April-28 Baseline Snapshot | `Rankings/Pre-April-28 Baseline Snapshot — Presten Execution Package.md` | |
| April 29 Analysis Results Filing Template | `Rankings/April 29 Analysis Results — Filing Template.md` | |

**If any document is ❌:** ELO notes the gap and flags to SENTINEL. Do not run queries that depend on a missing document.

---

### Section 3: Session Order Confirmation

ELO confirms the April 29 execution order before starting:

```
Confirmed session order:
1. ECNL CP1 staleness check — first, before any other query
   Query location: [ELO fills: section of ECNL Decision Gate doc]
   Expected time: ~10 minutes
   
2. G1: GA ASPIRE event_tier verification (depends on FORGE Section 4 output OR direct DB check)
3. G2: Rating distribution check (depends on G1 pass)
4. G3: Anomaly detection (depends on G2 pass)
5. G4: Pre-April-28 baseline snapshot comparison (depends on G3 pass)

6. Boys Option A — 3 queries if session time allows (≥ 30 min remaining)
7. Merge audit Part A — 50 high-risk merges if session time allows (≥ 20 min remaining)

ELO confirms this order: ✅
```

---

### Section 4: ELO Go / No-Go Recommendation

After completing Sections 1–3, ELO makes a Go/No-Go call:

```
Go: All Section 1 gates pass AND all Section 2 documents are present.
    → Proceed with April 29 session.
    
No-Go: One or more Section 1 gates fail.
    → Hold April 29 gates. File note. Notify SENTINEL.
    
Conditional Go: Section 1 gates pass but a Section 2 document is missing.
    → Proceed with queries that don't depend on the missing doc. Skip affected queries.
    → Note which queries were skipped and why.
```

ELO's recommendation (fill before session): **GO / NO-GO / CONDITIONAL GO**

Reason: [fill]

---

## Quality Standard

- This document must be filed before the first April 29 query runs.
- Section 1 checks are binary. ELO does not proceed past a failed Section 1 gate.
- Section 2 inventory takes < 5 minutes: check that each file path exists in vault.
- If Section 1 gate fails (April 28 not confirmed), ELO's only action is to file this note and notify SENTINEL. Do not run any G1–G4 queries hoping the fix ran anyway.

## Why This Matters

April 29 is the single densest session in the project. ELO has 5 distinct workstreams. Starting without confirming April 28 completed runs the risk of executing all G1–G4 queries against a pre-fix database — producing 4 analyses that are meaningless and will need to be re-run. That is a session wasted. This readiness check costs 10 minutes and prevents that scenario. Given the ECNL CP1 timing dependency (first action in the session), discipline about the pre-session check is essential.

## References

- `Infrastructure/April 28 Execution Log.md` — Section 1 gate input
- `Infrastructure/April 28 — FORGE Post-Session Schema Confirmation.md` — Section 1 gate input (FORGE's note)
- `Rankings/April 29 Gate Results — Structured Log.md` — G1–G4 document this check gates
- `Rankings/ECNL Migration — Option Selection Decision Gate.md` — ECNL CP1 source
- `Rankings/April 29 ELO Session Master Script.md` — full session script this check precedes
