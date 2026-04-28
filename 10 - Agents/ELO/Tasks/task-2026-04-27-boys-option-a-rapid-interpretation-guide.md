---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Rapid Interpretation Guide.md"
---

# Task: Boys Option A Rapid Interpretation Guide

## Context

ELO has committed to completing the Boys Option A Decision Document within 48 hours of receiving Presten's query results. The interpretation window opens April 29. The decision document template is already filed. What does not exist: the interpretation logic itself — the rules ELO uses to read specific query output patterns and map them to conclusions.

Without pre-written interpretation logic, ELO enters the 48-hour window and re-derives the interpretation criteria from scratch while working under deadline. This produces one of two failure modes:
- **Over-caution:** ELO hedges on edge cases because the threshold definitions weren't decided in advance, and the 48-hour clock ticks while ELO deliberates.
- **Under-rigor:** ELO produces an interpretation that hasn't been validated against the expected output structure, introducing calibration errors.

Writing the interpretation guide now — before the results arrive — forces ELO to commit to specific thresholds while there's no pressure to land a particular outcome. The guide becomes the interpretation standard ELO applies mechanically when results arrive.

---

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Rapid Interpretation Guide.md`

---

### Section 1: What Boys Option A Tests

One paragraph: what does the Boys Option A analysis measure, what query output does Presten produce, and what question does ELO need to answer from those results?

Specifically:
- What is the expected SQL output structure (columns, row structure)?
- Is ELO interpreting a full recalibration result, or a cross-league win rate analysis, or both?
- What calibration hypothesis is Option A testing? State it explicitly.

This section locks ELO's understanding of what the results represent before results arrive.

---

### Section 2: Expected Output Patterns

Define 3–4 distinct output patterns ELO should be ready to interpret:

**Pattern A — PASS (calibration is accurate):**
- What specific query result values indicate calibration is working correctly for Boys?
- Example: "Boys ECNL teams win X% of cross-league games against Boys GA teams — consistent with the expected tier differential."
- ELO commits to specific numeric ranges for each metric that constitute a PASS.

**Pattern B — SOFT FAIL (recalibration needed but minor):**
- What result values indicate miscalibration that is meaningful but correctable with a targeted adjustment?
- What is the magnitude threshold that separates "minor" from "significant"?

**Pattern C — HARD FAIL (significant miscalibration, requires full Boys recalibration cycle):**
- What result values indicate a calibration problem that cannot be fixed with a targeted parameter tweak?
- What does ELO recommend in this case? (Suspend Boys Club Rankings until corrected? Proceed with Girls-only at DSS?)

**Pattern D — AMBIGUOUS (results are in the grey zone):**
- What result ranges put ELO in a grey zone where a clear PASS/FAIL verdict is not possible?
- What additional query would ELO request from Presten to resolve the ambiguity?
- Maximum time ELO will spend in ambiguous state before escalating to SENTINEL: [ELO states the limit].

---

### Section 3: Decision Mapping

| Pattern | ELO Verdict | Decision Document Filing | SENTINEL Action Required |
|---------|------------|------------------------|------------------------|
| A — PASS | Option A calibration confirmed | File within 24 hours | None — ELO proceeds |
| B — SOFT FAIL | Targeted adjustment identified | File within 24 hours with adjustment spec | SENTINEL reviews adjustment before Presten executes |
| C — HARD FAIL | Boys recalibration required | File within 48 hours with escalation | SENTINEL escalates to Presten; Club Rankings Boys deferred |
| D — AMBIGUOUS | Additional query requested | No decision document until resolved | SENTINEL notified; ELO requests 1 additional query |

---

### Section 4: Filing Protocol

When results arrive, ELO executes in this order:

1. Read the query results against Section 2 patterns. Time-box this to 30 minutes.
2. Identify the matching pattern (or AMBIGUOUS if none match clearly).
3. File the determination in the FORGE briefing or ELO queue — "Option A: Pattern [X] observed — Decision Document in progress."
4. Complete the Decision Document within the pattern's time window (see Section 3).
5. If Pattern C: file a SENTINEL queue item within 2 hours of identifying it, before completing the full decision document.

The 30-minute pattern identification is non-negotiable. ELO should not spend more than 30 minutes on initial classification — if it takes longer, the result is AMBIGUOUS by default.

---

### Section 5: Known Edge Cases

Identify any edge cases ELO is aware of that could produce misleading results:

- Small sample sizes for specific age groups (if Boys have fewer cross-league games than Girls, the win rate confidence intervals are wider)
- Off-season timing effects (fewer recent games means results lean heavily on older data)
- MLS NEXT tier split effects: if Boys MLS NEXT teams were recently re-tiered, their ratings may not yet reflect the new tier calibration

For each edge case: state whether ELO accounts for it in the interpretation thresholds, and if not, how ELO flags it in the decision document.

---

## Quality Standard

- All thresholds in Section 2 must be specific numeric ranges, not directions ("higher than expected" is not a threshold).
- Section 3 filing timelines must be specific: 24 hours, 48 hours — not "as soon as possible."
- Section 4 must be a sequence of actions, not a description. ELO must be able to read Section 4 while actively working and know exactly what to do next.
- This is a pre-written guide, not a results document. ELO commits to interpretation logic that is independent of what the results actually say.

## Why This Matters

The Boys Option A decision is on the critical path for Club Rankings Boys inclusion and for the May 17 sprint plan. If ELO's 48-hour interpretation window slips to 72 hours because the interpretation logic wasn't pre-defined, SENTINEL misses the May 1 Club Rankings path decision window. With a pre-written guide, ELO opens the results, identifies the pattern in 30 minutes, and produces the decision document in under 24 hours for the most common patterns. Speed comes from not having to design the interpretation logic under deadline.

## References

- `Rankings/Boys Option A — Decision Document.md` (or equivalent template) — the document ELO fills using this guide
- `Rankings/Boys Calibration Status Summary — April 2026.md` — baseline Boys calibration state
- `Rankings/GA Coverage Audit — Boys Query Execution Package.md` — Boys cross-league query design
- `Rankings/Club Rankings — Girls-Only vs. Full Launch Recommendation.md` — why Boys calibration matters for DSS timing
