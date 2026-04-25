---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: pending
priority: medium
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — Rankings Impact Preview.md"
---

# Task: Event Strength Phase 1 — Rankings Impact Preview

## Context

Event Strength Phase 1 is authorized only after:
1. April 28 execution log confirms all 3 steps completed (SENTINEL's G1 gate)
2. Calibration stability is confirmed post-April-28 (ELO's monitoring spec)

The Event Strength Phase 1 Execution Package (filed 2026-04-24) covers the implementation steps. What it does not contain: ELO's prediction of what Phase 1 will do to rankings.

Phase 1 introduces event strength as a modifier in rating computation. That is a material change. SENTINEL needs to know, before authorizing Phase 1 to run:
- Which age groups and leagues are most affected
- Whether Phase 1 changes are additive (shift existing ratings up/down) or redistributive (narrow/widen spreads)
- What anomaly thresholds SENTINEL should use to confirm Phase 1 worked as expected

Without this preview, SENTINEL's Phase 1 authorization is: "ELO says it's ready, so run it." With it, SENTINEL's authorization is: "ELO predicts changes X, Y, Z — if April 29 results match the model, Phase 1 is authorized."

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — Rankings Impact Preview.md`

---

### Section 1: Mechanism Summary

One paragraph: what does Event Strength Phase 1 actually add to the rating computation?

Specifically:
- What is the event strength modifier and how does it interact with existing K-factor/calibration values?
- Is it a multiplier, an additive term, or a lookup table adjustment?
- Does it apply to all events or only specific tiers/leagues?

If Event Strength Phase 1 is scoped to a specific league or tier subset, state that scope explicitly. This determines which age groups are affected.

---

### Section 2: Affected Age Groups and Leagues

For each age group, estimate whether Phase 1 will have High / Medium / Low / Zero impact on ratings:

| Age Group | League(s) Affected | Expected Impact Level | Reason |
|-----------|--------------------|----------------------|--------|
| U13G | | | |
| U13B | | | |
| U14G | | | |
| U14B | | | |
| U15G | | | |
| U15B | | | |
| U16G | | | |
| U16B | | | |
| U17G | | | |
| U17B | | | |

Impact level definition (ELO sets the thresholds):
- **High:** Average rating shift expected > [N] pts
- **Medium:** Average rating shift expected [N/2]–[N] pts
- **Low:** Average rating shift < [N/2] pts
- **Zero:** Phase 1 does not apply to this age group

---

### Section 3: Predicted Rating Direction

For each High-impact age group: will ratings increase or decrease on average, and for which teams?

- Do teams at top-tier events gain or lose rating points relative to teams at lower-tier events?
- Do teams at lower-tier events gain or lose rating points?
- Net expected effect on rating spread (wider / narrower / same)

One sentence per age group at High impact level. This gives SENTINEL a clear expectation to compare against post-Phase-1 analysis.

---

### Section 4: Anomaly Detection Thresholds

What post-Phase-1 results would indicate Phase 1 did NOT run as expected?

| Signal | Expected | If Outside Range: Interpretation |
|--------|----------|----------------------------------|
| High-impact age group avg rating change | [predicted direction, N pts] | If opposite direction → Phase 1 computation error |
| Zero-impact age group avg rating change | ~0 pts | If > [N] pts → Phase 1 scope exceeded spec |
| Rating spread change (top–bottom) | [wider/narrower/same] | If opposite → modifier logic inverted |
| Teams with > [threshold]-pt shift | [estimate] | If much higher → modifier magnitude larger than expected |

These thresholds feed SENTINEL's Phase 1 post-run review. If actual results fall within these ranges, Phase 1 ran as predicted. If not, ELO investigates before DSS.

---

### Section 5: DSS Demo Implication

One paragraph: after Phase 1 runs (assuming it goes as predicted), what changes about the rankings story ELO tells at DSS?

- Are there specific teams or clubs whose ranking improves significantly due to event strength recognition?
- Does Phase 1 make the rankings more or less aligned with competitor platforms (e.g., USA Rank)?
- Should SENTINEL update the DSS demo script to reference event strength as a differentiator?

---

## Quality Standard

- This is a **prediction document** marked `[Pre-Run Preview — Written before Event Strength Phase 1 execution]`
- Section 2 impact levels must be based on the Phase 1 implementation spec, not on intuition — reference the spec
- Section 4 thresholds must be specific numbers. "Significant" or "large" are not thresholds.
- If ELO is uncertain about Phase 1's mechanism (e.g., the spec is underspecified), file a Queue item rather than guessing — and note in Section 1 what is uncertain

## Why This Matters

Event Strength Phase 1 is the most significant rating system change between now and DSS. SENTINEL's current authorization path is narrative review only. This document converts that to a structured prediction-vs-reality check: run Phase 1, compare against Section 4 thresholds, confirm or flag. The same pattern that made the April 28 Pre-Run Model useful for April 29 analysis applies here. Writing it before Phase 1 runs is the only way the post-run review is scientifically meaningful.

## References

- `Rankings/Event Strength — Phase 1 Execution Package.md` — the implementation this preview models
- `Rankings/Event Strength — Phase 2 Scope.md` — Phase 2 context (not in scope for this doc, but worth understanding Phase 1 boundary)
- `Rankings/Post-April-28 Expected Rating Shift — Pre-Run Model.md` — template for this document's format
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — May 5 stability gate that precedes Phase 1 authorization
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Phase 1 readiness cell
