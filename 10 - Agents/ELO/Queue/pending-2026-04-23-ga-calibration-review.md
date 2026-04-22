---
type: agent-queue
agent: ELO
category: recommendation
priority: medium
date: 2026-04-23
status: pending
answer: ""
---

# GA Calibration Review Request

## Context

The GA data coverage audit (`task-2026-04-23-ga-data-coverage-audit`) is complete. The audit identified three calibration items requiring Presten's decision before the May 18-20 production recalculation window.

---

## Item 1: GA ASPIRE — New Tier Needed (Recommended Change)

**Issue:** GA ASPIRE (launched February 2025) is currently tagged as `ga` in the `event_tiers` table. ASPIRE is a developmental sub-tier — it is positioned below the main GA competition level. Applying the full GA calibration of 140 to ASPIRE teams overstates their competitive level by an estimated 30-50 points.

**Estimated affected teams:** 350-1,050 teams across 7 age groups.

**Recommendation:** 
1. Build an ASPIRE event name classifier (all events containing "aspire" currently tagged as `ga`)
2. Add `ga_aspire` tier to `event_tiers` with calibration value **100**
3. Apply in the May 18-20 recalculation window

**Why 100?** GA ASPIRE is analogous to ECNL RL relative to ECNL. However, given ASPIRE is described as a direct entry-level sub-tier of GA (rather than a completely separate league), the drop is expected to be smaller than the ECNL→ECNL RL gap (130→55). Setting ASPIRE at 100 is conservative and matches the current GA boys calibration. A future empirical validation can adjust if needed.

**Action needed from Presten:** Approve or modify the proposed ga_aspire calibration of 100.

---

## Item 2: GA Girls (140) — Validation Pending DB Query

**Issue:** The GA girls calibration (140) is empirically plausible based on the cross-league analysis framework, but the actual win rate query has not been executed against the database.

**Expected result when run:** GA girls win 55-58% against ECNL girls, implying calibration 138-145. Current value 140 falls in range.

**Action needed from Presten:** None immediately — but run the validation SQL in `Reports/ga-calibration-validation-2026-04-23.md` Query 1 before May 18. If the actual win rate implies a value outside 135-150, flag for adjustment.

---

## Item 3: GA Boys (100) — Validation Uncertain

**Issue:** GA boys calibration (100) may not be validatable from our data due to small sample sizes. GA is primarily a girls league; boys GA events are rare.

**Action needed from Presten:** When running the validation queries, check sample size for GA boys vs ECNL boys cross-league games. If sample is below 200 games, accept 100 as a reasonable prior. No action needed. If sample is above 200, review the win rate and confirm calibration.

---

## Timeline

All three items should be resolved before: **May 15, 2026** (three days before the planned recalculation window).

The GA ASPIRE item is the only one that requires a decision to implement a change. Items 2 and 3 are confirmatory.

---

## Related Documents

- `Reports/ga-data-coverage-audit-2026-04-23.md` — full audit with SQL
- `Reports/ga-calibration-validation-2026-04-23.md` — cross-league win rate analysis
- [[GA]] — updated GA league note
- [[League Hierarchy]] — current calibration values
