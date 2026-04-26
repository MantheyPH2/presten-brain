---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Option A — April 29 Quick-Check Card.md"
---

# Task: Boys Option A — April 29 Quick-Check Card

## Context

Boys Option A is Steps 7–8 of the April 29 Session Master Script (conditional: only if Presten has run the Boys Option A queries). The Boys Option A Decision Document exists at `Rankings/Boys Option A — Decision Document.md` with expected values pre-populated.

What does not exist: a standalone quick-check card that Presten can run independently on April 29 without navigating the full Master Script. The Master Script is 130–170 minutes of dense work. Boys Option A is conditional — it may be run as its own session on April 30 or May 1 if April 29 runs long. In that case, Presten needs a standalone reference that doesn't require opening the Master Script and finding Steps 7–8.

The decision document covers the analytical decision. This quick-check card covers the execution: exactly which queries to run, in what order, what each output means, and what to file where.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Boys Option A — April 29 Quick-Check Card.md`

This is a short, linear document. Presten opens it, follows Steps 1–5, and is done in 20–30 minutes.

---

### Header Block

```
Purpose: Boys Option A calibration spot check
Reference: Rankings/Boys Option A — Decision Document.md (pre-populated with expected values)
Prerequisites:
  - April 28 fix confirmed (GA ASPIRE UPDATE + full recompute complete)
  - Access to psql production database
Estimated time: 20–30 minutes
When to run: April 29 (Step 7 of Master Script) or anytime April 29–May 5
```

---

### Step 1: Pull Boys vs. Girls Age Group Average Rating Delta

**Query:** [ELO writes the exact query]

Expected output format:
```
age_group | boys_avg_rating | girls_avg_rating | delta
U13B/U13G | [val] | [val] | [val]
...
```

Pass criterion: Delta ≤ 75 pts for every age group
Fail criterion: Delta > 100 pts for any age group

→ If FAIL: record which age group(s) and proceed to Step 2 for context
→ If PASS: note result, skip to Step 5

---

### Step 2 (Conditional — Only if Step 1 FAIL): Boys MLS NEXT Distribution Check

**Query:** [ELO writes the exact query]

Expected output: Distribution of Boys MLS NEXT team ratings by age group (min, p25, median, p75, max)

What to look for: Is the Boys MLS NEXT distribution centered near the expected elite range, or compressed? Compression relative to Girls ECNL distribution is the marker of Boys underweighting.

This is context for the Decision Document, not a standalone pass/fail.

---

### Step 3: Cross-Rank Comparison — 5 Reference Teams

Pull current ranks for 5 Boys reference clubs against their expected positions:

| Team Name | Age Group | Current Rank | Expected Rank | Delta |
|-----------|-----------|-------------|---------------|-------|
| [ELO fills known Boys reference teams with expected ranks from Decision Document] |

Pass criterion: All reference teams within 10 ranks of expected position
Concern criterion: Any reference team off by > 20 ranks

---

### Step 4: File Results in Decision Document

Open `Rankings/Boys Option A — Decision Document.md`. Fill in the "Query Results" section (Section 2 or wherever ELO positioned it). Fields:

- Step 1 delta results (all age groups)
- Step 1 verdict: PASS / FAIL
- Step 2 data (if run): Boys MLS NEXT distribution summary
- Step 3 reference team rankings
- Date and time of run

Do NOT write the final recommendation in the Decision Document yet — that is ELO's analysis step, not Presten's data-entry step.

---

### Step 5: File in Briefing

After completing Steps 1–4, file one of these notes in the next briefing (FORGE or ELO, whichever is active):

**If PASS:** "Boys Option A Step 1 PASS. All age group deltas ≤ 75 pts. Decision Document updated. ELO analysis within 48 hours."

**If FAIL:** "Boys Option A Step 1 FAIL. [Age group] delta = [N] pts (> 100 pt threshold). Decision Document updated with actuals. ELO analysis within 48 hours. Club Rankings Boys inclusion: conditional pending ELO review."

---

## Definition of Done

- Quick-check card written at `Rankings/Boys Option A — April 29 Quick-Check Card.md`
- All 5 steps present
- Step 1 and Step 3 queries are copy-paste ready SQL
- Pass/fail criteria for Step 1 match those in the Decision Document
- Card is self-contained: Presten can run it without opening any other document

## Why This Matters

The April 29 Session Master Script is a 130–170 minute document with 8 sequential steps. Boys Option A is a conditional branch that may not run on April 29 if the session runs long or Presten hasn't yet run the Boys queries. Having a standalone quick-check card means Boys Option A has no dependency on the Master Script's pacing. Presten can run it in a 30-minute window on April 30 or May 1 without re-reading the full script. The window closes May 5 — every day without the quick-check card is a day the window could compress.

## References

- `Rankings/Boys Option A — Decision Document.md` — the analysis document this card feeds
- `Rankings/April 29 ELO Session Master Script.md` — Steps 7–8 context (this card replaces the need to navigate those steps standalone)
- `Rankings/Boys Calibration Status Summary — April 2026.md` — background on Boys calibration state
- `Product & Planning/DSS Feature Readiness Tracker.md` — Boys cell gate this check feeds
