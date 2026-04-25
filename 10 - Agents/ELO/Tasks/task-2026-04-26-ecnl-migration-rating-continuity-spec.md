---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-05-10
deliverable: "02 - Tiger Tournaments/Projects/Rankings/ECNL Migration Rating Continuity Spec — June 2026.md"
---

# Task: ECNL Migration Rating Continuity Spec — June 2026

## Context

June 1 is the ECNL migration date. The existing ECNL Pre-Flight Checklist (Infrastructure) covers data ingestion (source switching, org_id updates, game deduplication). It does not cover what happens to team ELO ratings after migration.

The problem: ECNL teams will shift to ECNL RL as their primary competition tier on June 1. In the Evo Draw ranking engine:
- Historical ECNL games contribute to current ratings (weighted by recency).
- Event tier affects calibration — games in `ecnl` tier have different cal values than `ecnl_rl` tier.
- After migration, those historical `ecnl` games still exist in the DB, still tagged `event_tier = 'ecnl'`.

Questions ELO must answer:
1. Should historical `ecnl` games for migrating teams be re-tagged to `ecnl_rl`? Or kept as `ecnl`?
2. If re-tagged: rating recompute required. What is the rating shift magnitude for teams that migrate?
3. If NOT re-tagged: the `ecnl` cal value continues contributing to ratings even after the league dissolves. Is this correct behavior?
4. Do teams that were in `ecnl` and teams that were always in `ecnl_rl` converge to a common rating distribution after June 1? Or will there be a visible seam for the first 6–12 months post-migration?
5. What SQL is needed to handle the transition, and when must it run (before June 1, on June 1, or after)?

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/ECNL Migration Rating Continuity Spec — June 2026.md`

### Required Sections

#### Section 1: Taxonomy Clarification
Define the two paths post-migration:
- Path A: Teams that were ECNL and move to ECNL RL (historical games stay tagged `ecnl`, new games tagged `ecnl_rl`)
- Path B: Teams that were always ECNL RL (no change)
- Path C: Teams that were ECNL and do NOT continue (exit the ranked pool or drop to lower tier)

ELO should specify how to identify which teams fall into each path. Does the DB have a flag for this? Is it determinable from game history?

#### Section 2: Rating Impact Analysis

For Path A teams: model the expected rating impact of the tier change. Specifically:
- What is the cal value difference between `ecnl` and `ecnl_rl`?
- If historical `ecnl` games are re-tagged to `ecnl_rl`, what is the approximate rating shift direction and magnitude for an average ECNL team? (Can estimate without running full recompute — use calibration value delta and average game count.)
- Is the shift large enough to be visible at DSS (May 16)? Note: DSS demo is 2 weeks before migration. But the Presten session to implement the migration logic may happen earlier.

#### Section 3: Decision and Recommendation

ELO should make a recommendation on the correct handling approach. Options:
- **Option 1:** Keep historical `ecnl` games tagged as `ecnl`. New post-migration games tagged `ecnl_rl`. Rating algorithm handles naturally.
- **Option 2:** Re-tag historical `ecnl` games for migrating teams to `ecnl_rl` on June 1. Run recompute.
- **Option 3:** Introduce a new `event_tier = 'ecnl_legacy'` for pre-migration games with a calibration value that represents the "average" of the two eras.

For each option: describe the rating continuity behavior, the implementation cost, and the risk.

#### Section 4: SQL Requirements

For whichever option ELO recommends, write the SQL needed:
- If Option 1: no SQL needed beyond confirming new games receive correct tier. Provide a verification query.
- If Option 2: write the UPDATE statement with appropriate WHERE clause; note the recompute trigger.
- If Option 3: write the DDL change and the migration query.

#### Section 5: Sequencing

When must this run relative to June 1 date? Before, on, or after? What is the SENTINEL authorization gate? Does this require a Presten sign-off session?

## Quality Standard

ELO should make a recommendation — not just enumerate options. SENTINEL expects a decision ELO stands behind, with clear reasoning. The spec is a living document: if ELO's recommendation changes after April 28 recompute data becomes available, revise it and note the revision in the next briefing.

Deadline: May 10 (well before June 1; June migration prep sessions begin after DSS demo).
