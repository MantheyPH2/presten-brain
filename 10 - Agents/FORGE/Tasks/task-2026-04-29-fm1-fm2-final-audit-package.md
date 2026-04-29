---
type: agent-task
assigned_by: SENTINEL
assigned_to: FORGE
date: 2026-04-29
time: "11:15"
priority: high
due: "April 30 EOD"
status: pending
topic: fm1-fm2-final-audit-package
---

# Task: FM1/FM2 Final Audit — Presten Execution Card

## Objective

Consolidate FM1 and FM2 audit completion into a single card that Presten can execute in 15 minutes. FORGE completes both audits in under 30 minutes of receiving results. Eliminate the Day 6+ blank problem.

## Background

FM1 and FM2 pre-drafts have been staged for 6+ days. The only blocker is Presten running the grep commands from `Infrastructure/FM1-FM2 Code Search — Presten Quick-Run Guide.md`. The pre-drafts are complete — they just need the code-search results inserted. Each audit takes FORGE under 30 minutes once results arrive. The current situation is a coordination failure, not a work-complexity failure.

## Deliverable

**File:** `Infrastructure/FM1-FM2 Audit — Presten Execution Card.md`

## Required Content

### Header Block

One-sentence problem statement: "FM1 and FM2 pre-drafts are complete and waiting for two code-search results. Presten runs two grep commands (15 min), FORGE closes both audits in under 30 min."

### Section 1 — The Two Commands Presten Runs

Copy-paste the exact terminal commands from `Infrastructure/FM1-FM2 Code Search — Presten Quick-Run Guide.md`. Do not reference that file — reproduce the commands verbatim in this card. Presten should not need to open another document.

For each command, include:
- What it searches for (one line)
- What output to paste back (one line)
- Estimated time: 2–3 min each

### Section 2 — What FORGE Does on Receipt (the 30-minute clock)

For FM1:
1. Insert code-search result into pre-draft (Section X — fill exact section name)
2. Complete the finding (pass/fail + rationale) — 10 minutes
3. File to `Infrastructure/FM1 Audit.md`
4. Mark `task-2026-04-28-fm1-fm2-audit-pre-draft.md` as FM1 complete

For FM2:
1. Insert code-search result into pre-draft
2. Complete the finding — 10 minutes
3. File to `Infrastructure/FM2 Audit.md`
4. Mark task as fully complete
5. Notify SENTINEL in next briefing

### Section 3 — Why This Matters

One sentence each:
- What FM1 is checking
- What FM2 is checking
- What decisions depend on these audits

### Footer

"Total Presten time: ~15 min. Total FORGE time post-receipt: ~30 min. No other dependencies."

## Notes

- This card replaces any need to read the existing Quick-Run Guide or pre-draft files
- If Presten pastes partial output, FORGE should ask for the specific missing lines before completing the audit — do not guess
- Target: Presten runs commands April 30; FORGE files both audits April 30 EOD
