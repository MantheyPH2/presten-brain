---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: completed
priority: high
due: 2026-05-02
completed: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Club Rankings — May Launch Conditions.md"
---

# Task: Club Rankings — May Launch Conditions

## Context

Club Rankings have been in development since April 23. The implementation plan, reference clubs spec, girls-only conditional scope, and boys conditional timeline all exist. What does not exist: a clear, SENTINEL-readable decision document that states the minimum conditions for Club Rankings to launch, and whether those conditions are currently met.

Right now, SENTINEL does not know whether Club Rankings is on track to be part of the May 9 readiness assessment or the May 16 DSS demo. The existing specs define how Club Rankings would work — not whether they are ready.

This task: ELO files a go/no-go decision document for Club Rankings, covering Girls scope (the safer launch), the specific data requirements that must be satisfied, and the earliest realistic launch date.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Club Rankings — May Launch Conditions.md`

---

### Section 1: Current Status Summary

ELO states the current state of Club Rankings:
- Implementation spec: filed / not filed
- Reference club set: confirmed / in review / not confirmed
- Girls-only SQL: written / not written
- Last test run: [date] / never run
- Blocking issues outstanding: [list or NONE]

---

### Section 2: Launch Conditions — Girls Only

Minimum conditions for Girls Club Rankings to launch:

| Condition | Required | Current Status |
|-----------|----------|---------------|
| Reference clubs confirmed (Girls set) | YES | MET / NOT MET |
| Girls club ranking SQL written and tested on staging | YES | MET / NOT MET |
| Output covers ≥ 80% of Girls ECNL, MLS NEXT clubs | YES | MET / NOT MET |
| No club ranking anomalies in top-20 (spot-checked) | YES | MET / NOT MET |
| Rank bands integration confirmed for club view | YES | MET / NOT MET |
| SENTINEL authorization received | YES | PENDING |

**Girls Club Rankings launch: READY / NOT READY**

If NOT READY: list specifically what must happen and by when.

---

### Section 3: Launch Conditions — Boys

| Condition | Required | Current Status |
|-----------|----------|---------------|
| Boys Option A verdict: PASS | YES | PENDING (April 29) |
| Boys reference clubs confirmed | YES | |
| Boys club ranking SQL tested | YES | |

**Boys Club Rankings:** Deferred until Boys Option A confirmed. Earliest launch: [date based on Option A verdict + implementation time].

---

### Section 4: DSS Demo — Club Rankings Inclusion Decision

Should Club Rankings be included in the May 16 DSS demo?

ELO's recommendation: YES / NO / CONDITIONAL

If YES: which feature specifically? Club leaderboard? Search by club? Rank band for a club?
If CONDITIONAL: what must be true by May 9 for inclusion to stand?
If NO: reason (e.g., "Girls club data sufficient but Boys absent — mixed signal could confuse demo audience")

---

### Section 5: Earliest Launch Date

Given the conditions in Section 2:

**Earliest Girls Club Rankings launch date:** [date]
**Earliest Boys Club Rankings launch date (conditional on Option A):** [date]
**Recommended launch sequence:** Girls first → Boys on confirmation → Combined view TBD

---

### Section 6: What SENTINEL Should Authorize

Specific authorization request ELO is making:
1. Girls Club Rankings SQL testing on staging — authorize NOW (no production risk)
2. Girls Club Rankings production launch — authorize by [date] if Section 2 conditions met
3. Boys Club Rankings — authorize after Boys Option A verdict April 29

---

## Definition of Done

- Document filed at deliverable path by May 2
- All conditions in Section 2 have explicit MET / NOT MET status
- DSS inclusion recommendation is explicit (YES / NO / CONDITIONAL)
- Earliest launch dates are stated as specific dates, not relative ("2–3 days")
- ELO briefing May 2 references this document and states whether SENTINEL authorization has been requested

## Why This Matters

Club Rankings is one of the highest-value features for the DSS demo — it answers "which club is best in my area?" SENTINEL currently has no visibility on whether this feature makes the demo or not. The implementation work is happening in parallel with everything else; without a go/no-go document, there is no checkpoint. This document creates the checkpoint and puts the authorization decision in SENTINEL's hands before it becomes a last-minute scramble.

## References

- `task-2026-04-25-club-rankings-implementation-plan.md` — implementation spec
- `task-2026-04-25-club-rankings-reference-clubs.md` — reference club set
- `task-2026-04-27-club-rankings-girls-only-fallback-spec.md` — girls-only scope
- `task-2026-04-25-club-rankings-boys-conditional-timeline.md` — boys timeline
- `task-2026-04-24-club-rankings-implementation-spec.md` — technical spec
- `task-2026-04-25-dss-feature-readiness-tracker.md` — DSS readiness tracker
