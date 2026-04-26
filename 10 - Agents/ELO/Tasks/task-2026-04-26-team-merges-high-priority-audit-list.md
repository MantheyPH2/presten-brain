---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: medium
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Team Merges — High-Priority Audit List.md"
---

# Task: Team Merges — High-Priority Audit List

## Context

SENTINEL's mandate includes auditing the 27,820 team_merges entries for accuracy. ELO filed a Team Merges Audit Classification Framework (April 26) and the Team Merges Presten Execution Package (April 26). The classification framework defines risk categories (cross-gender, cross-age, different club names, large rating delta). The execution package prepares Presten to run queries.

What does not exist: the actual list of high-priority merges that Presten should review. The classification framework tells ELO how to identify suspicious merges. This task applies that framework and produces the list.

27,820 entries cannot be reviewed individually. The high-priority list narrows the scope to the 30–50 entries most likely to be incorrect, giving Presten a manageable review session with maximum quality impact.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Team Merges — High-Priority Audit List.md`

---

### Section 1: Filter Methodology

Describe the SQL logic used to identify high-priority merge candidates. ELO applies the classification framework categories in order of risk:

**Tier A — Almost certainly wrong:**
- Merges where the two teams have different gender designations
- Merges where the two teams have age group designations that differ by 2+ years
- Merges where team names differ entirely (not just abbreviation/spacing variants)

**Tier B — Probably wrong or needs verification:**
- Merges where the two teams have age group designations that differ by exactly 1 year
- Merges where club names differ (not just formatting) but could be related clubs
- Merges where rating delta between merged teams was > 150 points at time of merge

**Tier C — Worth reviewing but may be correct:**
- Merges where both names appear to be the same club but the records have conflicting data (different states, different org-IDs on different platforms)

State the SQL or query approach used to generate each tier. This section is SENTINEL's audit trail for how the list was built.

---

### Section 2: High-Priority Audit List

The actual list. Format: one row per merge candidate, sorted by risk tier then by rating delta (highest delta first within each tier).

| Merge ID | Team A Name | Team B Name | Age Group A | Age Group B | Gender A | Gender B | Rating Delta | Tier | Risk Reason |
|----------|------------|------------|-------------|-------------|----------|----------|-------------|------|-------------|
| | | | | | | | | A | Cross-gender merge |
| | | | | | | | | A | 3-year age gap |
| ... | | | | | | | | | |

Target: 30–50 rows. If the query returns more than 50 Tier A entries, cap at 50 (highest rating delta first) and note the total Tier A count so SENTINEL knows the fuller scope.

---

### Section 3: Presten Review Package

Each row in Section 2 should be verifiable by Presten without a coding session. For each merge, state what Presten needs to check:

Standard review question: "Are [Team A Name] and [Team B Name] the same team?"

For Tier A entries: "These teams have [cross-gender / large age gap / completely different names]. Unless there is an unusual explanation, this merge should be REVERSED."

Instructions for Presten:
- For each row: mark CONFIRM (merge is correct) / REVERSE (merge is incorrect) / INVESTIGATE (unclear, needs more data)
- REVERSE decisions: ELO de-merges and recomputes ratings for affected teams
- INVESTIGATE decisions: ELO files a Queue item with the specific ambiguity

ELO includes the SQL to reverse a merge (for each row marked REVERSE, Presten or ELO runs the unmerge query). The unmerge SQL should be in the Team Merges Presten Execution Package — reference it here rather than duplicating.

---

### Section 4: Expected Impact Assessment

Before Presten reviews, ELO estimates the rating impact of reversing the Tier A entries:

- Estimated number of teams whose ratings would change if all Tier A merges are reversed
- Direction of impact: does reversing cross-gender merges tend to raise or lower team ratings? (Depends on which direction the merge ran — ELO states which way)
- Age groups most affected by the high-priority list (age groups where Tier A concentrations are highest)

This section helps SENTINEL frame the review for Presten: "Reversing the 12 Tier A entries would affect approximately [N] team ratings, primarily in [age groups]."

---

### Section 5: Audit Status Tracker

A fill-in table for tracking review progress:

| Tier | Total Entries | Confirmed Correct | Reversed | Under Investigation | Not Yet Reviewed |
|------|--------------|------------------|---------|---------------------|-----------------|
| A | | | | | |
| B | | | | | |
| C | | | | | |
| **Total** | | | | | |

ELO updates this table in briefings as Presten reviews entries. SENTINEL monitors until "Not Yet Reviewed" reaches 0 for Tier A.

---

## Definition of Done

- File written at the deliverable path
- Filter methodology documents the SQL/query approach for all 3 tiers
- Section 2 has 30–50 rows, sorted by tier and rating delta
- Every row has all fields populated (no blank Tier or Risk Reason)
- Section 3 includes standard review language for Presten
- Expected impact assessment gives specific estimates, not ranges wider than ±50 teams
- Audit Status Tracker is pre-populated with totals per tier
- Summary in briefing: "Team Merges High-Priority Audit List filed. [N] Tier A entries (almost certainly wrong), [N] Tier B (verify), [N] Tier C (low priority). Reversing all Tier A would affect approximately [N] team ratings. List ready for Presten review session."

## Why This Matters

27,820 merges is an unauditable number without filtering. The classification framework (filed April 26) provides the theory — this task provides the list. Once Presten has this list, a 45-minute review session can produce CONFIRM/REVERSE decisions on the highest-risk entries, and ELO can recompute ratings for the reversed merges before DSS. An incorrect merge between two different-gender teams produces nonsensical ratings that would be visible in any DSS demo. Clearing the highest-risk merges before May 16 is a concrete quality improvement that SENTINEL can verify by checking the audit status tracker.

## References

- `task-2026-04-26-team-merges-audit-classification-framework.md` — classification framework this list applies
- `task-2026-04-26-team-merges-presten-execution-package.md` — unmerge SQL and query execution context
- `task-2026-04-23-team-merges-verification-campaign.md` — original verification campaign that established the 27,820 number
- [[Dedup Strategy]] — overall deduplication context and the 27,820 team_merges scope
