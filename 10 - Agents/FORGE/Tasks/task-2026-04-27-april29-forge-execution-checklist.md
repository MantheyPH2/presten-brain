---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/April 29 — FORGE Execution Checklist.md"
---

# Task: April 29 FORGE Execution Checklist

## Context

April 29 is the most operationally dense session in the current critical path. After April 28 closes, FORGE has multiple triggered actions: confirm the April 28 Execution Log is complete, verify the `ecnl_verified` column was added correctly, run the Section 4 query package, file Section 4 in the Source Baseline, and execute the FM1/FM2 audit outcome update if code search results are in. These actions are documented in separate files; what does not exist is a single ordered checklist FORGE works through linearly on April 29 without having to synthesize from multiple documents.

Without this checklist, April 29 for FORGE is a reconstruction exercise: open the briefing, remember what was assigned, navigate to 4–5 separate files. With it, FORGE opens one document and completes items in order. At this density of parallel tracks, that difference is a real execution risk reducer.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/April 29 — FORGE Execution Checklist.md`

This is a linear, ordered checklist — not a reference document. FORGE opens it on April 29, works from top to bottom, marks each item complete.

---

### Pre-Flight Gate (Before Any April 29 Action)

One block confirming April 28 is done:

```
[ ] April 28 Execution Log filed at: Infrastructure/April 28 Execution Log.md
[ ] Presten confirmed all 3 steps complete (Section: Post-Session Summary — "All 3 steps complete: YES")
[ ] SENTINEL disposition section not marked "HELD"
If any of the above are not confirmed: stop, flag to SENTINEL, do not proceed.
```

---

### Step 1 — Schema Verification (< 5 minutes)

```
[ ] Connect to DB (psql or equivalent)
[ ] Run: \d games
[ ] Confirm: ecnl_verified column is present with type BOOLEAN and default FALSE
[ ] If column absent: flag to SENTINEL immediately — April 28 Step 2 did not complete
```

---

### Step 2 — Section 4 Query Package Execution (< 15 minutes)

```
[ ] Open: Infrastructure/April 29 — Section 4 Source Baseline Query Package.md
[ ] Confirm pre-run gate: April 28 execution confirmed (see Pre-Flight Gate above)
[ ] Run S4-1: Total game count by source
[ ] Run S4-2: Active org-ID count
[ ] Run S4-3: Game recency by source
[ ] Run S4-4: ecnl_verified population rate
[ ] Run S4-5: GA ASPIRE event_tier post-fix count
[ ] Save all outputs
```

---

### Step 3 — Source Baseline Section 4 Filing (< 10 minutes)

```
[ ] Open: Infrastructure/Source Ingestion Baseline — April 2026.md (or equivalent)
[ ] Navigate to Section 4
[ ] Paste S4-1 through S4-5 query outputs into Section 4
[ ] Record date/time of queries
[ ] Save file
[ ] Note in next briefing: "Section 4 filed — [date/time]. Outputs: [S4-4 population rate] for ecnl_verified."
```

---

### Step 4 — FM1/FM2 Audit Outcome Update (If Results Available)

```
[ ] Check: Infrastructure/Pipeline Code Review Results.md
[ ] If FM1/FM2 path letters have been filed by Presten:
    [ ] Open: Infrastructure/[FM1 audit document] and Infrastructure/[FM2 audit document]
    [ ] Update outcome sections per task-2026-04-26-audit-documents-outcome-update-protocol.md
    [ ] File outcome update in briefing
[ ] If FM1/FM2 results are NOT yet filed:
    [ ] Note in briefing: "FM1/FM2 audit outcome update pending — results not yet in Pipeline Code Review Results.md"
    [ ] Do not proceed with any April 28 DB session action that depends on FM1/FM2 path
```

---

### Step 5 — Org-ID Browser Session Results (If Available)

```
[ ] Check with Presten: has the browser session returned any org-IDs?
[ ] If org-IDs received:
    [ ] Open: Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md
    [ ] Replace ORG_ID_PENDING with confirmed numeric values
    [ ] Update: Infrastructure/GotSport Org-ID Master Reference — April 2026.md (Section 2 rows → status confirmed)
    [ ] Target: < 5 minutes per entity
[ ] If no results: note in briefing, no action.
```

---

### Briefing Checklist

Before filing the April 29 briefing, confirm:

```
[ ] Section 4 filed (or blocker documented)
[ ] ecnl_verified column status confirmed
[ ] FM1/FM2 update filed or pending status noted
[ ] Org-ID updates filed if browser session ran
[ ] Any April 28 anomalies flagged (unexpected row counts, errors in Execution Log)
```

---

## Quality Standard

- Every step ends with a ✅ or a documented escalation — no step concludes with "investigate."
- The Pre-Flight Gate is non-optional. If April 28 is not confirmed complete, FORGE stops and flags before running any query.
- Steps are ordered by dependency: schema check before Section 4 (S4-4 depends on ecnl_verified existing); Section 4 before briefing.

## Why This Matters

April 29 has FORGE, ELO, and Presten all executing in parallel. The Section 4 query output feeds ELO's G1 gate — ELO cannot close G1 without FORGE's Section 4 results. Any FORGE confusion on April 29 creates a bottleneck in ELO's gate sequence. This checklist eliminates FORGE confusion at its source: one document, linear execution, no synthesis required.

## References

- `Infrastructure/April 28 Execution Log.md` — pre-flight gate input
- `Infrastructure/April 29 — Section 4 Source Baseline Query Package.md` — Step 2 queries
- `Infrastructure/Source Ingestion Baseline — April 2026.md` — Step 3 target
- `task-2026-04-26-audit-documents-outcome-update-protocol.md` — Step 4 procedure
- `Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md` — Step 5 document
