---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: urgent
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Results Interpretation Guide.md"
---

# Task: Boys Option A — Results Interpretation Guide

## Context

The Boys Option A Spot Check Execution Package (filed 2026-04-25) gives Presten two queries to run between April 29 and May 5. The queries return numbers. What the execution package does not contain: how to interpret those numbers. Presten runs Query 1, gets a result table, and then must decide: does this pass or fail? The execution package states the threshold (> 100 pts divergence = FAIL) but does not explain what "Boys vs. Girls divergence for the same age group" looks like in practice, what values indicate a healthy system vs. a broken one, or what the pass result should look like.

This is a gap in the Boys Option A authorization chain. ELO should write the interpretation guide before April 29 — before Presten runs the queries. If results need interpretation under time pressure, having the guide ready means ELO can respond to Presten's results in < 30 minutes instead of re-deriving the framework.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Results Interpretation Guide.md`

This is ELO's annotated reader's guide to the Boys Option A query outputs. It assumes Presten has run the queries and is looking at results.

---

### Section 1: What a Passing Result Looks Like

For **Query 1 — Boys vs. Girls Rating Divergence:**

Describe the expected output format (columns, approximate row count) and what ELO expects to see in a passing result:
- What is the expected Boys–Girls divergence for a healthy calibration? Is 40 pts normal? 80 pts?
- Are there specific age groups where higher divergence is expected (e.g., U17 Boys typically rated higher than U17 Girls)?
- What does a clean, passing result table look like? Give an example row:
  ```
  age_group | boys_avg | girls_avg | divergence
  U15B/G    | 1420     | 1380      | 40        -- PASS: within expected range
  ```

For **Query 2 — Boys Tier Consistency Check:**

What does a healthy result look like? Describe the expected distribution and what values confirm Option A is calibrating Boys tiers correctly.

---

### Section 2: What a Failing Result Looks Like

For Query 1:
- One example of a FAIL result with explanation:
  ```
  age_group | boys_avg | girls_avg | divergence
  U15B/G    | 1520     | 1380      | 140       -- FAIL: > 100 pts, exceeds threshold
  ```
- Is the failure direction asymmetric? (i.e., is 140 pts Boys-higher different from 140 pts Girls-higher?)
- Are there age groups where ELO expects query results to be noisy (low game volume, wide confidence intervals)? Flag them so Presten doesn't over-react to a borderline result in a low-volume age group.

---

### Section 3: Borderline Results

The threshold is > 100 pts = FAIL. But what about 95–105 pts? ELO should state a policy:
- Is the 100-pt threshold hard (anything above triggers Girls-only fallback) or soft (100–110 requires ELO investigation before deciding)?
- If a result is 98 pts for one age group and 85 pts for all others, what is the verdict?
- One sentence ELO files in the briefing when a result is borderline: "Boys Option A result: [N] age groups borderline (95–110 pts). ELO assessment: [pass/investigate/fail] because [reason]."

---

### Section 4: How to Send Results to ELO

Presten runs the queries during a session and either:
a) Pastes results into the briefing (if ELO is reviewing asynchronously)
b) Files results in the Queue for ELO to assess

State which format ELO needs. One paragraph:
- Exact file path where Presten should save raw query output: `Reports/boys-option-a-results-[YYYY-MM-DD].md`
- Format: copy-paste from psql is fine; ELO will parse it
- ELO turnaround: ELO files interpretation + verdict in the next briefing after results are received

---

### Section 5: Verdict Formats

ELO files one of three verdicts in the briefing after reviewing results:

**PASS:**
> "Boys Option A spot check: PASS. Max divergence [N] pts (threshold: 100 pts). All [N] age groups within range. Boys club rankings cleared for full implementation. Girls-Only Fallback Spec on hold — not activated."

**FAIL:**
> "Boys Option A spot check: FAIL. [Age group X] divergence [N] pts, exceeds 100-pt threshold. Girls-Only Fallback Spec ACTIVATED. See `Rankings/Club Rankings — Girls-Only Fallback Spec.md`. FORGE informed."

**INCONCLUSIVE:**
> "Boys Option A spot check: INCONCLUSIVE. [N] age groups borderline. ELO investigation required. No immediate activation of Girls-Only Fallback — holding pending [specific next step by specific date]."

---

## Quality Standard

- Example result tables must use realistic rating values, not round numbers (e.g., 1385 not 1400)
- The verdict format strings must be copy-paste ready for ELO to use in the actual briefing
- Section 3 borderline policy must be ELO's actual call — not hedged language. If ELO would investigate anything between 90–110, say "90–110 requires investigation." That is a valid policy.

## Why This Matters

The Boys Option A spot check window is April 29 – May 5. The Club Rankings fallback decision is gated on May 5. If Presten runs the queries on April 30 and the results are borderline, ELO needs to file a verdict in the next briefing — not the next session. Without this interpretation guide, ELO is re-deriving thresholds under a May 5 deadline. With it, ELO reads one page and files the verdict in < 30 minutes.

## References

- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — the execution package this guide annotates
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — trigger: Query 1 FAIL
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys calibration baseline
- `task-2026-04-24-boys-calibration-option-a-interpretation.md` — earlier calibration interpretation work (check for overlap)
