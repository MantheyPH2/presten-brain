---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: completed
completed: 2026-04-28
priority: high
due: 2026-04-29 (before Presten's April 30 ECNL decision window)
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/ECNL Migration — Option B FORGE Pipeline Impact Brief.md"
---

# Task: ECNL Migration — Option B FORGE Pipeline Impact Brief

## Context

ELO filed an Option A recommendation (preserve ratings, no reset) and has pre-staged the Option A implementation prep package. Presten's decision is due April 30. ELO's Option A package is ready to activate immediately.

What does not exist: a FORGE assessment of what Option B (rating reset) requires from the pipeline side. If Presten selects Option B, FORGE should be able to respond same-session rather than spending the first 30 minutes diagnosing pipeline impact.

This task produces that assessment now, before the decision. It does not advocate for either option — it documents pipeline impact only.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/ECNL Migration — Option B FORGE Pipeline Impact Brief.md`

### Section 1: Option B Definition (from ELO's recommendation document)

Pull Option B definition from `10 - Agents/ELO/Tasks/task-2026-04-28-ecnl-migration-option-recommendation.md` or ELO's filed recommendation. Summarize in 2–3 sentences: what a rating reset means, what tables are affected, what the cutover date is.

### Section 2: FORGE Pipeline Changes Required for Option B

Answer these questions from existing FORGE documents (pipeline code review results, FM1/FM2 audit docs, ECNL migration pipeline impact spec):

| Question | Answer | Source |
|----------|--------|--------|
| Does Option B require schema changes beyond `ecnl_verified`? | | |
| Are there pipeline game ingestion changes required? | | |
| Does the archive dry-run interact with the reset? (timing risk) | | |
| Does Stage 1 launch on May 1 create any Option B conflicts? | | |
| Does the FM1/FM2 activation path change under Option B? | | |

### Section 3: Option B vs Option A — FORGE Effort Delta

| Item | Option A FORGE Work | Option B FORGE Work | Delta |
|------|--------------------|--------------------|-------|
| Schema changes | | | |
| Pipeline code changes | | | |
| Data backfill | | | |
| Coordination with ELO | | | |
| Timeline to June 2 readiness | | | |

### Section 4: FORGE Position on Option B Feasibility

One paragraph. Not advocacy — feasibility. Can FORGE complete required changes in time for June 2 under Option B? What is the critical path dependency?

### Section 5: FORGE Authorization Block (for SENTINEL)

```
SENTINEL: If Presten selects Option B, FORGE requires:
- [ ] ELO written confirmation of Option B scope (specific tables, reset logic)
- [ ] SENTINEL authorization for any schema changes beyond ecnl_verified
- [ ] Revised June 2 execution sequence from ELO
```

## Definition of Done

- Sections 1–4 populated from existing vault documents (no new decisions required from FORGE)
- Section 5 SENTINEL authorization block formatted and blank (ready for SENTINEL to fill on Option B decision)
- Filed before April 29 EOD so FORGE can respond immediately if Presten selects Option B on April 30

## Why This Matters

Option A is recommended and likely. But if Presten selects Option B at the April 30 decision point, FORGE has 33 days to June 2. Without a pre-assessed pipeline impact brief, the first hour of the April 30 session burns on diagnosis. This task converts that diagnosis into a ready-state document.

## References

- `10 - Agents/ELO/Tasks/task-2026-04-28-ecnl-migration-option-recommendation.md` — ELO's option analysis
- `10 - Agents/FORGE/Tasks/task-2026-04-28-ecnl-migration-pipeline-impact-spec.md` — existing FORGE pipeline impact spec
- `10 - Agents/ELO/Tasks/task-2026-04-28-ecnl-option-a-implementation-prep-package.md` — Option A reference
- `10 - Agents/ELO/Tasks/task-2026-04-28-ecnl-migration-elo-forge-handoff-checklist.md` — handoff timeline
