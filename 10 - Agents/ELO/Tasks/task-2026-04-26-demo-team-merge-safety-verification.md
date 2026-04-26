---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-26
priority: medium
due: 2026-05-07
deliverable: "02 - Tiger Tournaments/Projects/Rankings/DSS Demo Teams — Merge Safety Verification.md"
---

# Task: DSS Demo Teams — Merge Safety Verification

## Context

ELO's Team Merges High-Priority Audit List (filed April 26) identifies 30–50 highest-risk merge candidates using three Tier A filters (cross-gender, age gap ≥ 2 years, cross-club). Section 4 of that document notes:

> "Demo team names (Football Academy NJ, PA Classics, PDA, NJ Wildcats, FC Stars) should be cross-checked against the returned rows before filing with Presten."

This cross-check has not been done. The DSS demo on May 16–18 features these teams as ranking examples. If any demo team is involved in a suspicious merge — one that crosses genders, crosses clubs, or spans a large age gap — the merge could be artificially inflating or deflating that team's ranking. Demonstrating a team with a bad merge is the worst possible outcome for a data quality pitch.

This document closes that gap. It is a narrow, pre-demo safety check, not a general audit. The general audit is addressed by the High-Priority Audit List task.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/DSS Demo Teams — Merge Safety Verification.md`

---

### Section 1: Demo Teams Under Review

| Team Name | Age Group(s) | Gender | Primary Demo Feature |
|-----------|-------------|--------|---------------------|
| Football Academy NJ | [ELO fills from previous research] | | Rank Bands example |
| PA Classics | [ELO fills] | | Club Rankings |
| PDA | [ELO fills] | | Club Rankings |
| NJ Wildcats | [ELO fills] | | [ELO fills] |
| FC Stars | [ELO fills] | | [ELO fills] |

Fill in known information from prior briefings and research. If an age group is unknown, note "all age groups — check all."

---

### Section 2: Merge Check Queries

ELO writes three queries. These are designed to run on the same session where Presten executes the High-Priority Audit List Tier A queries. They run against the same `team_merges` table.

**Query A — Are any demo teams the primary team in a merge?**

```sql
-- ELO writes the actual query
-- Should return: merge_id, primary_team_id, team_name, merged_team_id, merge_type (if any classification exists)
-- Filter: WHERE t.name ILIKE ANY (ARRAY['%Football Academy NJ%', '%PA Classics%', '%PDA%', '%NJ Wildcats%', '%FC Stars%'])
-- And t.id = primary_team_id
```

**Query B — Are any demo teams the secondary (absorbed) team in a merge?**

```sql
-- Same structure but t.id = merged_team_id
-- These are teams that were merged INTO another team — may not appear in rankings under their known name
```

**Query C — For any merge found in A or B: classify it**

For each merge returned by A or B:
```sql
-- Check gender match: t1.gender = t2.gender?
-- Check club match: t1.club_id = t2.club_id?
-- Check age group proximity: |t1.age_group - t2.age_group| <= 1?
```

A merge is SAFE if: same gender, same club, adjacent age group (≤ 1 year gap).
A merge is SUSPICIOUS if: different gender OR different club OR age gap ≥ 2 years.

---

### Section 3: Verification Results

| Team Name | Found as Primary? | Found as Secondary? | Merge Count | Merge Classification | SAFE / SUSPICIOUS |
|-----------|------------------|--------------------|-----------|--------------------|-------------------|
| Football Academy NJ | | | | | |
| PA Classics | | | | | |
| PDA | | | | | |
| NJ Wildcats | | | | | |
| FC Stars | | | | | |

This section is empty at document creation — ELO populates it when Presten runs the queries (April 29 session or first available psql session).

---

### Section 4: Risk Assessment

| Result | Implication | Action |
|--------|-------------|--------|
| No demo team appears in any merge | All clear. Proceed with demo. | None. |
| Demo team in SAFE merge(s) only | Low risk. Normal same-club age-proximity merge. | Note in SENTINEL briefing; no demo change. |
| Demo team in SUSPICIOUS merge | Rating may be distorted. | SENTINEL flags to Presten. Consider using alternate demo team for that feature. Alternate candidates: [ELO proposes 2 backup teams per feature]. |
| Demo team absorbed (secondary) and no longer ranking | Team may not appear in current rankings. | SENTINEL flags. Use primary team name instead, or pick alternate demo team. |

---

### Section 5: Alternate Demo Teams

For each primary demo team, propose 2 backup teams in case a suspicious merge disqualifies the primary. Backup teams should:
- Serve the same demo purpose (e.g., Club Rankings backup = another top Girls club in the expected range)
- Not appear in the suspicious merge list
- Have enough games for stable rankings

ELO proposes backups based on current ranking knowledge. These do not need merge verification — they are contingency candidates that SENTINEL holds in reserve.

| Primary | Demo Feature | Backup 1 | Backup 2 |
|---------|-------------|----------|----------|
| Football Academy NJ | Rank Bands | | |
| PA Classics | Club Rankings | | |
| PDA | Club Rankings | | |
| NJ Wildcats | [feature] | | |
| FC Stars | [feature] | | |

---

## Definition of Done

- Document written at `Rankings/DSS Demo Teams — Merge Safety Verification.md`
- All 5 sections present
- Three queries written and copy-paste ready
- Section 2 Table populated when Presten runs the queries (flag in next briefing when queries run: "Demo team merge check complete — results in DSS Demo Teams Merge Safety doc")
- Alternate demo teams proposed for all 5 primary teams
- Risk assessment table is present at document creation (even if Section 3 is empty)

## Why This Matters

The DSS demo is the highest-stakes output of all current Evo Draw work. A rank or club ranking that is distorted by a bad merge — demonstrated in front of potential customers — would undermine the credibility of the entire system. This check takes 30 minutes of Presten psql time and 2 hours of ELO analysis. The cost of not doing it is: discovering a bad merge during the May 9 readiness check, 7 days before the demo, when there is no time to reverify and the alternate team hasn't been prepared. Running this check now means May 9 is a confirmation, not a discovery.

## References

- `Rankings/Team Merges — High-Priority Audit List.md` — Tier A queries (Section 4 note about demo teams)
- `Rankings/Team Merges — Presten Execution Package.md` — psql session context
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — demo feature map
- `Rankings/Rank Bands — Demo Examples.md` — Football Academy NJ context
- `Rankings/Club Rankings — Reference Clubs.md` — PA Classics, PDA context
