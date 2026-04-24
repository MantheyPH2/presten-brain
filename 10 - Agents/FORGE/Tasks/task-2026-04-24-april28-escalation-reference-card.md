---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: urgent
due: 2026-04-25
deliverable: "02 - Tiger Tournaments/Projects/Product & Planning/April 28 Escalation Reference Card.md"
---

# Task: April 28 Escalation Reference Card

## Context

SENTINEL has declared April 28 as the hard escalation date for two P0/urgent items:
1. **GA ASPIRE tier fix** — ~40pt overcalibration affecting 350–1,050 teams. SQL staged. ~45–60 min to execute.
2. **ECNL dedup P0 fix** — One-line code change to `ecnl-transition-dedup.js`. ~15 min to deploy.

If these are not done by April 28, SENTINEL posts queue alerts and flags them as DSS accuracy risks.

Today is April 24. There are 4 days. The Presten Session Plans document (completed April 25) covers what to do across 1-hour and 3-hour blocks but does not specifically focus on the April 28 deadline or what SENTINEL will do if items remain incomplete.

This task: write a short, Presten-facing reference card to use specifically on April 28 — a single document that functions as a same-day checklist. If Presten opens this document on April 28, they should be able to complete both items in under 90 minutes with no context-switching.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Product & Planning/April 28 Escalation Reference Card.md`

---

### Section 1: Deadline Summary

```
Date: April 28, 2026
Time required: ~75–90 minutes total
Items: 2 (both must be done today to avoid SENTINEL escalation)
```

One-line consequence for each item if not done by April 28:
- ECNL P0 fix: June 1 ECNL migration risk — teams may lose game history. SENTINEL posts queue alert.
- GA ASPIRE fix: DSS demo shows rankings with ~40pt overcalibration on all GA ASPIRE-level teams. SENTINEL posts queue alert.

---

### Section 2: Item 1 — ECNL Dedup P0 Fix (~15 min)

Pull the complete, copy-paste instruction from the existing spec (`Reports/ecnl-dedup-verified-flag-2026-04-24.md`):
- What file to open
- What line to change (the exact one-line fix)
- How to verify the change is correct (the confirmation SQL query from the pre-flight checklist)

Format as a 3-step numbered list. No prose. Just: open file → make change → verify.

Verification query (include it here verbatim):
```sql
SELECT COUNT(*) FROM team_merges 
WHERE merge_type = 'ecnl_transition' 
  AND (verified IS NULL OR verified = false);
-- Expected: 0
```

---

### Section 3: Item 2 — GA ASPIRE Tier Fix (~45–60 min)

Pull the complete execution instructions from the GA ASPIRE fix spec. The relevant file is either:
- `Rankings/calibration-fix-pre-deploy-2026-04-23.md` — check this first
- `Reports/ga-aspire-tier-fix-2026-04-[date].md` — if this file exists

From whichever file contains the staged SQL:
- List the exact SQL queries to run, in order
- Note the expected row count changes (how many teams are affected)
- Note the verification step (the post-fix query to confirm the calibration value changed)

Format as a numbered list. No context. No background. Just what to run.

**Note for FORGE:** If the GA ASPIRE fix file at `Reports/ga-aspire-tier-fix-2026-04-[date].md` does not exist (this has been flagged as a possible gap), check `Rankings/calibration-fix-pre-deploy-2026-04-23.md` for the SQL. If neither file contains the specific GA ASPIRE SQL, write a flag at the top of this section: "⚠️ GA ASPIRE SQL NOT FOUND — SENTINEL must locate or reconstruct. Check ELO's calibration spec." and leave Section 3 body empty. Do not invent SQL.

---

### Section 4: Done State

```
After completing both items:
□ ECNL fix: query returns 0 (verified)
□ GA ASPIRE fix: calibration change confirmed in DB
□ Note completion time and add to Presten Session Plans log
```

One sentence confirming what SENTINEL will mark as resolved when these are done.

---

## Definition of Done

- Reference card is written at the specified path
- Section 2 contains copy-paste ECNL fix instructions sourced from the existing spec
- Section 3 contains copy-paste GA ASPIRE SQL instructions (or an explicit flag if the source SQL is not found)
- Total content is short enough to read in under 2 minutes
- Summary in briefing: "April 28 reference card written. GA ASPIRE SQL: [found at path / NOT FOUND — flag]."

## Why This Matters

SENTINEL has repeated the April 28 escalation in every briefing since April 23. The problem is not that Presten doesn't know — it's that the fix instructions are spread across 3 different documents. On April 28, under time pressure, Presten should open one document and be done in 90 minutes. This card is that document.

## References

- `02 - Tiger Tournaments/Projects/Reports/ecnl-dedup-verified-flag-2026-04-24.md` — ECNL P0 fix spec
- `02 - Tiger Tournaments/Projects/Rankings/calibration-fix-pre-deploy-2026-04-23.md` — possible GA ASPIRE SQL location
- `02 - Tiger Tournaments/Projects/Product & Planning/Presten Session Plans — April 2026.md` — broader execution context
- SENTINEL Briefing 2026-04-24 08:52 — April 28 escalation flags
