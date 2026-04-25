---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-25
priority: medium
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 2 — Scope and Plan.md"
---

# Task: Event Strength Phase 2 — Scope and Plan

## Context

Event Strength Phase 1 is fully specced and execution-ready (`Rankings/Event Strength Phase 1 — Full Execution Package.md`). Phase 1 labels events with strength metrics but does NOT apply event strength to ranking computation — it is labeling only.

Phase 2 is the application step: using event strength to improve ranking accuracy (e.g., a win at a high-strength event should have more rating impact than a win at a low-strength event). This has been referenced but never scoped.

This task: scope Phase 2 now, before Phase 1 is implemented. The risk is that Phase 2's design might require schema changes to `event_strength` that conflict with Phase 1's DDL. If ELO designs Phase 2 after Phase 1 is live, a costly rebuild is possible. Scoping Phase 2 first eliminates that risk.

## What to Produce

Write: `02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 2 — Scope and Plan.md`

---

### Section 1: Phase 1 → Phase 2 Handoff

What Phase 1 delivers (pull from the execution package — do not re-derive):
- `event_strength` table schema and the 5 metrics it stores
- Events labeled with `strength_tier` (High, Above Average, Average, Below Average) and `strength_percentile`
- No change to the ranking computation pipeline — Phase 1 is read-only from rankings' perspective

What Phase 2 must add (ELO defines):

Write the one-sentence definition of Phase 2's primary change. Likely one of:
- "Phase 2 applies `strength_tier` to scale the K-factor used for games in each event"
- "Phase 2 adds a weight parameter to the Glicko update formula based on event strength percentile"
- "Phase 2 applies a post-update rating adjustment based on event quality"

State which interpretation ELO recommends and why. If uncertain between approaches, name the two options and the decision criteria.

### Section 2: Core Design — Applying Event Strength to Rankings

The central design question: how does an event's strength score affect a team's rating change from games played at that event?

Write the proposed formula. Example structure (ELO fills in the actual values):
```
K_adjusted = K_base × event_strength_factor
where event_strength_factor ∈ [min, max]
and event_strength_factor = f(strength_percentile)
```

Define:
- The scaling range: what is the maximum bonus for a High-strength event? What is the floor for a Low-strength event?
- Whether the scaling applies symmetrically to wins AND losses, or only to wins
- Whether teams rated far above/below event average get reduced adjustments (similar to existing K-factor behavior)

Calibrate the scaling range against the known system properties. Reference the Girls GA ASPIRE overcalibration case (40+ points of distortion was the signal for "something is wrong") and the existing K-factor values to bound the Phase 2 adjustment range.

### Section 3: Schema Impact Assessment

Does Phase 2 require any changes to the `event_strength` table DDL that Phase 1 defines?

- If yes: specify the column(s) to add and why Phase 1's DDL should include them proactively. Flag to SENTINEL immediately — this changes the Phase 1 execution package.
- If no: confirm explicitly that Phase 1 can deploy as-is and Phase 2 reads from it without schema modification.

This section is the primary reason this task exists. If SENTINEL learns Phase 2 needs a schema change after Phase 1 is live, that triggers a migration. If SENTINEL learns now, Phase 1 includes the correct schema from the start.

### Section 4: Implementation Estimate

Based on Phase 1 as reference (7 hours including schema, computation, and validation):

- Schema changes (if any from Section 3): [N hours]
- Computation change (modifying the ranking pipeline to read from `event_strength`): [N hours]
- Validation: [N hours — ELO defines what "Phase 2 is working correctly" looks like]
- Total estimated session time: [N hours]

### Section 5: Timeline Recommendation

Given:
- DSS is May 16 — Phase 1 is the DSS feature; Phase 2 is post-DSS
- Calibration changes deploy May 18–20 (do not run Phase 2 during calibration deployment window)
- Club Rankings session is May 2–3

ELO's recommendation for Phase 2 timing: [ELO defines — likely June, after DSS + calibration + Club Rankings are stable]. State the specific constraint that sets the earliest Phase 2 start date and the recommended target window.

## Definition of Done

- Document written at `Rankings/Event Strength Phase 2 — Scope and Plan.md`
- Phase 1 → Phase 2 handoff is explicitly defined
- Core formula or application method is stated — no vague "apply strength to rankings"
- Schema impact assessment answers the binary question: does Phase 1 DDL need to change?
- Implementation estimate is present with breakdown
- Timeline recommendation is stated with justification
- Summary in briefing: "Event Strength Phase 2 scoped. Primary change: [one sentence on application method]. Schema impact on Phase 1: [yes/no — specifics]. Estimated session: [N hours]. Recommended timing: [month/window]."

## Why This Matters

Phase 1 implements the `event_strength` table. Phase 2 reads from it. If Phase 2's design requires a column that Phase 1's DDL doesn't include, Phase 2 starts with a migration instead of a clean build. That's avoidable — but only if ELO designs Phase 2 before Phase 1 runs. This task exists entirely to close that architectural gap.

## References

- `Rankings/Event Strength Phase 1 — Full Execution Package.md` — Phase 1 DDL and computation
- `Rankings/Event Strength Ratings — Implementation Plan.md` — full feature scope
- `Rankings/League Hierarchy.md` — calibration values (context for Phase 2 scaling design)
- `Rankings/Calibration Validation 2026-04-22.md` — existing K-factor and calibration reference
