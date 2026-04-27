---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Query Pack Validation Note.md"
---

# Task: Boys Option A Query Pack Validation Note

## Context

The Boys Option A spot check window opens April 29. Presten will run 3 queries from `Rankings/Boys Option A Spot Check — Presten Execution Package.md`. The results feed into `Rankings/Boys Option A — Decision Document.md`, which has pre-written result tables for Presten to fill in.

What has not been verified: whether the 3 queries in the execution package are aligned with the pre-written tables in the Decision Document. Specifically:
- Do the query output columns match the column headers in the Decision Document's result tables?
- Do the query output row structure (by age group, by tier) match what ELO needs to fill in the pass/fail verdicts?
- Are the query filter conditions correct for the post-April-28 DB state? (e.g., do the queries correctly handle GA ASPIRE after the event_tier fix?)

If there is a mismatch, Presten runs the queries on April 29 and produces output that doesn't slot cleanly into the Decision Document tables. ELO then needs to re-interpret the output under time pressure, or file an incomplete Decision Document. That delays the Boys Option A verdict and the go/no-go for Boys club rankings.

ELO audits the query pack now, while there is no time pressure, and files a confirmation note. If changes are needed, ELO updates the queries in the execution package before April 29.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Query Pack Validation Note.md`

---

### Section 1: Query Inventory

List all 3 queries from the execution package with a one-line description:

| Query | Description | Output Columns | Filters |
|-------|-------------|---------------|---------|

Pull from `Rankings/Boys Option A Spot Check — Presten Execution Package.md`. Do not paraphrase — copy the exact output columns and filter conditions as specified in the package.

---

### Section 2: Decision Document Alignment Check

For each query, check that its output aligns with the pre-written tables in `Rankings/Boys Option A — Decision Document.md`:

| Query | Decision Doc Section | Output Column Match? | Row Structure Match? | Notes |
|-------|---------------------|---------------------|---------------------|-------|
| Query 1 | | ✅ / ❌ | ✅ / ❌ | |
| Query 2 | | ✅ / ❌ | ✅ / ❌ | |
| Query 3 | | ✅ / ❌ | ✅ / ❌ | |

**Column match:** Does each column in the query output map to a field in the Decision Document table? If the query returns `avg_rating_boys` and the Decision Document expects `boys_avg`, that is a ❌.

**Row structure match:** Does the query return one row per age group (or the expected grouping)? If the Decision Document tables are organized by age group and the query does not group by age group, that is a ❌.

---

### Section 3: Post-April-28 Filter Validity

After April 28, the DB will have:
- GA ASPIRE event_tier fixed (previously 'ga', now 'ga_aspire' for those events)
- ecnl_verified column added with default FALSE (all existing rows = FALSE until FORGE backfill runs)

ELO checks: do any Boys Option A queries reference `event_tier` or `ecnl_verified` in their filter conditions?

| Query | References event_tier? | References ecnl_verified? | Post-April-28 Filter Valid? | Notes |
|-------|----------------------|--------------------------|---------------------------|-------|
| Query 1 | | | | |
| Query 2 | | | | |
| Query 3 | | | | |

**Concern to check:** If Query 1 filters on `event_tier = 'ga'` for Boys events (not GA ASPIRE, which is Girls-only), that filter is still valid post-fix. But if any query assumes GA ASPIRE was in 'ga' tier and uses that for Boys analysis, the filter may return unexpected results after the fix. ELO confirms Boys queries are not affected by the April 28 event_tier change.

---

### Section 4: Required Corrections (If Any)

If any check in Sections 2–3 returns ❌:

| Query | Issue | Correction Required | Updated Query | ELO Confidence |
|-------|-------|--------------------|--------------:|---------------|

ELO makes the correction in the execution package directly and notes it here. If the correction changes the pass/fail threshold in the Decision Document, ELO updates both documents simultaneously.

If no corrections are required: "No corrections needed. All 3 queries are aligned with the Decision Document and post-April-28 DB state."

---

### Section 5: ELO Clearance

One line:

```
Boys Option A query pack cleared for April 29 execution: YES / NO (corrections pending)
Last reviewed: [date/time]
```

If "NO": specify which query requires correction and what must change before Presten runs it.

---

## Quality Standard

- Section 2 checks must be done by reading both the execution package and the Decision Document side-by-side. Do not rely on memory of what they contain.
- If the execution package or Decision Document was filed before April 28 pre-conditions were set (i.e., before ecnl_verified existed), Section 3 is especially important.
- Any correction in Section 4 must be reflected in the actual execution package file — not just described in this note. ELO updates `Rankings/Boys Option A Spot Check — Presten Execution Package.md` directly.
- Section 5 clearance is what SENTINEL reads first. The detail is in Sections 2–4.

## Why This Matters

Boys Option A has a hard window: April 29 – May 5. If Presten runs the queries on April 29 and the output doesn't match the Decision Document tables, ELO cannot file a complete verdict within 48 hours. That pushes the Boys club rankings decision past May 5, which compresses the May 9 DSS readiness assessment. Pre-validating the query pack on April 28 costs 20 minutes and eliminates that failure mode entirely. Given that April 29 is the most loaded session in the plan, ELO should not be debugging query alignment while also running 4 other workstreams.

## References

- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — the query pack being validated
- `Rankings/Boys Option A — Decision Document.md` — the document this pack feeds
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys baseline context
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Boys cell this verdict feeds
