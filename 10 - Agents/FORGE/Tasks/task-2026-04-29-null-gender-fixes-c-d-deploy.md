---
type: agent-task
assigned_by: SENTINEL
assigned_to: FORGE
date: 2026-04-29
priority: high
due: "2026-04-30 session open"
status: pending
topic: null-gender-fixes-c-d-deploy
---

# Task: Deploy Null Gender Fixes C and D — April 30 Session Open

## Objective

Deploy null gender Fixes C and D at the very start of the April 30 Presten session. These fixes require no query results and no SENTINEL authorization — FORGE can deploy them immediately. Do not defer.

## Background

From `Infrastructure/Null Gender Audit — Code Implementation Specs.md`:

- **Fix C** (team name pattern expansion): Add B14, G14, 14B, 14G formats to the team name pattern array. No pre-condition.
- **Fix D** (event name pattern expansion): Add ECNL Girls, NPL Boys, GA Girls, and compound phrase patterns to the event name pattern array. No pre-condition.

Together these are estimated to resolve **800–2,300 null gender records** immediately. They have been staged since April 29 and have been ready to deploy since the code specs were filed.

Fixes A and B remain blocked on Presten running investigation queries (Query 1D for Fix A, Query 1C for Fix B). This task covers only C and D.

## Execution Steps

1. Open `Null Gender Audit — Code Implementation Specs.md` and copy the exact pattern additions for Fix C and Fix D.
2. Apply Fix C to the codebase (team name pattern array).
3. Apply Fix D to the codebase (event name pattern array).
4. Run the null gender count query before and after to confirm reduction:
   - Pre-deploy null count: `SELECT COUNT(*) FROM teams WHERE gender IS NULL;`
   - Post-deploy null count: same query
5. Report in next briefing: pre-count, post-count, delta, and any unexpected results.

## Deliverable

- Fixes C and D deployed to production.
- Before/after null count reported in April 30 briefing.
- No new null gender tasks required for C and D.

## Notes

- If deploy fails for any reason (syntax error, unexpected behavior), FORGE stops and reports to SENTINEL immediately — do not attempt workarounds.
- Fix A and Fix B remain pending Presten running investigation queries. This task is independent.
- If Presten provides Query 1C and 1D results during the same session, FORGE can proceed to Fixes A and B in the same session per the deployment order table in the code specs doc.
