---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Reports/gotsport-org-expansion-stage1-results-[date].md"
---

# Task: Stage 1 Backfill Results Report

## Context

The GotSport org-ID expansion is a 3-stage process:

- **Stage 1:** TX org_id pilot crawl — run against Texas org IDs as a test cohort. Presten executes.
- **Stage 2:** Expand to remaining U.S. org IDs using the full list from `Infrastructure/full-gotsport-org-id-list.md`.
- **Stage 3:** International org IDs (out of scope until Stage 2 is stable).

Stage 1 execution is pending Presten running the TX test crawl (per `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` Section 2). Stage 2 cannot begin until Stage 1 exit conditions are verified and documented.

This task: when Presten runs Stage 1, FORGE writes the results report. The report is the gate that authorizes Stage 2.

## What to Do

### Step 1: Confirm Stage 1 Has Executed

Check for the following signals that Presten has run Stage 1:
- New games appear in the database from TX org_ids (check `Reports/` for any session notes Presten may have dropped)
- The Pipeline Deployment Validation Spec Section 2 pass/fail has been evaluated

If Stage 1 has NOT executed yet: this task remains pending. Do not write a placeholder report. Re-check at the next session.

### Step 2: Compile Stage 1 Results

Once execution is confirmed, write `02 - Tiger Tournaments/Projects/Reports/gotsport-org-expansion-stage1-results-[date].md` where `[date]` is the execution date.

The report must include:

**A. Game Ingestion Summary**
```
Total new games ingested in Stage 1 crawl: [N]
Org IDs in TX cohort: [N]
Games per org_id (range): [min] – [max]
Latest ingestion timestamp: [datetime]
```

**B. Duplicate Check Result**
Per the Validation Spec, this query was run:
```sql
SELECT event_id, home_team_id, away_team_id, game_date, COUNT(*) AS count
FROM games
WHERE created_at > NOW() - INTERVAL '24 hours'
GROUP BY event_id, home_team_id, away_team_id, game_date
HAVING COUNT(*) > 1;
```
Result: `[0 rows — pass] | [N duplicate rows — FAIL, see Section D]`

**C. Stage 1 Exit Condition Assessment**
Per the Stale Team Backfill Runbook: "Stage 1 is complete when game count for new org_ids stabilizes (no new games added on second crawl)."
- Was a second crawl run? [Yes / No]
- If yes: game count delta = [N]. If 0: exit condition met.
- If second crawl not yet run: note this and confirm it must run before Stage 2.

**D. Issues / Escalations**
Any org_ids that returned errors, produced duplicates, or had unexpected game counts. List with org_id and observation.

**E. Stage 2 Authorization**
State one of:
- "Stage 1 exit conditions met. Stage 2 is authorized. Recommend Presten proceed with full org-ID list per Infrastructure/full-gotsport-org-id-list.md."
- "Stage 1 exit conditions not met. [Reason]. Do not proceed to Stage 2 until [resolution]."

### Step 3: Update Source Gap Inventory

Update the Source Gap Inventory for any leagues whose org_ids were included in the TX cohort:
- Change `Last Known Ingestion` from "unknown" to the actual ingestion date.
- Change `Status` from Stale → Active if ingestion confirmed and game count > 0.

### Step 4: Brief SENTINEL

Include in your next briefing: "Stage 1 results report filed at `Reports/gotsport-org-expansion-stage1-results-[date].md`. New games: [N]. Duplicates: [0 or N]. Stage 2 authorized: [Yes/No]."

## Definition of Done

- Report written at `Reports/gotsport-org-expansion-stage1-results-[date].md`
- All five sections present
- Stage 2 authorization explicitly stated (Yes or No)
- Source Gap Inventory updated for affected leagues
- Summary in briefing

## Why This Matters

The org-ID expansion is the highest-leverage infrastructure change pending — the TX cohort alone could add thousands of previously uncaptured games. But Stage 2 must not run until Stage 1 is verified clean. This report is the handoff document from execution (Presten) to authorization (SENTINEL/FORGE) to proceed.

## References

- `02 - Tiger Tournaments/Projects/Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — Section 2: TX crawl validation criteria
- `02 - Tiger Tournaments/Projects/Infrastructure/Stale Team Backfill Runbook — April 2026.md` — Stage 1 exit conditions
- `02 - Tiger Tournaments/Projects/Infrastructure/full-gotsport-org-id-list.md` — Stage 2 source list
- `10 - Agents/FORGE/Tasks/task-2026-04-24-gotsport-org-id-expansion.md` — expansion design
