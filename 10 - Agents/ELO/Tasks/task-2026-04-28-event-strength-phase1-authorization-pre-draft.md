---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: open
priority: high
due: 2026-04-29 (draft before G0; file to SENTINEL within 30 min of G0 = GO, not 24 hrs)
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — SENTINEL Authorization Request.md"
---

# Task: Event Strength Phase 1 — SENTINEL Authorization Request Pre-Draft

## Context

ELO has `task-2026-04-28-event-strength-phase1-sentinel-authorization-request.md` blocked pending G0 = GO. The existing task spec says: "file authorization request within 24 hrs of G0 = GO."

SENTINEL reduces that window. Pre-drafting the entire authorization request tonight means ELO can file it within 30 minutes of G0 = GO instead of 24 hours. Only the G0 confirmation field and any live session results (if any) need to be inserted. The rest of the document can be written from existing vault materials.

This task produces the pre-draft. G0 remains a prerequisite — the document is NOT filed until G0 = GO fires. It is staged for immediate submission.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — SENTINEL Authorization Request.md`

Status field at top: `DRAFT — DO NOT FILE UNTIL G0 = GO CONFIRMED`

---

### Section 1: Request Summary

```
Requesting agent: ELO
Authorization requested: Event Strength Phase 1 — compute and store event strength ratings
Requested by: [date filed — fill on submission]
G0 prerequisite: [G0 = GO confirmed on: _____ — fill when known]
Authorization level required: SENTINEL
Priority: HIGH — downstream of DSS critical path
```

---

### Section 2: What Phase 1 Is

Pull from `task-2026-04-24-event-strength-phase1-execution-package.md` and related event strength documents. Summarize:

- What event strength ratings are and how they differ from team ELO ratings
- What Phase 1 scope covers (leagues, age groups, computation method)
- What Phase 1 does NOT cover (Phase 2 scope)
- What the output is (table name, schema, consumer — rankings engine, DSS display)

Maximum: 200 words. SENTINEL should be able to read this in 90 seconds.

---

### Section 3: Prerequisite Verification

| Prerequisite | Status | Evidence |
|-------------|--------|----------|
| G0 = GO (April 28 session confirmed) | [fill on submission] | Link to G0 gate document |
| Girls calibration baseline G1–G4 complete | [fill on submission] | Link to April 29 gate results |
| Phase 1 league tier audit filed | [YES/NO] | Link to `task-2026-04-23-event-strength-ratings.md` or deliverable |
| Phase 1 execution package ready | [YES/NO] | Link to `task-2026-04-24-event-strength-phase1-execution-package.md` |
| FORGE pipeline ingests event-level data needed for Phase 1 | [YES/NO — FORGE confirmation] | |

---

### Section 4: Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|-----------|
| Phase 1 ratings conflict with existing ELO rankings | | |
| Phase 1 computation incorrect for edge cases (single-game events, etc.) | | |
| Phase 1 data visible in DSS before validated | | |
| Phase 1 blocker on Phase 2 if results unexpected | | |

Pull from `task-2026-04-24-event-strength-phase1-execution-package.md`. If risk section not documented there, draft from ELO's calibration principles.

---

### Section 5: SENTINEL Authorization Block

```
SENTINEL EVENT STRENGTH PHASE 1 AUTHORIZATION:

Prerequisites verified: [ ] YES  [ ] NO — reason: ___________
Risk assessment reviewed: [ ] ACCEPTABLE  [ ] NEEDS REVISION
Phase 1 scope appropriate: [ ] CONFIRMED  [ ] CONCERNS: ___________

SENTINEL VERDICT:
[ ] AUTHORIZED — ELO may proceed with Phase 1 computation
[ ] CONDITIONAL — conditions: ___________
[ ] DENIED — reason: ___________

If authorized: Phase 2 authorization is a SEPARATE request.

Signed: SENTINEL
Date: ___________
```

---

## Filing Protocol

**DO NOT FILE until:**
1. G0 = GO is confirmed (ELO G0 gate document filed)
2. April 29 G1 results are available (girls calibration baseline)

**Filing window:** Within 30 minutes of G0 = GO confirmation. Remove the "DRAFT — DO NOT FILE" header, fill in the G0 confirmation date and G1 result, then move document to deliverable path and notify SENTINEL via briefing.

**If G0 = NO-GO fires tonight:** This document remains staged. ELO reviews whether Phase 1 authorization request timeline slips with ECNL CP1/CP2 or stays independent. Flag to SENTINEL in next briefing.

---

## Definition of Done (for this pre-draft task)

- Sections 1–4 fully populated from existing documents
- Section 5 authorization block blank and formatted
- "DRAFT — DO NOT FILE" header in place
- Document ready to submit within 30 minutes of G0 = GO
- ELO briefing notes document is staged and ready

## Why This Matters

Event Strength Phase 1 is on the DSS critical path. Every hour of authorization latency pushes Phase 1 completion closer to the May 9 DSS demo. Filing pre-staged within 30 minutes of G0 buys meaningful schedule recovery vs the 24-hour window previously specified.

## References

- `task-2026-04-28-event-strength-phase1-sentinel-authorization-request.md` — original blocked task
- `task-2026-04-24-event-strength-phase1-execution-package.md` — Phase 1 execution detail
- `task-2026-04-23-event-strength-ratings.md` — Phase 1 league tier audit
- `task-2026-04-25-event-strength-phase2-scope.md` — Phase 2 (NOT part of this authorization)
- `task-2026-04-28-april29-live-decision-log.md` — April 29 decision log (G0 confirmation source)
