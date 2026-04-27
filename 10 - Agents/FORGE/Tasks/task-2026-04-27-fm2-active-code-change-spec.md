---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/FM2 Active — Code Change Specification.md"
---

# Task: FM2 Active — Code Change Specification

## Context

April 28 is the GA ASPIRE fix session. Before it runs, Presten must confirm FM1 vs. FM2 via the Quick-Run Guide. If FM1 = Path A (archive script), no code change is needed — the DB UPDATE is sufficient. If FM1 = Path B or FM2 is confirmed active, the recompute may produce incorrect results without a `compute-rankings.js` change.

The FM1/FM2 Contingency Action Plan (filed April 26) handles the decision tree: "if FM2 active, file SENTINEL-alert and hold." That is the right governance response. But it defers the technical spec to a future session. If FM2 turns out to be active, there will be pressure to resolve it same-day rather than waiting for a new session. FORGE should have the code change spec ready before April 28 closes, so that if Presten and SENTINEL authorize the change, FORGE can act within hours rather than scheduling a design session.

What does not yet exist: the specific, reviewable code change specification for the FM2 code path.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/FM2 Active — Code Change Specification.md`

---

### Section 1: FM2 Code Path Description

Describe what FM2 does, based on the FM1/FM2 Quick-Run Guide results and FORGE's knowledge of the codebase structure. Specifically:

- Which file contains FM2? (expected: `compute-rankings.js` or adjacent ranking computation file)
- What does FM2 do that FM1 does not? (expected: FM2 may handle stale team archival or team exclusion logic differently)
- What is the consequence for the April 28 recompute if FM2 runs with the old behavior?

If FORGE cannot determine the FM2 behavior without running the code, state that explicitly and write a "pending confirmation" placeholder — but complete all other sections.

---

### Section 2: Proposed Code Change

Provide the specific code change needed if FM2 is active and the current behavior is incorrect:

- **File:** exact file path
- **Function or line range:** be specific
- **Current behavior:** what the code does now
- **Required behavior:** what it should do after the change
- **Change type:** additive (new conditional, new flag check), modification (parameter change), or removal (dead code path suppression)

If FORGE determines the change requires access to the running codebase (not available in vault), state that explicitly and write the change in pseudocode with notes on what Presten would need to confirm.

---

### Section 3: Test Plan

How does FORGE confirm the code change works correctly before the full recompute runs?

- **Smoke test:** 1–2 queries or log checks that confirm FM2 now routes to the correct path
- **Expected vs. actual:** what output indicates the change worked
- **Rollback trigger:** if the smoke test fails, what is the rollback command?

---

### Section 4: Rollback Plan

If FM2 change is deployed and produces unexpected results in the recompute:

1. What is the rollback command? (git revert, config flag, or DB undo)
2. How long does rollback take?
3. Does rollback require another recompute? If so, how long?
4. Is the DB state recoverable to pre-April-28 state, or does rollback only stop further damage?

---

### Section 5: SENTINEL Authorization Request

Write the exact text FORGE would send to SENTINEL if FM2-active is confirmed and the code change is needed:

> "FM2-active confirmed. Code change specified at [file]:[line]. Change is [type]. Test plan: [smoke test]. Rollback: [command]. Requesting authorization to deploy before April 28 recompute."

Pre-writing this ensures FORGE doesn't improvise the authorization request under session pressure.

---

## Quality Standard

- Section 2 must contain a specific file path, not "probably in compute-rankings.js."
- If FORGE genuinely cannot determine the file path without running the code, say so clearly in Section 2 rather than guessing. The value of this document is honest specificity — a spec that names the wrong file is worse than an honest "unknown."
- Section 4 rollback plan must include a time estimate. "Rollback takes ~10 minutes + 45-minute recompute" is the level of specificity needed.

## Why This Matters

If FM2 is confirmed active on April 28 and FORGE has no code change spec, there are two bad outcomes: (1) proceed without the fix and run a recompute that produces incorrect results, or (2) hold the session until FORGE completes a design session — adding a day to the critical path. This spec eliminates outcome (2). It is a 1–2 hour deliverable that removes a potential day-delay from the most critical session in the pre-DSS plan.

## References

- `Infrastructure/FM1-FM2 Code Search — Presten Quick-Run Guide.md`
- `Infrastructure/FM1-FM2 Contingency Action Plan.md`
- `Infrastructure/Pipeline Code Review Results.md` — fill this before running the Quick-Run Guide
- SENTINEL 05:24 briefing — FM1/FM2 flagged for fourth consecutive session as the April 28 GO/HOLD gate
