---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-05-09
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Decision Document.md (completed with actual results)"
---

# Task: Boys Option A Spot Check — Execution and Verdict

## Context

ELO filed `Rankings/Boys Option A — Decision Document.md` on April 26. That document is a decision template with expected values pre-populated. It cannot be completed until Presten runs the three spot-check queries and sends results to ELO.

The query execution window is April 29 – May 5. May 9 is the deadline for the Boys Club Rankings go/no-go. That leaves ELO 4 days (May 5–9) to classify results, fill the decision document, and file the verdict.

**This task has a dependency:** Presten must run the three Boys Option A queries from `Rankings/Boys Option A Spot Check — Presten Execution Package.md`. ELO cannot complete this task without those results. However, ELO is responsible for:
1. Reminding SENTINEL in each briefing whether results have arrived
2. Completing the decision document within 48 hours of receiving results
3. Filing the go/no-go verdict no later than May 9

## What to Do

### When Results Arrive (target: April 29 – May 5)

As soon as Presten sends Boys Option A query output, ELO:

**Step 1: Fill Q1 Results Table (Boys vs. Girls Rating Delta)**

Open `Rankings/Boys Option A — Decision Document.md`. Navigate to the Q1 results table. The expected values column is already filled with: expected delta ≤ 75 pts (hard FAIL threshold: > 100 pts for any age group). Fill the "Actual" column with Presten's output.

| Age Group | Expected Delta (Boys vs. Girls) | Actual Delta | PASS / FAIL |
|-----------|--------------------------------|--------------|-------------|
| U13 | ≤ 75 pts | [actual] | |
| U14 | ≤ 75 pts | [actual] | |
| U15 | ≤ 75 pts | [actual] | |
| U16 | ≤ 75 pts | [actual] | |
| U17 | ≤ 75 pts | [actual] | |

A single age group > 100 pts delta is an automatic FAIL (activate Girls-only fallback).

**Step 2: Fill Q2 Results Table (Boys GA Game Volume)**

Expected minimum: ≥ 500 Boys GA games per age group. Fill actual counts from Presten's output.

If any age group is below 500: note as borderline. Does not auto-fail, but ELO must assess whether the sample is large enough to trust the Boys calibration.

**Step 3: Fill Q3 Results Table (MLS NEXT Boys Distribution)**

Expected distribution: median 1,400–1,600; top 10% ≥ 1,700; bottom 10% ≤ 1,300. Compare against Presten's output.

---

### ELO Analysis Section (must be written before filing verdict)

Two paragraphs. No hedging.

**Paragraph 1:** What do the three results tables tell ELO about the state of Boys calibration? Reference specific numbers. Are Q1 deltas consistent with GA overcalibration affecting Boys, or is this a different pattern? Does Q2 game volume provide sufficient confidence to trust the Boys ratings?

**Paragraph 2:** Given the evidence, which outcome applies?
- **PROCEED:** All Q1 deltas ≤ 75 pts, Q2 volume sufficient, Q3 distribution plausible. Boys Club Rankings included in DSS demo.
- **ACTIVATE GIRLS-ONLY FALLBACK:** Any Q1 delta > 100 pts. Boys Club Rankings excluded. ELO activates `Rankings/Club Rankings — Girls-Only Fallback Spec.md` immediately.
- **CONDITIONAL PROCEED:** Q1 deltas 75–100 pts. ELO recommends Boys inclusion with caveats. State the caveats explicitly — SENTINEL must approve conditional inclusion before it is presented at DSS.

---

### SENTINEL Authorization Request

After the ELO analysis section, fill the authorization request block:

```
ELO decision: PROCEED / ACTIVATE FALLBACK / CONDITIONAL PROCEED
Basis: Q1 [pass/fail], Q2 [pass/fail], Q3 [pass/fail]
If CONDITIONAL: caveats are [list]
SENTINEL disposition: [leave blank — SENTINEL fills this]
Date filed: [date]
```

---

### Status Tracking (ELO reports in every briefing until resolved)

In each ELO briefing from April 29 to May 9, include one line:
- "Boys Option A: awaiting Presten query results (day [N] of window)"
  OR
- "Boys Option A: results received [date], analysis in progress"
  OR
- "Boys Option A: decision filed [date], awaiting SENTINEL disposition"

---

## Quality Standard

- ELO must complete the decision document within 48 hours of receiving results. The May 9 deadline is fixed — if results arrive May 5, the document must be filed May 7 at the latest.
- Do not file a CONDITIONAL PROCEED without explicit caveats. Vague hedging ("Boys calibration may have issues") is not acceptable for a DSS feature decision.
- If results arrive and Q1 auto-fails (any age group > 100 pts), ELO activates the Girls-only fallback immediately — same session, same briefing. Do not wait for SENTINEL confirmation to activate the fallback.
- The decision document is the DSS feature authorization record. Write the analysis as if it will be read by Presten at DSS prep.

## If Results Do Not Arrive by May 5

If Presten has not run the Boys Option A queries by May 5, ELO flags in the May 5 briefing:

> "Boys Option A window closes May 5. Results not yet received. Boys Club Rankings cannot be authorized for DSS without the spot check. SENTINEL must decide: extend window or default to Girls-only. If Girls-only fallback activates by default, FORGE needs 7 days minimum to implement — deadline May 9, DSS May 16."

This flag gives SENTINEL and Presten 4 days (May 5–9) to act before the implementation window closes.

## Why This Matters

Boys Club Rankings is a DSS demo feature. SENTINEL flagged in the April 26 06:39 briefing: "the authorization to include Boys needs to be documentable." If ELO does not file this decision document before DSS, Boys inclusion is undocumented. More critically: if Boys calibration is off and ELO does not catch it via this spot check, the DSS demo presents inaccurate Boys rankings to potential clients. The spot check exists to prevent that scenario. This task makes sure it actually runs and produces a signed decision.

## References

- `Rankings/Boys Option A — Decision Document.md` — the document to complete
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — Presten's query execution guide
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — activated if Q1 fails
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Boys Club Rankings cell
- `Rankings/Boys Calibration Status Summary — April 2026.md` — baseline context
