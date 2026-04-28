---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: open
priority: high
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Rankings/GA ASPIRE Fix — Post-April-28 Verification Query Results.md"
---

# Task: GA ASPIRE Fix — Post-April-28 Verification Query Package

## Context

The April 28 session will execute the GA ASPIRE event_tier UPDATE (Step 1 of the Execution Log). The G0 gate (ELO's April 29 G0 Gate Confirmation document) confirms the session completed without errors. But G0 is a pass/fail binary. What it does NOT confirm: whether the ASPIRE event_tier values are now correct, how many teams' ratings will shift as a result, and whether any non-ASPIRE events were accidentally reclassified.

ELO needs a forensic verification layer beyond G0. This task produces the specific SQL queries to run this forensic check, plus a results template to fill in and file during the April 29 session — before ELO proceeds to the G1–G4 gate sequence.

The queries must be ready NOW so that the April 29 session begins immediately with verified data rather than spending 20 minutes writing queries on the fly.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/GA ASPIRE Fix — Post-April-28 Verification Query Results.md`

File by April 29 (template written today; results filled after April 29 session).

---

### Section 1: Verification Query Set

**Query V1 — Event Tier Distribution Before vs. After**
```sql
-- Expected: events formerly miscategorized as 'aspire' now show correct tier
SELECT event_tier, COUNT(*) as event_count
FROM events
WHERE state = 'GA'
GROUP BY event_tier
ORDER BY event_count DESC;
```
Expected result: `aspire` tier should contain only events matching ASPIRE criteria. Document the expected `aspire` count based on pre-run knowledge.

**Query V2 — Teams Affected by ASPIRE Fix**
```sql
-- Count teams that had at least one game against an event whose tier changed
SELECT COUNT(DISTINCT team_id) as teams_affected
FROM games g
JOIN events e ON g.event_id = e.id
WHERE e.state = 'GA'
  AND e.event_tier = 'aspire'
  AND g.played_at >= NOW() - INTERVAL '2 years';
```
Expected range: document the expected count from pre-run analysis. If actual > 2× expected, flag.

**Query V3 — Rating Shift Magnitude (GA Girls, Post-Recompute)**
```sql
-- Compare ratings before and after for GA girls teams
-- Requires pre-run snapshot from task-2026-04-24-pre-april28-baseline-snapshot.md
SELECT
  t.team_name,
  t.age_group,
  r_before.rating as rating_before,
  r_after.rating as rating_after,
  (r_after.rating - r_before.rating) as delta
FROM teams t
JOIN ratings r_after ON t.id = r_after.team_id
JOIN ratings_snapshot_pre_april28 r_before ON t.id = r_before.team_id
WHERE t.state = 'GA'
  AND t.gender = 'girls'
ORDER BY ABS(r_after.rating - r_before.rating) DESC
LIMIT 20;
```
Note: Adjust table/column names to match actual schema.

**Query V4 — Reference Club Spot Check**
```sql
-- Confirm 3 known GA ASPIRE clubs now have ASPIRE tier correctly assigned
-- ELO fills in known reference club names from the team merges audit
SELECT t.team_name, e.event_name, e.event_tier
FROM teams t
JOIN games g ON t.id = g.team_id
JOIN events e ON g.event_id = e.id
WHERE t.team_name IN ('[ASPIRE Club 1]', '[ASPIRE Club 2]', '[ASPIRE Club 3]')
  AND e.state = 'GA'
  AND e.event_tier = 'aspire'
LIMIT 10;
```
ELO fills in reference club names from prior calibration work.

**Query V5 — Collateral Damage Check (Non-ASPIRE GA Events)**
```sql
-- Confirm no non-ASPIRE GA events were accidentally reclassified
SELECT event_name, event_tier, COUNT(DISTINCT team_id) as teams
FROM events e
JOIN games g ON e.id = g.event_id
WHERE e.state = 'GA'
  AND e.event_tier != 'aspire'
  AND e.updated_at >= '[session timestamp - 1 hour]'
GROUP BY event_name, event_tier
ORDER BY teams DESC;
```
Expected result: zero or few rows. Any rows indicate potential collateral reclassification — ELO investigates each one.

---

### Section 2: Results Template (Fill After April 29 Session)

| Query | Expected | Actual | Status |
|-------|----------|--------|--------|
| V1: ASPIRE event count | _[fill pre-run]_ | _[fill post-run]_ | PASS / FAIL |
| V2: Teams affected | _[fill pre-run]_ | _[fill post-run]_ | PASS / FAIL |
| V3: Largest rating shift (top mover) | < 80 pts expected | _[fill]_ | PASS / FAIL |
| V4: Reference clubs confirmed ASPIRE | All 3 show aspire tier | _[fill]_ | PASS / FAIL |
| V5: Collateral reclassifications | 0 | _[fill]_ | PASS / FAIL |

**Overall ASPIRE Fix Verdict:** CONFIRMED CLEAN / ANOMALY DETECTED

If ANOMALY DETECTED: ELO files SENTINEL queue item (priority: high) before proceeding to G1–G4.

---

### Section 3: Rating Impact Summary

Based on Query V3 results:

| Metric | Value |
|--------|-------|
| GA girls teams with rating change > 10 pts | _[fill]_ |
| GA girls teams with rating change > 30 pts | _[fill]_ |
| Direction of changes (mostly UP / mostly DOWN / mixed) | _[fill]_ |
| Largest individual shift | _[fill team name + delta]_ |
| Is the direction consistent with ASPIRE fix intent? | YES / NO |

If NO: ELO documents why and whether this is expected or anomalous.

---

## Definition of Done

- Query set (Section 1) filed in the document by end of April 28
- Results template (Section 2) pre-populated with expected values by end of April 28
- Results filled in during April 29 session, before G1–G4 gates begin
- If any query returns FAIL: SENTINEL queue alert filed before ELO proceeds
- ELO briefing April 29 references this document and states overall verdict

## Why This Matters

The G0 gate confirms the session ran without crashing. It does not confirm the data is correct. The GA ASPIRE fix touches event tier assignments for a significant number of GA girls games — the wrong `event_tier` value has been in production for an unknown period. ELO must confirm the fix did exactly what was intended and nothing else before trusting the post-recompute ratings in G1–G4 analysis.

## References

- `task-2026-04-24-pre-april28-baseline-snapshot.md` — snapshot to compare against
- `task-2026-04-23-ga-aspire-tier-fix.md` — original fix spec
- `task-2026-04-24-ga-aspire-update-sql.md` — the UPDATE SQL ELO wrote
- `task-2026-04-25-april29-gate-results-filing-document.md` — G1–G4 gate results document
- `task-2026-04-27-april28-signal-monitoring-checklist.md` — signal monitoring context
