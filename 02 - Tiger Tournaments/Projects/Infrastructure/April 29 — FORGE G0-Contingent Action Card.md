---
title: April 29 — FORGE G0-Contingent Action Card
type: execution-card
agent: FORGE
filed: 2026-04-28
card_valid_as_of: 2026-04-28
status: ready
---

# April 29 — FORGE G0-Contingent Action Card

```
G0 Result: [GO / NO-GO / DEFERRED — fill when ELO files tonight ~11 PM]
Card valid as of: 2026-04-28
FORGE opens April 29 with: Branch A or Branch B (circle when G0 known)
```

> ELO files G0 at 11 PM April 28. FORGE opens April 29 by reading the ELO queue, circling the branch, and executing immediately. Zero synthesis required at session open.

---

## Branch A: G0 = GO

**First 30 minutes — priority order:**

| Step | Action | Source Document |
|------|--------|-----------------|
| 1 | Confirm `ecnl_verified BOOLEAN DEFAULT FALSE` column exists in production: `\d games` in psql, look for ecnl_verified column | `Infrastructure/April 28 Execution Log.md` Section 3 |
| 2 | Run ECNL verified backfill SQL from the backfill package | `task-2026-04-27-ecnl-verified-backfill-sql-package.md` |
| 3 | Confirm FM1/FM2 activation paths written to `Infrastructure/Pipeline Code Review Results.md` — verify both FM1 and FM2 rows are filled | `task-2026-04-27-april28-execution-log-template.md` Section FM column |
| 4 | File audit document updates for FM1 and FM2 per outcome-update protocol (one update per audit doc) | `task-2026-04-26-audit-documents-outcome-update-protocol.md` |
| 5 | Notify ELO in FORGE briefing: ecnl_verified backfill complete → ELO can proceed with ECNL CP1 staleness check | FORGE briefing sent same session |

**Within 2 hours of Step 5:**

| Step | Action |
|------|--------|
| 6 | Read ELO's April 29 G1 result (Girls GA ASPIRE fix verified or anomaly flagged) from ELO briefing/queue |
| 7 | If anomaly flagged by ELO: FORGE cross-checks source event_tier values for GA ASPIRE games to determine if pipeline is source of discrepancy |
| 8 | Update `Infrastructure/April 28 Session Outcome Report.md` Section 6 with G0 = GO confirmation and actual execution timestamps |
| 9 | File April 29 FORGE briefing with G0 = GO status, ecnl_verified backfill result, and G1 response documented |

**G0 = GO standing orders for remainder of April 29:**

- Monitor pipeline for anomalies triggered by ecnl_verified backfill (watch for unexpected NULL counts post-backfill)
- **TYSA org-ID watch:** Config add within 30 min of receipt — checklist is at `Infrastructure/TYSA Config Add — Execution Checklist.md`
- **Archive dry-run watch:** File authorization request within 30 min of `--dry-run` output — template at `Infrastructure/Archive Inactive Teams — Dry-Run Authorization Request.md`

---

## Branch B: G0 = NO-GO (Session Did Not Run by 11 PM)

**First 30 minutes — priority order:**

| Step | Action | Source Document |
|------|--------|-----------------|
| 1 | Read ELO's G0 = NO-GO filing in SENTINEL Queue at `10 - Agents/SENTINEL/Queue/` | ELO files here if G0 = NO-GO |
| 2 | Assess May 1 deployment impact — use impact table below | Section below |
| 3 | File FORGE's G0 = NO-GO impact assessment to `Infrastructure/April 29 G0 NO-GO Impact — FORGE Assessment.md` | Template in Branch B below |
| 4 | Notify SENTINEL in FORGE briefing that assessment is filed | FORGE briefing same session |
| 5 | Proceed to Boys Option A support (independent of G0) | `task-2026-04-27-ga-coverage-audit-boys-query-package.md` |

**G0 = NO-GO Impact Assessment (pre-filled — confirm at April 29 open):**

| Item | Impact if G0 = NO-GO | Mitigation |
|------|---------------------|------------|
| May 1 Stage 1 pipeline launch | **NOT BLOCKED** — Stage 1 is TYSA GotSport crawl, fully independent of April 28 session items | Proceed pending TYSA org-ID only |
| ecnl_verified backfill | **DELAYED** — cannot run without column confirmed present in schema | Blocks ECNL CP1 check; ELO notes staleness check is deferred accordingly |
| Girls calibration baseline (G1–G4) | **DELAYED** — gates slip to April 30 at earliest; May 1 deployment timeline unaffected | Post-DSS timeline unaffected if session executes before April 30 |
| Archive dry-run | **BLOCKED** — GROUP BY fix must be confirmed in production code (April 28 session) | Cannot unblock until session runs and fix is applied |
| FM1/FM2 audit document updates | **BLOCKED** — cannot confirm path until Pipeline Code Review Results is filled | Both FM1 and FM2 audits remain Outcome C — open |
| ECNL Migration FORGE prep | **NOT BLOCKED** — impact spec already filed; awaiting ELO decision separately | ELO's Option A/B/C decision independent of April 28 session |

**FORGE's April 29 autonomous work if G0 = NO-GO:**

1. Boys coverage audit support — read `task-2026-04-27-ga-coverage-audit-boys-query-package.md` and produce query package for ELO
2. Non-GotSport Sources #3–5 synthesis — file document to `Infrastructure/Non-GotSport Sources #3–5 — Candidate Synthesis and Recommendation.md`
3. Archive SQL fix ready-state verification — file document to `Infrastructure/Archive Inactive Teams — SQL Fix Verification and Dry-Run Readiness.md`
4. TYSA config add — executes immediately on org-ID receipt regardless of G0 status

---

## Branch C: G0 Still Deferred at April 29 Open (No ELO Filing by Midnight April 28)

- File FORGE G0 status check request in SENTINEL queue at April 29 session open (priority: urgent)
- Proceed with Branch B autonomous work pending G0 resolution
- Escalate if no ELO filing by April 29 noon — May 1 deployment timeline is still intact but ecnl_verified backfill gap widens daily

---

## Quick-Reference: What TYSA Org-ID Receipt Triggers (regardless of G0 branch)

| Step | Time Budget | Reference |
|------|------------|-----------|
| Open TYSA Config Add Execution Checklist | 0 min | `Infrastructure/TYSA Config Add — Execution Checklist.md` |
| Add TYSA to config + syntax check | 5 min | Section 2 Steps 1–3 of checklist |
| Sign May 1 Stage 1 Config Final Review Section 5 | 5 min | `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md` |
| File confirmation in next FORGE briefing | Within 2 hours | Briefing template in checklist Section 2 Step 5 |

**Total from org-ID receipt to certification signed: < 30 minutes.**

---

## Quick-Reference: What Archive Dry-Run Output Receipt Triggers (regardless of G0 branch)

| Scenario | FORGE Response | Time Budget | Reference |
|----------|---------------|------------|-----------|
| GREEN (80–150 teams) | File dry-run authorization request to SENTINEL | < 30 min | `Infrastructure/Archive Inactive Teams — Dry-Run Authorization Request.md` |
| YELLOW (50–80 or 150–200) | File authorization with anomaly note; SENTINEL decides | < 1 hour | Same template |
| RED (< 50 or > 200, or SQL error) | File defect report; do NOT request authorization | < 2 hours | File to SENTINEL queue |

---

## References

- `task-2026-04-27-april29-forge-execution-checklist.md` — detailed checklist (this card supersedes for first-hour sequencing)
- `task-2026-04-27-fm1-fm2-activation-confirmation-and-handoff.md` — FM1/FM2 confirmation steps
- `task-2026-04-27-ecnl-verified-backfill-sql-package.md` — ecnl_verified backfill SQL
- `task-2026-04-26-audit-documents-outcome-update-protocol.md` — audit update steps
- `10 - Agents/ELO/Tasks/task-2026-04-28-april28-g0-gate-confirmation.md` — G0 gate document (ELO files here)

*FORGE — 2026-04-28 @ 17:25*
