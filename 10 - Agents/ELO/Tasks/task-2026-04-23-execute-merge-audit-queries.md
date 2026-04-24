---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-23
priority: urgent
deadline: 2026-05-10
status: pending
topic: execute-merge-audit-queries
---

# Task: Execute Merge Audit DB Queries and Flag Merges for Fix

You wrote two strong specs today — [[high-risk-merges-2026-04-23]] and [[unverified-merges-analysis-2026-04-23]] — but both are waiting on database execution. The analysis is complete. The data is not. Ship the data.

## What to do

### Part A — High-Risk Merges (top 50)

1. Run the top-50 high-risk merges query against the live `youth_soccer` DB.
2. Populate the review table in the spec with real data.
3. Apply the risk flags (`fuzzy_age_mismatch`, `large_rating_gap`, `unknown_method`, `coach_name_date_gap`, `geographic_mismatch`).
4. Identify which merges exceed the 75-point rating shift threshold (immediate fix needed).
5. Generate the un-merge SQL for the top candidates.
6. Write populated results to: `02 - Tiger Tournaments/Projects/Rankings/high-risk-merges-results-2026-04-23.md`

### Part B — Unverified Merges (top 2,000)

1. Run the top-2,000 unverified merges query.
2. Apply the auto-verify criteria: exact_name_match + same age_group + overlapping date ranges + GotSport source. Target: ~420 auto-verified records.
3. Flag the ~150-250 high-risk records.
4. Produce two outputs:
   - SQL batch to auto-verify the ~420 safe records
   - List of high-risk records for Presten manual review
5. Write to: `02 - Tiger Tournaments/Projects/Rankings/unverified-merges-results-2026-04-23.md`

## Acceptance criteria

- [ ] Top-50 high-risk merge table populated with real DB data
- [ ] Risk flags applied, un-merge SQL written for top candidates
- [ ] Auto-verify batch SQL written for ~420 records
- [ ] High-risk list ready for Presten review
- [ ] Both results files written

## Why this matters

The unverified-merges analysis estimates 8,000–20,000 games may be corrupted by bad merges. The Brier score analysis from [[Calibration Validation 2026-04-22]] estimates fixing top-10 high-risk merges alone could improve U13/U14 accuracy by 0.005–0.015. This must be done before the May 18–20 recalculation. Deadline: **May 10** to leave time for Presten review.
