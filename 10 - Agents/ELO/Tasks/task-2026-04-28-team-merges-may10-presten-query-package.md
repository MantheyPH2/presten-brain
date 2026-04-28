---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: pending
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Team Merges Audit — Presten Execution Package.md"
---

# Task: Team Merges Audit — Presten Execution Package

## Context

The team merges audit (`task-2026-04-23-execute-merge-audit-queries.md`) has a May 10 deadline and SQL ready. The block: ELO cannot run psql directly. ELO has been waiting for a "psql session" as if this requires a joint working session, but it doesn't. What it requires is: a clearly packaged set of commands Presten can paste into psql in < 10 minutes, return the output to ELO, and ELO processes the results.

The gap is not a psql session. The gap is an execution package that requires no explanation and no ELO presence — just Presten running 3–5 commands and sharing the output.

This task produces that package. It is the difference between this audit happening before May 10 and it not happening.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Team Merges Audit — Presten Execution Package.md`

This document is addressed to Presten, not to FORGE or SENTINEL. Write it as if Presten has never seen the audit context. Clear, numbered, pasteable.

---

### Section 1: What This Is (For Presten)

> **You can run this in < 10 minutes. Paste each query into psql, copy the output, and share it with ELO.**
>
> Context: Evo Draw has 27,820 team merges. SENTINEL has flagged a subset that may be incorrect — teams merged that shouldn't be, or teams not merged that should be. This audit identifies the highest-risk merges for ELO to review. It does not change any data.
>
> **Deadline: May 10.** If queries aren't run by May 7, ELO cannot complete the review in time.

---

### Section 2: Pre-Run Checks (30 seconds)

Before pasting queries, confirm:

```sql
-- Check 1: How many team merges exist?
SELECT COUNT(*) FROM team_merges;
-- Expected: approximately 27,820. If < 20,000 or > 35,000, note the discrepancy.

-- Check 2: Confirm the column structure
SELECT column_name FROM information_schema.columns 
WHERE table_name = 'team_merges' 
ORDER BY ordinal_position;
-- Share the full output. ELO needs to confirm the column names match the queries below.
```

---

### Section 3: Audit Queries (Paste in Order)

**Query A — High-Volume Merges (Largest Merge Groups)**
```sql
-- Teams merged with > 3 other team IDs — highest dedup risk
SELECT 
  primary_team_id,
  COUNT(*) as merged_count,
  STRING_AGG(merged_team_id::text, ', ') as merged_ids
FROM team_merges
GROUP BY primary_team_id
HAVING COUNT(*) > 3
ORDER BY merged_count DESC
LIMIT 50;
```
Share full output.

**Query B — Cross-State Merges**
```sql
-- Merges where teams are from different states — likely errors
SELECT 
  tm.*,
  t1.state as primary_state,
  t2.state as merged_state
FROM team_merges tm
JOIN teams t1 ON tm.primary_team_id = t1.id
JOIN teams t2 ON tm.merged_team_id = t2.id
WHERE t1.state != t2.state
LIMIT 100;
```
Share full output.

**Query C — Cross-Gender Merges**
```sql
-- Merges where teams have different gender assignments — likely errors
SELECT 
  tm.*,
  t1.gender as primary_gender,
  t2.gender as merged_gender,
  t1.team_name as primary_name,
  t2.team_name as merged_name
FROM team_merges tm
JOIN teams t1 ON tm.primary_team_id = t1.id
JOIN teams t2 ON tm.merged_team_id = t2.id
WHERE t1.gender != t2.gender
  AND t1.gender IS NOT NULL
  AND t2.gender IS NOT NULL
LIMIT 100;
```
Share full output.

**Query D — Cross-Age-Group Merges (> 1 year gap)**
```sql
-- Merges where teams differ by more than 1 year in age group — likely errors
SELECT 
  tm.*,
  t1.age_group as primary_age,
  t2.age_group as merged_age,
  t1.team_name as primary_name,
  t2.team_name as merged_name
FROM team_merges tm
JOIN teams t1 ON tm.primary_team_id = t1.id
JOIN teams t2 ON tm.merged_team_id = t2.id
WHERE ABS(
  CAST(REGEXP_REPLACE(t1.age_group, '[^0-9]', '', 'g') AS INTEGER) -
  CAST(REGEXP_REPLACE(t2.age_group, '[^0-9]', '', 'g') AS INTEGER)
) > 1
LIMIT 100;
```
Note: If this query errors on the REGEXP_REPLACE, share the error and ELO will adjust.

**Query E — Recent Merges (Last 30 Days)**
```sql
-- Merges added recently — highest chance of being incorrect
SELECT 
  tm.*,
  t1.team_name as primary_name,
  t2.team_name as merged_name
FROM team_merges tm
JOIN teams t1 ON tm.primary_team_id = t1.id
JOIN teams t2 ON tm.merged_team_id = t2.id
WHERE tm.created_at >= NOW() - INTERVAL '30 days'
ORDER BY tm.created_at DESC
LIMIT 100;
```
Share full output.

---

### Section 4: What ELO Will Do With Results

ELO will classify each flagged merge as:
- **CONFIRMED CORRECT** — merge is accurate, no action
- **LIKELY ERROR** — merge appears incorrect, ELO flags for Presten decision
- **NEEDS INVESTIGATION** — unclear, ELO will request additional data

ELO will file the classified results at `Rankings/Team Merges Audit — Classified Results.md` within 48 hours of receiving query output.

---

### Section 5: Results Template (ELO Fills Post-Query)

ELO pre-stages the triage table here after Section 2–3 confirm column structure:

| Query | Total Results | Likely Errors | Confirmed Correct | Needs Investigation |
|-------|--------------|--------------|------------------|---------------------|
| A (High-volume) | | | | |
| B (Cross-state) | | | | |
| C (Cross-gender) | | | | |
| D (Cross-age) | | | | |
| E (Recent) | | | | |
| **TOTAL** | | | | |

---

## Definition of Done

- Document filed at deliverable path by April 30
- Section 2 pre-run checks finalized with correct column names (if ELO can verify from schema knowledge; otherwise flagged)
- Section 3 queries verified for correctness against known schema
- Section 5 results template pre-staged
- ELO's April 30 briefing includes: "Team merges execution package ready for Presten — awaiting query output. Deadline: May 7 to process before May 10."

## Why This Matters

27,820 team merges is the most significant unvalidated data quality risk in Evo Draw. SENTINEL's mandate includes auditing these. The May 10 deadline is real. The only reason this audit hasn't run is the absence of a self-contained execution package Presten can run without coordination overhead. This task eliminates that gap.

## References

- `task-2026-04-23-execute-merge-audit-queries.md` — original audit task (SQL may need updating)
- `task-2026-04-26-team-merges-audit-classification-framework.md` — triage framework ELO will use
- `task-2026-04-26-team-merges-high-priority-audit-list.md` — high-priority merges (input to query targeting)
- `task-2026-04-26-demo-team-merge-safety-verification.md` — DSS demo merge safety context
- `task-2026-04-23-team-merges-verification-campaign.md` — original campaign scope
