---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: urgent
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/FM1-FM2 Code Search — Presten Quick-Run Guide.md"
---

# Task: FM1/FM2 Code Search — Presten Quick-Run Guide

## Context

Both the GA ASPIRE Pipeline Classifier Audit and the ecnl_verified Ingestion Path Audit returned Outcome C — the pipeline codebase is not in the vault. The `Pipeline Code Review Results.md` form is ready and waiting for Presten to fill in FM1-1 through FM2-3. The bottleneck is not the form — it's that Presten does not have a terminal-ready guide for locating the relevant code in under 5 minutes.

The two audit documents contain search terms and code patterns, but they are written as audit documents — multi-section, full context. Presten needs a one-page run guide that says: open terminal, run these commands, paste the output into Pipeline Code Review Results.md.

This task produces that guide. April 28 cannot be classified GO/HOLD until FM1/FM2 are resolved. This guide is the unlock.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/FM1-FM2 Code Search — Presten Quick-Run Guide.md`

---

### Header

```
Target time: ≤ 5 minutes in terminal
Open alongside: Infrastructure/Pipeline Code Review Results.md
Complete before: April 28 DB session start
```

---

### FM1: GA ASPIRE Classifier (fills FM1-1, FM1-2, FM1-3)

**Step 1 — Find `ga_aspire` in the codebase:**
```bash
grep -r "ga_aspire" . --include="*.js" --include="*.ts" --include="*.py" -l
```
Expected: one or more files returned. If zero results → FM1 Path B or C; note in FM1-1.

**Step 2 — Find event tier classification logic:**
```bash
grep -r "event_tier" . --include="*.js" --include="*.ts" --include="*.py" -n | grep -v "test" | head -30
```
Look for where `event_tier` is assigned or mapped. Note the file:line in FM1-2.

**Step 3 — Check for ON CONFLICT revert risk:**
```bash
grep -r "ON CONFLICT" . --include="*.js" --include="*.ts" --include="*.py" -n | grep -i "event_tier"
```
If this returns results that include `event_tier`: FM1-3 is a FAIL. If empty: FM1-3 is a PASS.

**FM1 mapping:**
- `ga_aspire` found in classifier map with correct value → Path A
- `ga_aspire` not found, but classifier uses a simple lookup dict/table → Path B (≤ 5-line fix)
- Classifier logic is complex/structural → Path C

---

### FM2: ecnl_verified Scraper Write (fills FM2-1, FM2-2, FM2-3)

**Step 1 — Find `ecnl_verified` in TGS scraper:**
```bash
grep -r "ecnl_verified" . --include="*.js" --include="*.ts" --include="*.py" -n
```
If found: note file:line in FM2-1. If not found: FM2-1 is a FAIL (column not written).

**Step 2 — Check if the Population Plan code change has been applied:**
```bash
grep -r "ecnl_verified.*true\|ecnl_verified: true" . --include="*.js" --include="*.ts" --include="*.py" -n
```
If found: FM2-2 PASS. If not found: FM2-2 FAIL (change not yet applied).

**Step 3 — If not applied, confirm it is on May 1 plan:**
Check `Infrastructure/May 1 Deployment — Day-Of Runbook.md`. Is `ecnl_verified` scraper fix listed? YES → FM2-3 PASS. NO → FM2-3 FAIL (add to runbook).

**FM2 mapping:**
- Scraper writes `ecnl_verified = TRUE` for ECNL games → Path A
- Scraper does not write `ecnl_verified`; adding it is a ≤ 5-line change → Path B
- Scraper architecture prevents simple field addition → Path C

---

### Results Paste Template

Copy-paste the following into `Pipeline Code Review Results.md` notes sections:

```
FM1-1 — `ga_aspire` found at: [paste grep output line]
FM1-2 — Classification logic at: [paste file:line]
FM1-3 — ON CONFLICT includes event_tier? YES / NO

FM2-1 — `ecnl_verified` found at: [paste grep output line, or "not found"]
FM2-2 — Code change applied? YES / NO
FM2-3 — On May 1 plan? YES / NO / N/A

FM1 Path: A / B / C
FM2 Path: A / B / C
```

---

### After Completing

1. Fill in the SENTINEL Disposition section of `Pipeline Code Review Results.md`
2. Send result to SENTINEL in next briefing — SENTINEL updates FM1/FM2 path letters and routes to April 28 Decision Tree

---

## Quality Standard

- Every command must be copy-paste ready (no placeholders in the command itself)
- All three FM1 steps and all three FM2 steps must map directly to a field in `Pipeline Code Review Results.md`
- Total run time estimate must be stated at the top (target ≤ 5 minutes)
- The results paste template must match the exact field names in Pipeline Code Review Results.md

## Why This Matters

FM1/FM2 paths determine whether April 28 is GO or HOLD. Both audits returned Outcome C two hours ago. The form is built. The guide does not exist. FORGE writes this tonight so Presten can run the code review first thing tomorrow — April 27 — and return FM1/FM2 paths to SENTINEL before April 28.

## References

- `Infrastructure/Pipeline Code Review Results.md` — the form this guide unlocks
- `Infrastructure/GA ASPIRE Pipeline Classifier Audit — April 2026.md` — FM1 source patterns
- `Infrastructure/ecnl_verified Ingestion Path Audit — April 2026.md` — FM2 source patterns
- `Infrastructure/April 29 Pipeline Code Fail — Decision Tree.md` — routing table after FM1/FM2 paths known
