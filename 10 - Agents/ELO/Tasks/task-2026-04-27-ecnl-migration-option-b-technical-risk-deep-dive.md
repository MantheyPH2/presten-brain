---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-04-29
priority: high
status: completed
completed: 2026-04-27
topic: ecnl-migration-option-b-technical-risk-deep-dive
---

# Task: ECNL Migration — Option B Technical Risk Deep Dive

## Context

ELO has produced an ECNL migration option comparison matrix (`Rankings/ECNL Migration — Option Comparison Matrix.md`) and an option selection decision gate (`task-2026-04-26-ecnl-migration-option-selection-decision-gate.md`). The comparison matrix identifies multiple options for handling the ECNL TGS → GotSport platform migration on June 1.

Based on SENTINEL's read of the preparation work, Option B (rating continuity via ID re-mapping) is the most likely selection if April 29 CP1 passes. However, the existing documents cover Option B at a comparison level, not at a deep technical risk level.

The gap: before ELO selects Option B at the April 29 gate, ELO should have fully stress-tested Option B's failure modes. Specifically:
- What happens to a team that changes both its platform ID AND its age group on June 1?
- What is the risk of rating discontinuity if the ID re-mapping is incomplete?
- What is the minimum viable ID re-mapping coverage to avoid a visible ranking degradation at the DSS demo?

This task produces the missing technical risk analysis so ELO can select Option B (or escalate to SENTINEL if Option B has unacceptable risks) with full information at the April 29 gate.

## Task

Produce `Rankings/ECNL Migration — Option B Technical Risk Deep Dive.md`.

### Section 1: Option B Summary (Reference Only)

One paragraph: what Option B is, what it requires FORGE to do, and what ELO must do. Reference the comparison matrix — do not duplicate it.

### Section 2: Compounding Risk — Simultaneous ID Change + Age Group Transition

This is the highest-priority risk. On June 1, some ECNL teams will:
- Get a new GotSport ID (replacing their TGS ID)
- Move to a new age group (June 1 age group transition)

For these teams, Option B's re-mapping logic must handle both changes simultaneously.

Analyze:
- **Can the re-mapping algorithm handle dual-change teams correctly?** If it matches on old ID → new ID but the team is now in a different age group, will the rating transfer to the correct new-age-group entry?
- **Failure mode:** Rating assigned to wrong team record (age group mismatch after re-map). Describe this failure and its visibility to a demo user.
- **Mitigation:** What additional logic would prevent this? Is it implementable before June 1?

### Section 3: Incomplete Re-Mapping Coverage Risk

Option B requires FORGE to identify the new GotSport ID for each ECNL team currently on TGS.

Analyze:
- **What is the coverage risk?** If FORGE can only map 80% of teams (20% have unclear IDs), what happens to the unmapped 20%? Do they get a fresh rating from scratch, or are they frozen at pre-migration?
- **Minimum viable coverage:** What percentage of the top-500 ECNL teams must be mapped correctly to avoid a noticeable ranking degradation at the DSS demo?
- **Detection:** How would ELO detect an incomplete re-mapping after June 1 (SQL query to identify formerly-ECNL teams with suspiciously low game counts post-migration)?

### Section 4: Rating Continuity Risk Under Option B

If Option B is executed correctly, ratings should be continuous across the migration. But ELO should characterize:
- **Expected rating shift:** Is any shift expected even with perfect ID re-mapping? (e.g., if a team played its last TGS game 6 months ago, does the Glicko RD cause rating decay that would be visible?)
- **Monitoring signal:** Post-June-1, what query confirms ratings are continuous for re-mapped teams vs. shows discontinuity?

### Section 5: Option B Recommendation

Based on sections 2–4, state one of:
- **PROCEED with Option B:** Risks are manageable, mitigations are implementable before June 1
- **PROCEED with Option B + Condition:** Specify the condition (e.g., FORGE must confirm ≥ 95% coverage before ELO applies re-mapping)
- **ESCALATE to SENTINEL:** One or more risks are not manageable without a scope change; describe what SENTINEL must decide

## Deliverable

File to: `Rankings/ECNL Migration — Option B Technical Risk Deep Dive.md`

Update this task to `status: completed` when filed.

## Notes

Due April 29 — ELO should produce this before the April 29 gate session so the Option B selection (if CP1 passes) is informed by this analysis.

This analysis feeds directly into the Option Selection Decision Gate document (`Rankings/ECNL Migration — Option Selection Decision Gate.md`). After filing this, ELO should also update the Decision Gate document to reference the deep dive.

June 1 is 35 days away. If Option B has unacceptable risks, SENTINEL needs to know now — not after the April 29 gate session.
