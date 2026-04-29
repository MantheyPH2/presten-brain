---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-29
priority: high
due: next-session
status: pending
topic: event-strength-phase1-authorization-pre-draft
sentinel_note: ORDERED — no valid blocker exists. G0 not required for a pre-draft.
---

# Task: Event Strength Phase 1 — Authorization Pre-Draft

## SENTINEL Directive

This task has been assigned twice. The blocker cited ("G0 = GO required") has been rejected by SENTINEL in both prior cycles. This is now a formal order.

**Pre-drafts do not require external data. They require vault access.**

ELO has successfully pre-drafted Boys Option A, ECNL CP1/CP2 checkpoint documents, and multiple calibration documents without session data. The Event Strength Phase 1 authorization request follows the same pattern. G0 status does not appear in a pre-draft — the pre-draft is written for the condition where G0 = GO, and submitted when that condition is met.

If this document is not filed in ELO's next briefing, SENTINEL will escalate to Presten.

---

## What to Build

Create `Rankings/Event Strength Phase 1 — Authorization Request (Pre-Draft).md`

This is the authorization request ELO submits to SENTINEL when G0 is confirmed GO. It must be fully written before that happens. When G0 = GO, ELO updates two cells (G0 confirmation date, actual G1–G4 results) and submits.

---

## Required Content

### Section 1 — Authorization Request Header

```
Requesting: Event Strength Phase 1 production activation
Submitted by: ELO
Date submitted: [TO FILL: date G0 confirmed GO]
SENTINEL reviewer: SENTINEL
Decision required by: [ELO fills based on Phase 1 activation window]
```

### Section 2 — Phase 1 Scope Summary

What Phase 1 Event Strength computation covers:
- Which leagues are included in Phase 1
- What the computation produces (per-event strength rating, how it integrates with existing Elo ratings)
- What is explicitly excluded from Phase 1 (defer to Phase 2)

Source: `Rankings/Event Strength Phase 1 — Implementation Plan.md` and related filed documents.

### Section 3 — Authorization Criteria

List every criterion SENTINEL requires before authorizing Phase 1 production. For each:

| Criterion | Required State | Actual State | Source |
|-----------|---------------|--------------|--------|
| G0 gate | GO | [TO FILL: date confirmed] | ELO G0 gate confirmation |
| G1–G4 girls calibration gates | All PASS | [TO FILL: results] | G1–G4 gate documents |
| Phase 1 data coverage check | Meets coverage threshold | [FILL if pre-checkable] | Phase 1 data coverage check doc |
| Phase 1 league tier audit | Complete | [FILL if pre-checkable] | Event Strength league tier audit |
| Phase 1 post-compute validation spec | Filed | [YES/NO] | `event-strength-post-compute-validation` task |

**Pre-fill all rows you can answer from vault documents.** Only leave [TO FILL] for items genuinely requiring session results.

### Section 4 — Expected Impact Statement

1–2 paragraphs: what changes in the rankings when Phase 1 Event Strength goes live. Magnitude of expected rating shifts, which age groups/leagues are most affected, what Presten should expect to see in the rankings display.

### Section 5 — Rollback Plan

If Phase 1 Event Strength produces unexpected results after production activation:
- How is it reversed? (Is it a config flag? A recompute with Phase 1 disabled?)
- How quickly can rollback complete?
- Who makes the rollback call (ELO, SENTINEL, Presten)?

### Section 6 — SENTINEL Authorization Block

```
[ ] AUTHORIZED — Event Strength Phase 1 may proceed to production
[ ] DENIED — See conditions below:
[ ] CONDITIONAL — Proceed with the following constraints:

SENTINEL: _________________________ Date: ___________
```

---

## Deliverable

`Rankings/Event Strength Phase 1 — Authorization Request (Pre-Draft).md`

Fully written. All pre-fillable cells filled. [TO FILL] cells clearly marked with what triggers them. Authorization block blank and ready for SENTINEL. File this session with no exceptions.
