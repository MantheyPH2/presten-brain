---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-04-29
priority: medium
status: completed
topic: post-fix-rating-distribution-analysis-spec
---

# Task: Post-Fix Rating Distribution Analysis Spec

## Context

The pre-run model (filed April 27) predicts Girls GA ASPIRE teams shift −30–50 pts after the April 28 recompute. This is ELO's prediction going in. What is missing is a structured analysis spec: how ELO will actually measure whether the distribution shift was correct, what queries to run, and what conclusions to draw. Without this, April 29 produces narrative observations instead of a systematic calibration check.

## Deliverable

File `Rankings/Post-Fix-Rating-Distribution-Analysis-Spec-2026-04-27.md` in `02 - Tiger Tournaments/Projects/` with:

1. **Pre-fix baseline query:** SQL to capture average rating by age group for ASPIRE-heavy teams (defined as teams with ≥3 ASPIRE events in their game history). To be run from the pre-fix snapshot Presten took April 27.

2. **Post-fix comparison query:** Same query, run after April 28 recompute. Delta = post − pre per age group.

3. **Expected delta table:** Based on the pre-run model, fill in expected deltas:
   | Age Group | Expected Delta | Confidence |
   |-----------|---------------|------------|
   | U13G | −30 to −50 | Medium |
   | U14G | ... | ... |
   | U15G | ... | ... |
   | U16G | ... | ... |
   | U17G | ... | ... |

4. **Interpretation thresholds:**
   - Within expected range → calibration confirmed, proceed
   - Larger than expected (>−50) → flag to SENTINEL, hold Rank Bands session
   - Smaller than expected (<−10) → investigate whether April 28 fix actually applied
   - Positive shift (rating went UP) → critical flag, escalate immediately

5. **Boys check:** Did Boys ratings shift at all? Expected: 0 ± 5 pts. Any shift > 10 pts on Boys side is unexpected and must be explained before April 29 gate checks proceed.

6. **Non-ASPIRE teams sanity check:** Pick 3 well-known stable teams. Confirm their ratings did not shift more than 5 pts. If they shifted more, the recompute may have had unintended scope.

## Success Criteria

- On April 29, ELO fills in the "actual" column and can immediately classify each age group as confirmed/flagged/critical
- This converts April 29 calibration work from exploration to criteria-check
- Takes < 20 minutes to execute on April 29

## Notes

- The pre-fix baseline query must be designed to run against the snapshot, not the live database — confirm with FORGE which table or export file holds the baseline
- If Presten did not take a pre-fix snapshot before April 28, note this and propose an alternative comparison method
