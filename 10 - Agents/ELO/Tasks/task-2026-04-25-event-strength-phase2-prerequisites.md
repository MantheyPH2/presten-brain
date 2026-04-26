---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: completed
priority: medium
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 2 — Prerequisites and Scope.md"
---

# Task: Event Strength Phase 2 — Prerequisites and Scope

## Context

Event Strength Phase 1 has a filed execution package and SENTINEL authorization criteria. Phase 1 assigns per-event quality ratings based on the finalized league hierarchy calibration values.

Event Strength Phase 2 has no defined scope, no prerequisites, and no timeline. SENTINEL does not know:
- What Phase 2 changes relative to Phase 1
- Whether Phase 2 is DSS-relevant (i.e., must it complete before May 16?)
- What conditions must hold before Phase 2 is safe to run
- Whether Phase 2 depends on Boys Option A outcome

If Phase 2 is not DSS-relevant, it can be deferred post-May 16 and this task's deliverable becomes a planning document for the summer. If Phase 2 IS DSS-relevant, SENTINEL needs to know now — before the May 9 readiness gate — so it can be incorporated into the DSS timeline.

This task resolves the ambiguity. ELO defines Phase 2 scope and prerequisites; SENTINEL determines DSS relevance from those definitions.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 2 — Prerequisites and Scope.md`

---

### Section 1: Phase 2 Scope Definition

What exactly does Event Strength Phase 2 change relative to Phase 1?

ELO answers:
- Does Phase 2 add per-game strength-of-schedule adjustments, or does it modify the Phase 1 event-level ratings?
- Does Phase 2 affect Girls only, Boys only, or both genders?
- Does Phase 2 require new data inputs (new source fields, cross-league win-rate updates) that are not in the current database?
- Is Phase 2 reversible without a full recompute?
- Estimated computation time for Phase 2 (FORGE execution session hours)?

If Phase 2 scope is not yet defined: ELO states the design options and recommends a scope. SENTINEL does not need to confirm the design; it needs to know what it would authorize.

---

### Section 2: DSS Relevance Assessment

Does Phase 2 need to complete before May 16 DSS?

ELO assesses each DSS accuracy claim from the DSS Ranking Accuracy Claims spec:
- Which claims (if any) are materially improved by Phase 2?
- Which claims are fully supported by Phase 1 alone?
- If Phase 2 were deferred to post-DSS, would any authorized DSS claim become inaccurate or unsupportable?

Verdict: `Phase 2 is DSS-critical` / `Phase 2 is DSS-beneficial but not required` / `Phase 2 is post-DSS`.

If DSS-critical: ELO states the latest possible Phase 2 authorization date given the May 9 readiness gate and the pipeline schedule.

---

### Section 3: Prerequisites

What conditions must hold before Phase 2 runs safely?

Structure as a table (same format as the Phase 1 Authorization Criteria doc):

| # | Condition | Source of Confirmation | Pass Threshold | Notes |
|---|-----------|----------------------|----------------|-------|
| 1 | Phase 1 complete and validated | Phase 1 post-validation note | ELO clearance filed | Phase 2 cannot run concurrently with Phase 1 validation |
| 2 | Calibration stability confirmed post-Phase-1 | Post-Fix Calibration Stability Monitoring Spec | [N] consecutive clean runs | Phase 2 introduces a second calibration variable; system must be stable first |
| 3 | Boys Option A verdict filed | Boys Option A Verdict Filing Document | PASS or FAIL declared (not pending) | Phase 2 should not run during an open Boys investigation |
| 4 | [Additional ELO-defined conditions] | | | |

ELO also states any **hard blockers** — conditions where Phase 2 must NOT run regardless of other criteria:
- Any active RED pipeline state
- Within [N] hours of a Boys Option A investigation active
- [Other ELO-defined blockers]

---

### Section 4: Timeline Options

Given DSS relevance assessment and prerequisites:

**If Phase 2 is DSS-critical:**
- Earliest possible authorization: [date, given prerequisites]
- Latest authorization for May 9 gate: [date, working back from May 9]
- Risk if Phase 2 slips: [ELO describes]

**If Phase 2 is post-DSS:**
- Recommended authorization window: [summer timeline]
- Dependencies: [what must complete first — Boys Option A, Club Rankings, etc.]
- No risk to May 9 gate

---

### Section 5: SENTINEL Authorization Gate (skeleton)

ELO writes the gate conditions in the same format as the Phase 1 Authorization Criteria document. SENTINEL will not authorize Phase 2 without a formal gate, but the gate cannot be written until scope is defined. This section creates the gate structure; SENTINEL will add its disposition block when Phase 2 is ready.

```
Phase 2 Authorization Request — ELO
Date: ___________
Phase 1 clearance: confirmed ✅ [date]
Calibration stability: [N] consecutive clean runs ✅ [date]
Boys Option A verdict: [PASS/FAIL] filed [date] ✅
[Additional conditions: ✅]
Phase 2 scope: [one-line description]
Expected Phase 2 effects: [rating impact summary]
SENTINEL authorization request: Phase 2 authorized to run from [date].
```

---

## Quality Standard

- Section 2 DSS relevance verdict must be a single statement: critical / beneficial / post-DSS. No hedging.
- Section 3 prerequisites must use explicit thresholds (same standard as Phase 1 Authorization Criteria).
- If Phase 2 scope is genuinely undefined at the time of writing: ELO states the design options, recommends one, and writes the prerequisites for that recommended scope. Do not defer the document because scope is uncertain.
- Timeline must state specific dates, not "after Phase 1 completes."

## Why This Matters

SENTINEL currently has no view of Phase 2. If Phase 2 is DSS-critical and nobody identifies this before May 9, the readiness gate passes without Phase 2 on the checklist — and the DSS demo runs on Phase 1 data when Phase 2 was required. Discovering this on May 10 is too late to fix. This document surfaces the Phase 2 question now, while the timeline is still actionable. If Phase 2 is not DSS-relevant, this document takes 30 minutes to write and saves nothing; that is an acceptable cost. If Phase 2 IS DSS-relevant, this document saves the DSS timeline.

## References

- `Rankings/Event Strength — Phase 1 Execution Package.md` — Phase 1 scope that Phase 2 extends
- `Rankings/Event Strength Phase 1 — SENTINEL Authorization Criteria.md` — gate format to replicate for Phase 2
- `Rankings/DSS Ranking Accuracy Claims — Authorized Spec.md` — claims ELO assesses in Section 2
- `Rankings/Club Rankings — Boys-Conditional Implementation Timeline.md` — Boys dependency ELO assesses in Section 3
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Phase 2 feature cell to update
