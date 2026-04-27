---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: medium
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Tier 2 Undefined Leagues — Calibration Recommendation.md"
---

# Task: Tier 2 Undefined Leagues — Calibration Recommendation

## Context

ELO's League Hierarchy Calibration Sanity Check (filed 2026-04-26) identified 4 Tier 2 leagues with no `event_tier` calibration value:
- USL Academy
- EDP Soccer
- Elite 64
- NAL (National Academy League)

ELO noted that these affect any game tagged to these leagues and provided a volume-check SQL query for Presten to run on April 29. SENTINEL flagged this as a medium-priority item: if these leagues represent > 2,000 games, pre-DSS calibration assignment is needed.

What does not yet exist: ELO's formal recommendation for what calibration values to assign. The question in SENTINEL's Queue asks Presten: "Do you authorize cal=55 for all four, or individual values?" That question implies ELO has already reasoned through the values. But there is no filed document showing that reasoning. If Presten answers "individual values," ELO needs to have pre-built that option.

This task: write the calibration recommendation document before April 29. When Presten runs the volume-check SQL, ELO's recommendation is ready to act on immediately — rather than starting a new analysis session.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Tier 2 Undefined Leagues — Calibration Recommendation.md`

---

### Section 1: League Profiles

For each of the 4 undefined leagues, document:

| League | Tier | Geographic Scope | Gender | Age Groups | Competitive Level vs. Known Benchmarks |
|--------|------|-----------------|--------|------------|---------------------------------------|
| USL Academy | 2 | National | Boys | U13–U17 | |
| EDP Soccer | 2 | Northeast Regional | Boys + Girls | U9–U19 | |
| Elite 64 | 2 | National (tournament format) | Boys + Girls | U12–U17 | |
| NAL | 2 | Regional | Boys | U13–U17 | |

ELO fills in the competitive-level column by comparing to known calibrated leagues. What ECNL tier or Regional league is this most comparable to? Use that comparison to anchor the calibration recommendation.

---

### Section 2: Calibration Reasoning

For each league, ELO provides:

**Proposed cal value:** [numeric value]

**Reasoning:** (2–3 sentences) What makes this the right value? What known league is the closest calibration anchor? Does this league trend above or below the Tier 2 midpoint?

**Confidence:** High / Medium / Low — and why.

**Acceptable range:** If ELO is uncertain, what is the acceptable band? (e.g., "35–55 would be defensible; below 35 would underweight this league's competitive level.")

Write this section for all 4 leagues. Do not defer the reasoning to "pending Presten's volume check." The volume check determines whether to apply the value before DSS — ELO's calibration reasoning is volume-agnostic.

---

### Section 3: Volume-Gated Decision Matrix

Presten will run the volume-check SQL on April 29. ELO pre-writes the decision matrix:

| Game Count (SQL Result) | Action |
|------------------------|--------|
| ≥ 2,000 games | Apply calibration before DSS: update League Hierarchy and trigger partial recompute |
| 500–1,999 games | Apply calibration before DSS if session time allows; not critical-path |
| < 500 games | Defer to post-DSS; low volume means minimal accuracy impact |
| 0 games | League may not be in use; confirm before adding cal value |

ELO should also specify: which game count crosses the threshold for DSS accuracy claim impact? (i.e., at what volume does "we don't have calibration values for these leagues" become a claim ELO cannot make with confidence?)

---

### Section 4: Implementation Spec (If Authorized)

If Presten authorizes the calibration values (one-line authorization in a session), ELO provides the exact steps:

1. Update `League Hierarchy` with the 4 new cal values
2. Identify affected games in the DB: `SELECT COUNT(*) FROM games WHERE event_tier IN ('usl_academy', 'edp', 'elite64', 'nal')` (confirm the exact event_tier strings ELO uses)
3. Trigger recompute: full recompute or partial (does this require a full recompute or just the affected leagues?)
4. Verify: post-recompute rating sanity check — do the newly calibrated teams shift in the expected direction?

SENTINEL will not authorize ELO to implement until Presten approves the values. But having the implementation spec pre-written means ELO can execute within one session of authorization rather than one session to plan + one session to execute.

---

### Section 5: DSS Accuracy Claim Impact

State explicitly: if these 4 leagues remain uncalibrated through DSS (May 16), what is the honest answer to "how complete are your calibration values?" for the DSS demo?

- Is this a claim ELO can still defend? (e.g., "These 4 leagues represent < 1% of games and have minimal impact on rankings accuracy for the demo teams.")
- Or does it materially undermine a claim ELO was planning to make?

This section gives SENTINEL the language to use if Presten asks about calibration completeness at DSS.

---

## Quality Standard

- Calibration values in Section 2 must be specific numbers — not "approximately 55" or "between 40 and 60."
- The reasoning in Section 2 must name a specific anchor league (e.g., "comparable to Regional Premier in competitive level, which is calibrated at 65").
- Section 3 game counts are thresholds, not ranges — pick the boundary and commit to it.
- Section 5 must give SENTINEL a sentence it can say to Presten, not a hedge.

## Why This Matters

The Tier 2 undefined league gap is currently a question for Presten: "what value do you want?" That is the wrong framing — ELO is the calibration expert and should be providing the recommendation, not asking Presten to make a calibration judgment call. Presten's role is authorization, not design. This document converts the question from "what value?" to "ELO recommends X for Y reasons — do you authorize?" That is the correct framing, and it closes the decision faster.

## References

- `Rankings/League Hierarchy Calibration Sanity Check — April 2026.md` — audit that identified this gap
- `Rankings/League Hierarchy.md` — current calibration values (anchor references)
- `Rankings/DSS Ranking Accuracy Claims Spec.md` — DSS claim this gap affects (Section 5 input)
- SENTINEL Queue `pending-2026-04-22-gotsport-api-risk.md` — carries the original question for Presten
