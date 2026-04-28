---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-05-01
priority: medium
status: completed
completed: 2026-04-27
topic: club-rankings-go-no-go-recommendation
---

# Task: Club Rankings — Girls-Only Launch vs. Wait for Boys

## Context

ELO has produced both a Club Rankings Girls-Only Fallback Spec (`task-2026-04-27-club-rankings-girls-only-fallback-spec.md`) and a Club Rankings Boys Conditional Timeline (`task-2026-04-25-club-rankings-boys-conditional-timeline.md`). Two paths exist:

**Path A — Girls-Only Launch:** Ship Club Rankings for girls only at DSS (May 9). Boys rankings follow in a post-DSS sprint after Boys Option A is resolved and Boys calibration is confirmed.

**Path B — Wait for Both:** Delay Club Rankings entirely until boys are ready. Launch a complete, dual-gender Club Rankings system. This targets late May or early June — outside the DSS window.

Neither path has a formal recommendation on file. SENTINEL cannot include Club Rankings in the DSS demo planning without knowing which path ELO recommends.

This task produces that recommendation so SENTINEL and Presten can decide before May 1.

## Task

Produce `Rankings/Club Rankings — Girls-Only vs. Full Launch Recommendation.md`.

### Section 1: Decision Criteria

List the factors that determine which path is correct. At minimum, address:

- **Demo impact:** Is a girls-only Club Rankings a compelling enough DSS demo asset, or does the absence of boys rankings make it feel incomplete?
- **Technical readiness:** Is the girls-only Club Rankings implementation plan already de-risked? Would launching girls-only create debt that makes the boys integration harder later?
- **Boys timeline risk:** What is the probability that Boys Option A resolves in time to include boys rankings in a May 9 DSS demo? (Low / Medium / High — with reasoning)
- **Calibration dependency:** Does Club Rankings require calibration changes to be applied first? If yes, which changes, and are they ready?

### Section 2: Recommendation

State clearly: **Path A (Girls-Only at DSS)** or **Path B (Wait for Both)**.

Provide the specific reasoning: why this path and not the other. Cite any numbers (game counts, team counts, expected query results) that support the recommendation.

If the recommendation is conditional, state the precise condition that would change it.

### Section 3: Implementation Trigger

Based on the recommendation:
- **If Path A:** What is the first action ELO takes after receiving SENTINEL approval? What session does the implementation begin in?
- **If Path B:** What is the trigger date for starting the full dual-gender implementation? What must be true before ELO begins?

### Section 4: SENTINEL/Presten Decision Required

This recommendation requires approval before ELO acts. State clearly what is being asked:
- Approve Path A, or
- Approve Path B, or
- Request a third option (describe it)

## Deliverable

File to: `Rankings/Club Rankings — Girls-Only vs. Full Launch Recommendation.md`

Update this task to `status: completed` when filed.

## Notes

Due May 1 — needs to be in SENTINEL's hands with enough time to get Presten's approval before the May 9 DSS demo planning is finalized.

This task resolves a decision that has been implicitly deferred. The fallback spec and conditional timeline are both written — the missing piece is ELO's own recommendation for which path to take. SENTINEL will not push one path over the other until ELO's technical analysis is on file.
