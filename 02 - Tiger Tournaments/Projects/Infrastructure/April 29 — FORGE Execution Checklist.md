---
title: April 29 — FORGE Execution Checklist
tags: [infrastructure, pipeline, execution-checklist, forge, april-29, evo-draw]
created: 2026-04-27
updated: 2026-04-27
author: FORGE
status: ready — use on April 29
---

# April 29 — FORGE Execution Checklist

> **Linear execution document.** Open on April 29 and work top to bottom. Every step ends with a ✅ or a documented escalation — no step concludes with "investigate."

---

## Pre-Flight Gate (Before Any April 29 Action)

Confirm April 28 closed cleanly before running anything.

```
[ ] April 28 Execution Log filed at: Infrastructure/April 28 Execution Log.md
[ ] Presten confirmed all 3 steps complete (Post-Session Summary → "All 3 steps complete: YES")
[ ] SENTINEL Disposition section not marked "HELD"
```

**If any of the above are not confirmed: STOP. Flag to SENTINEL. Do not proceed.**

---

## Step 1 — Schema Verification (< 5 minutes)

```
[ ] Connect to DB (psql)
[ ] Run: \d games
[ ] Confirm: ecnl_verified column is present, type BOOLEAN, default FALSE
```

```sql
SELECT column_name, data_type, column_default
FROM information_schema.columns
WHERE table_name = 'games' AND column_name = 'ecnl_verified';
```

**PASS:** One row returned, `data_type = boolean`, `column_default = false`.  
**FAIL:** Zero rows returned → `ecnl_verified` column not added. Flag to SENTINEL immediately. **Do not proceed to Step 2.**

---

## Step 2 — Section 4 Query Package Execution (< 15 minutes)

Reference: `Infrastructure/April 29 — Section 4 Source Baseline Query Package.md`

```
[ ] Pre-run gate confirmed: April 28 execution confirmed (Pre-Flight above)
[ ] Run S4-1: Total game count by source
[ ] Run S4-2: Active org-ID count
[ ] Run S4-3: Game recency by source
[ ] Run S4-4: ecnl_verified population rate
[ ] Run S4-5: GA ASPIRE event_tier post-fix count
[ ] Save all outputs before closing psql
```

**S4-4 escalation trigger:** If `ecnl_verified` population rate = 0% for ECNL source games, flag to SENTINEL before filing briefing. Run the backfill SQL from `Infrastructure/ECNL ecnl_verified — Backfill SQL Package.md` if the column exists but is unpopulated.

**S4-5 escalation trigger:** If `ga_aspire` game count = 0 after the April 28 fix, flag to SENTINEL immediately. April 28 Step 1 may not have applied correctly.

---

## Step 3 — Source Baseline Section 4 Filing (< 10 minutes)

```
[ ] Open: Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md
[ ] Navigate to Section 4 (placeholder block)
[ ] Fill in S4-1 through S4-5 query outputs
[ ] Record run date/time at top of Section 4: "Filled: 2026-04-29 [HH:MM]"
[ ] Save file
```

Note in Step 5 briefing checklist: "Section 4 filed — [date/time]. GA ASPIRE confirmed [yes/no]. ecnl_verified population: [%]."

---

## Step 4 — FM1/FM2 Audit Outcome Update (If Results Available)

```
[ ] Open: Infrastructure/Pipeline Code Review Results.md
[ ] Check: FM1 Path Letter and FM2 Path Letter filed by Presten?
```

**If FM1/FM2 path letters ARE filed:**
```
[ ] Open Infrastructure/GA ASPIRE Pipeline Classifier Audit — April 2026.md
[ ] Update status, add confirmed path letter, fill confirmed outcome section
[ ] Open Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md
[ ] Update status, add confirmed path letter, fill confirmed outcome section
[ ] Reference: task-2026-04-26-audit-documents-outcome-update-protocol.md for exact field edits
[ ] File outcome in briefing: "FM1 Path [A/B/C] confirmed. FM2 Path [A/B/C] confirmed. Audit docs updated."
```

**If FM1/FM2 results NOT filed:**
File Queue item at `10 - Agents/FORGE/Queue/pending-2026-04-29-fm1-fm2-unresolved.md`:
```
FM1/FM2 code review results not in Pipeline Code Review Results.md as of April 29.
Both audit documents remain Outcome C (open). Audit closure blocked.
SENTINEL must escalate to Presten if FM1/FM2 results are needed before May 1 deployment.
```

---

## Step 5 — Org-ID Browser Session Results (If Available)

```
[ ] Check with Presten: has the browser session returned org-IDs?
```

**If org-IDs received:**
```
[ ] Open: Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md
[ ] Replace ORG_ID_PENDING with confirmed numeric values
[ ] Open: Infrastructure/GotSport Org-ID Master Reference — April 2026.md
[ ] Move confirmed entities from Section 2 (pending) to Section 1 (active)
[ ] Update Section 4 Status Summary block (counts + last-updated timestamp)
[ ] File in briefing: "[N] entities confirmed. Org-IDs: [list]."
[ ] Target: < 5 minutes per entity
```

**If no results received:** Note in briefing. No action.

---

## Briefing Checklist

Before filing the April 29 FORGE briefing, confirm all of the following are addressed:

```
[ ] Section 4 filed in Source Baseline (or blocker documented with escalation)
[ ] ecnl_verified column status confirmed (PASS or FLAG)
[ ] ecnl_verified population rate filed (or backfill executed and noted)
[ ] FM1/FM2 update filed OR pending status Queue item filed
[ ] Org-ID updates applied if browser session ran
[ ] Any April 28 anomalies flagged (unexpected row counts, errors in Execution Log)
[ ] tasks_completed count accurate in briefing frontmatter
```

---

## Step Dependency Summary

| Step | Prerequisite | Output |
|------|-------------|--------|
| Pre-Flight Gate | April 28 session ran | Gate — HELD = stop |
| 1: Schema Verification | Pre-Flight ✅ | ecnl_verified confirmed present |
| 2: S4 Query Package | Step 1 ✅ (column must exist for S4-4) | 5 query outputs saved |
| 3: Source Baseline Filing | Step 2 ✅ | Section 4 filled |
| 4: FM1/FM2 Update | Presten's code review results | Audit docs updated OR Queue item filed |
| 5: Browser Session | Presten browser session ran | Config + Master Reference updated |

**Total time (Steps 1–5 all active):** ~45–55 minutes.

---

## References

- `Infrastructure/April 28 Execution Log.md` — Pre-flight gate input
- `Infrastructure/April 29 — Section 4 Source Baseline Query Package.md` — Step 2 queries
- `Infrastructure/Source Ingestion Baseline — Pre-Launch Snapshot.md` — Step 3 target
- `Infrastructure/ECNL ecnl_verified — Backfill SQL Package.md` — Step 2 escalation path
- `task-2026-04-26-audit-documents-outcome-update-protocol.md` — Step 4 procedure
- `Infrastructure/Stage 2 Org-ID Config — Pre-Staged Additions.md` — Step 5 document

*FORGE — 2026-04-27*
