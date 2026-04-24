---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-23
priority: urgent
deadline: 2026-05-14
status: pending-execution
topic: ga-aspire-tier-fix
deliverables_written: "02 - Tiger Tournaments/Projects/Rankings/ga-coverage-audit-results-2026-04-23.md"
blocker: "Requires psql execution to run audit queries and apply ASPIRE UPDATE — classifier SQL and staged UPDATE are ready in deliverable doc"
---

# Task: Execute GA Audit and Apply ASPIRE Tier Fix

Your GA coverage audit spec ([[ga-data-coverage-audit-2026-04-23]]) identified a critical calibration error: GA ASPIRE events (launched Feb 2025) are currently tagged as `ga` tier (cal=140) instead of `ga_aspire` (cal=100). This is overcalibrating 350–1,050 teams by ~40 points. Fix it before DSS.

## What to do

### Step 1 — Run the 6 audit SQL queries

Execute the queries from the spec against `youth_soccer` DB:
- Total GA games by source
- Age group distribution
- Season coverage
- ASPIRE game identification (events with "ASPIRE" in name since Feb 2025)
- Teams affected by ASPIRE miscalibration
- Boys GA sample size

Document actual query results (not expected ranges) in: `02 - Tiger Tournaments/Projects/Rankings/ga-coverage-audit-results-2026-04-23.md`

### Step 2 — Build ASPIRE event classifier

Write the classifier function/query that identifies GA ASPIRE events:
- Events with "ASPIRE" in event name
- Event start date ≥ 2025-02-01
- Currently classified as `ga` tier

Produce: a SQL UPDATE statement that re-tags these events from `ga` → `ga_aspire` in the `event_tiers` table.

### Step 3 — Validate before applying

Before running the UPDATE:
- Count how many events will change
- Count how many teams are affected
- Estimate the average rating shift (should be ~−40 points for affected teams)
- Write validation summary

### Step 4 — Apply the fix

Run the UPDATE. Re-run `compute-rankings.js` for the affected age groups. Document before/after rating deltas for a sample of 10 affected teams.

### Step 5 — Boys GA calibration flag

Confirm whether the Boys GA win-rate cross-league study has been run. If not, flag it for Presten's attention — the current Boys GA cal value is empirically unvalidated.

## Acceptance criteria

- [ ] Actual audit query results documented (not expected ranges)
- [ ] ASPIRE classifier produces a valid SQL UPDATE
- [ ] Validation summary written before fix applied
- [ ] Fix applied and before/after deltas documented for 10 teams
- [ ] Boys GA calibration status flagged
- [ ] All output written to `ga-coverage-audit-results-2026-04-23.md`

## Why this matters

350–1,050 teams are currently ranked ~40 points too high because ASPIRE events are incorrectly classified. DSS division placements on May 16 are based on these rankings. This must be fixed in the recalculation before rankings are used for seeding. Deadline: **May 14** (two days before DSS).
