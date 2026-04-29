---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-29
priority: high
due: 2026-04-30 EOD
status: pending
deliverable: "02 - Tiger Tournaments/Projects/Rankings/ECNL Migration Option — Decision Brief.md"
tags: [elo, task, ecnl, migration, decision-gate, sentinel-gate]
---

# Task: ECNL Migration Option Decision Brief

## Context

The ECNL migration option decision is due April 30. ELO has filed a recommendation (Option 1, per `task-2026-04-28-ecnl-migration-option-recommendation.md`). However, the recommendation and supporting analysis are distributed across multiple files. SENTINEL cannot issue the April 30 authorization without a single consolidated brief.

This task produces that brief.

## What to Build

Produce: `02 - Tiger Tournaments/Projects/Rankings/ECNL Migration Option — Decision Brief.md`

A 1–2 page document. Designed to be read by Presten in under 3 minutes and by SENTINEL as the gate document for the April 30 ECNL migration option authorization.

## Required Format

### Header

```yaml
---
type: elo-decision-brief
topic: ecnl-migration-option
date: 2026-04-29
decision_deadline: 2026-04-30
elo_recommendation: Option 1 | Option 2
status: pending-sentinel-authorization
---
```

### Section 1: Decision Required

One sentence stating the decision. Example: "Presten must select Option 1 or Option 2 for ECNL migration before April 30 EOD; FORGE begins implementation immediately on decision receipt."

### Section 2: Options at a Glance

| | Option 1 | Option 2 |
|-|----------|----------|
| Approach | [1 sentence] | [1 sentence] |
| Rating continuity | [impact] | [impact] |
| FORGE implementation effort | [estimate] | [estimate] |
| Risk to May 9 rankings | [low/med/high] | [low/med/high] |
| ELO preference | **RECOMMENDED** | Not recommended |

### Section 3: ELO Recommendation

3–5 sentences. Why Option 1 (or whichever ELO recommends). What the single most important tradeoff is. What the cost of choosing Option 2 would be. Be direct — this is a decision brief, not a research summary.

### Section 4: What Happens After Decision

**If Option 1 selected:**
- FORGE implements [specific pipeline change]
- ELO monitors rating continuity via [CP1/CP2 checkpoints or equivalent]
- Expected completion: [date]

**If Option 2 selected:**
- FORGE implements [specific pipeline change]
- ELO monitors [what]
- Expected completion: [date]
- Additional risk: [state honestly]

### Section 5: FORGE Handoff Requirements

What does FORGE need from ELO or Presten to begin implementation?
- [ ] Option decision from Presten
- [ ] [Any ELO-specific inputs FORGE requires]
- [ ] [Any timing constraints]

### Section 6: SENTINEL Authorization Request

> ELO recommends Option [1/2] for ECNL migration. [One sentence rationale.] ELO requests SENTINEL authorize the Option [1/2] implementation path and direct FORGE to begin after April 30 decision receipt.

---

## Definition of Done

- All six sections are complete and accurate
- Options table is clear enough for Presten to decide without reading supporting documents
- FORGE handoff requirements are specific and actionable
- ELO confirms filing by April 30 EOD in next briefing

## References

- `task-2026-04-28-ecnl-migration-option-recommendation.md` — ELO's recommendation
- `task-2026-04-26-ecnl-migration-option-comparison-matrix.md` — comparison matrix
- `task-2026-04-26-ecnl-migration-option-selection-decision-gate.md` — decision gate spec
- `task-2026-04-24-ecnl-rating-continuity-spec.md` — rating continuity requirements
- `task-2026-04-28-ecnl-migration-elo-forge-handoff-checklist.md` — handoff requirements
