---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-04-28
priority: high
status: completed
completed: 2026-04-27
topic: april28-signal-monitoring-checklist
---

# Task: April 28 — Signal Monitoring Checklist

## Context

ELO's 14:06 briefing states that April 28 is a Presten execution day and that ELO will "monitor via vault: when the April 28 Execution Log and FORGE Post-Session Schema Confirmation are filed, ELO reviews to confirm G0 prerequisites are met before April 29."

This monitoring plan is described verbally but has no structured protocol. When ELO checks the vault on April 28, it needs to know: which exact files to look for, what each file's existence (or absence) means, and when a missing or RED signal requires SENTINEL escalation before April 29 begins.

If ELO receives conflicting signals — for example, the Execution Log is filed but the compute-rankings.js run is not confirmed — ELO needs unambiguous instructions on whether to flag SENTINEL, wait, or mark G0 as conditional. Without this protocol, ELO will improvise, and improvisation under time pressure creates errors.

## Task

Produce a one-page April 28 Signal Monitoring Checklist. ELO opens this when Presten's April 28 session is expected to be complete and works through it before April 29 begins.

## Document Must Include

### Section 1: Files to Check (in order)

A numbered checklist of vault files ELO looks for. For each file:
- File path (exact)
- What its existence confirms
- What its absence or RED status means
- Estimated check time (5 minutes total for all items)

Minimum files to check:
1. `Infrastructure/April 28 Execution Log.md` (or similar) — confirms Presten's session started and completed
2. `Infrastructure/FORGE Post-Session Schema Confirmation.md` (or similar) — confirms compute-rankings.js ran and schema is intact
3. `Rankings/compute-rankings.js run log or output` (if filed separately) — confirms rankings recomputed post-GA-ASPIRE UPDATE

Note: FORGE's FM1/FM2 Activation Confirmation template path (`Infrastructure/FM1-FM2-Activation-Confirmation-2026-04-28.md`) — ELO does NOT need to check this for G0; it is a FORGE activation document, not a prerequisite for ELO's gate assessment.

### Section 2: G0 Decision Logic

Three possible states ELO can conclude from the checklist:

**G0 = GO:** Both required files are filed and show no RED signals. ELO confirms G0 in the April 29 Pre-Session Readiness Check and proceeds.

**G0 = NO-GO (Session Did Not Complete):** Execution Log not filed by [time threshold — ELO should set a reasonable cutoff, e.g., 10:00 PM April 28]. Action: file a brief G0 = NO-GO note to `Rankings/April 29 Pre-Session Readiness Check.md` and flag SENTINEL via queue item. Do NOT begin April 29 workstreams.

**G0 = CONDITIONAL (Partial Signals):** Execution Log filed but schema confirmation missing or ambiguous. Action: note the specific gap, flag SENTINEL, proceed with April 29 items that do NOT depend on schema confirmation (e.g., Boys Option A if data is available, Girls GA ASPIRE stability monitoring queries). Gate the schema-dependent workstreams until SENTINEL resolution.

### Section 3: Escalation Protocol

If G0 = NO-GO or CONDITIONAL:
- File a queue item to `10 - Agents/SENTINEL/Queue/` with category: alert, priority: urgent
- Queue item title: "G0 Signal Gap — April 28 Execution Not Fully Confirmed"
- Queue item body: which files were found, which were missing, ELO's assessment
- Do NOT begin ECNL CP1 or the Gate Results Structured Log without G0 confirmation

### Section 4: Quick Reference — What ELO Needs Before Each April 29 Workstream

| April 29 Workstream | G0 Required? | Additional Prerequisite |
|--------------------|-------------|-------------------------|
| ECNL CP1 | Yes — schema must be intact | ecnl_verified column confirmed (G2 will test this in-session) |
| G1–G4 Gate Results Log | Yes | Baseline snapshot taken before April 28 recompute |
| Girls GA ASPIRE Stability Monitoring | Yes (fix must have run) | G1 result |
| Boys Option A (3 queries) | No — independent of April 28 fix | Boys query pack validated |
| Post-Fix Distribution Analysis | Yes | rankings_snapshot taken before April 28 recompute |

## Deliverable

File to: `Rankings/April 28 — Signal Monitoring Checklist.md`

Update this task to `status: completed` when filed.

## Notes

This document is ELO's operational tool for one evening (April 28). It should be simple and fast to use — not a comprehensive analysis. The goal is a structured 5-minute vault check, not a thorough investigation.

If ELO cannot find the relevant file paths from vault memory (they may not exist yet, since April 28 hasn't happened), define the checklist using the expected paths from FORGE's template filings (FM1/FM2 template, Schema Confirmation note, etc.). Note them as "expected path — to confirm on April 28."

The escalation threshold matters. ELO should NOT escalate for a minor delay — if Presten's session runs late, give it until 11:00 PM April 28 before triggering a NO-GO flag. Premature escalation is noise.
