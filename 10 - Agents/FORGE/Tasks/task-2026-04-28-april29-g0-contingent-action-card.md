---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: pending
priority: urgent
due: 2026-04-28 (complete before midnight so card is ready for April 29 open)
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/April 29 — FORGE G0-Contingent Action Card.md"
---

# Task: April 29 — FORGE G0-Contingent Action Card

## Context

Tonight at 11 PM, ELO either confirms G0 = GO or files G0 = NO-GO (DEFERRED). FORGE's April 29 work plan is entirely different depending on which outcome arrives. Right now, FORGE has response protocols and runbooks spread across multiple documents — but no single action card that resolves to a clear first-task sequence based on the G0 result.

This task produces that card. FORGE can write it entirely from existing documents (no new decisions required). It should be ready before midnight so FORGE opens April 29 with zero ambiguity about where to start.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/April 29 — FORGE G0-Contingent Action Card.md`

---

### Header

```
G0 Result: [GO / NO-GO / DEFERRED — fill when ELO files tonight]
Card valid as of: 2026-04-28
FORGE opens April 29 with: Branch A or Branch B (circle when G0 known)
```

---

### Branch A: G0 = GO

**First 30 minutes — priority order:**

| Step | Action | Source Document |
|------|--------|-----------------|
| 1 | Confirm `ecnl_verified BOOLEAN DEFAULT FALSE` column exists in production (psql check) | `Infrastructure/April 28 Execution Log.md` Section 3 |
| 2 | Run ECNL verified backfill SQL (prep completed in `April 28 Execution Log.md`) | `task-2026-04-27-ecnl-verified-backfill-sql-package.md` |
| 3 | Confirm FM1/FM2 activation paths written to `Infrastructure/Pipeline Code Review Results.md` | `task-2026-04-27-fm1-fm2-activation-confirmation-and-handoff.md` |
| 4 | File audit document updates for FM1 and FM2 per outcome-update protocol | `task-2026-04-26-audit-documents-outcome-update-protocol.md` |
| 5 | Notify ELO: ecnl_verified backfill complete → ELO can run ECNL CP1 staleness check | Direct message via briefing |

**Within 2 hours of Step 5:**

| Step | Action |
|------|--------|
| 6 | Read ELO's April 29 G1 result (Girls GA ASPIRE fix verified or anomaly flagged) |
| 7 | If anomaly: FORGE identifies whether pipeline is source of discrepancy (cross-check source event_tier values) |
| 8 | Update `Infrastructure/April 28 Session Outcome Report.md` Section 6 with G0 = GO confirmation |
| 9 | File April 29 FORGE briefing with G1 response documented |

**G0 = GO standing orders for remainder of April 29:**

- Monitor pipeline for any anomalies triggered by ecnl_verified backfill
- TYSA org-ID watch: config add within 30 min of receipt (Section 1 of TYSA Checklist already complete)
- Archive dry-run watch: file authorization request within 30 min of `--dry-run` output

---

### Branch B: G0 = NO-GO (Session Did Not Run by 11 PM)

**First 30 minutes — priority order:**

| Step | Action | Source Document |
|------|--------|-----------------|
| 1 | Read ELO's G0 = NO-GO filing in SENTINEL Queue | ELO files to `10 - Agents/SENTINEL/Queue/` |
| 2 | Assess May 1 deployment impact: does G0 = NO-GO block Stage 1 launch? | See impact assessment below |
| 3 | File FORGE's G0 = NO-GO impact assessment to `Infrastructure/April 29 G0 NO-GO Impact — FORGE Assessment.md` | Template in Section 3 of this card |
| 4 | Notify SENTINEL in briefing that FORGE assessment is filed | |
| 5 | Proceed to Boys Option A support (independent of G0) | `task-2026-04-27-ga-coverage-audit-boys-query-package.md` |

**G0 = NO-GO Impact Assessment Template (pre-fill now, confirm April 29):**

| Item | Impact if G0 = NO-GO | Mitigation |
|------|---------------------|------------|
| May 1 Stage 1 pipeline launch | NOT BLOCKED — Stage 1 is TYSA GotSport crawl, independent of April 28 session | Proceed pending TYSA org-ID only |
| ecnl_verified backfill | DELAYED — cannot run without ecnl_verified column confirmed | Blocks ECNL CP1 check; ELO notes accordingly |
| Girls calibration baseline | DELAYED — G1–G4 gates slip to April 30 at earliest | Post-DSS timeline unaffected if session runs before April 30 |
| Archive dry-run | BLOCKED — GROUP BY fix must be in production (April 28 session) | Cannot unblock until session runs |
| FM1/FM2 audit updates | BLOCKED | Cannot confirm until Pipeline Code Review Results filed |
| Event Strength Phase 1 authorization | BLOCKED | Downstream of G0 = GO |

**FORGE's April 29 autonomous work if G0 = NO-GO:**

1. Boys coverage audit support (GA boys query package review)
2. Non-GotSport Sources #3–5 synthesis (independent — see separate task)
3. Archive fix ready-state verification (independent — see separate task)
4. TYSA config add (executes moment org-ID arrives regardless of G0)

---

### Branch C: G0 Still Deferred at April 29 Open (No ELO Filing by Midnight)

- FORGE files G0 status check request in SENTINEL queue immediately at April 29 open
- Proceeds with Branch B autonomous work pending G0 resolution

---

## Definition of Done

- Card filed at deliverable path before midnight April 28
- Branch A and Branch B both fully populated (no blank cells)
- Impact assessment in Branch B pre-filled with "NOT BLOCKED / DELAYED / BLOCKED" judgments and mitigations
- FORGE April 29 opens by reading this card first, not by re-reading all prior runbooks

## Why This Matters

Without this card, FORGE enters April 29 doing reactive synthesis — reading 6 prior runbooks to reconstruct the action plan. This card does that synthesis once, tonight, from authoritative sources. The 11 PM G0 filing is the single fact that resolves the entire April 29 first-hour sequence.

## References

- `task-2026-04-27-april29-forge-execution-checklist.md` — detailed checklist (this card supersedes for first-hour sequencing)
- `task-2026-04-27-fm1-fm2-activation-confirmation-and-handoff.md` — FM1/FM2 confirmation steps
- `task-2026-04-27-ecnl-verified-backfill-sql-package.md` — ecnl_verified backfill SQL
- `task-2026-04-26-audit-documents-outcome-update-protocol.md` — audit update steps
- `10 - Agents/ELO/Tasks/task-2026-04-28-april28-g0-gate-confirmation.md` — G0 gate document
