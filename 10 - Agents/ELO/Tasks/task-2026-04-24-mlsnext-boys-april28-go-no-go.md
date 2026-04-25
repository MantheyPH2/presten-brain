---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: high
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Boys Tier Split — April 28 Decision.md"
---

# Task: MLS NEXT Boys Tier Split — April 28 Go/No-Go Decision

## Context

ELO completed two MLS NEXT calibration documents:
- `Rankings/MLS NEXT Tier Split Calibration Spec — April 2026.md` (`task-2026-04-24-mlsnext-tier-split-calibration-spec`)
- `Rankings/MLS NEXT Calibration Application Spec — April 2026.md` (`task-2026-04-24-mlsnext-calibration-application-spec`)

The April 28 Escalation Reference Card (`Product & Planning/April 28 Escalation Reference Card.md`) currently covers two items:
1. ECNL spec review (~15 min)
2. GA ASPIRE tier fix (~45–60 min)

The MLS NEXT Boys tier split is NOT in the reference card. SENTINEL has no record of a decision being made to include or exclude it from April 28. The calibration work was completed, but the scheduling decision is unresolved.

This task: ELO makes the call and documents it. One document, one clear decision.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Boys Tier Split — April 28 Decision.md`

---

### Section 1: What the Tier Split Changes

One paragraph. Summarize the MLS NEXT Boys tier split as designed — what parameters change, which age groups are affected, expected direction of rating movement for Tier 1 vs Tier 2 teams.

Pull directly from the application spec. Do not re-derive.

### Section 2: April 28 Feasibility Check

Answer these three questions:

**Q1: Does the GA ASPIRE fix create a sequencing dependency?**
The GA ASPIRE fix (April 28, Item 2) updates `event_tier` for GA ASPIRE events. The MLS NEXT tier split changes calibration values for MLS NEXT events. Are these independent operations, or does one affect the other?
- If independent: both can run on April 28 in sequence. State the correct order (ASPIRE first, or MLS NEXT first — and why).
- If dependent: one must complete and rankings must recompute before the other begins. State whether this is feasible in a single April 28 session or requires a separate session.

**Q2: How much additional Presten time does the tier split add?**
From the application spec: what is the estimated execution time for the MLS NEXT tier split (just the application, not the Brier analysis)?
- If ≤30 minutes: append to April 28 with minimal risk.
- If >30 minutes: flag as a scheduling constraint.

**Q3: Is the calibration validated for Boys MLS NEXT Tier 2?**
The Boys calibration gap analysis flagged uncertainty about Boys-specific calibration. Is the Boys MLS NEXT tier split calibration value validated to the same standard as the Girls fix, or is it a structural analogy? State clearly.
- If validated: GO is justified.
- If structural analogy only: recommend deferring to post-DSS alongside the Boys Brier analysis.

### Section 3: Decision

State one of:

**OPTION A — Include on April 28:**
"MLS NEXT Boys tier split is approved for April 28. Add to reference card after GA ASPIRE fix. Estimated additional time: [N minutes]. No sequencing conflicts. SQL to add: [copy-paste from application spec]."

**OPTION B — Defer to post-DSS:**
"MLS NEXT Boys tier split deferred to post-DSS. Reason: [sequencing conflict / unvalidated Boys calibration / time constraint]. Re-open this decision after the Boys Brier analysis (May 17–30)."

### Section 4: If Option A — Reference Card Update

If the decision is Option A, specify exactly what to add to `Product & Planning/April 28 Escalation Reference Card.md`:
- The section header
- The SQL (verbatim from the application spec)
- The post-application verification query
- The rollback SQL

Format it so it can be copy-pasted directly into the reference card without modification.

## Definition of Done

- Decision document written at `Rankings/MLS NEXT Boys Tier Split — April 28 Decision.md`
- All four sections present
- Decision is unambiguous: GO or DEFER (not "it depends")
- If GO: reference card addition is copy-paste ready
- If DEFER: a re-open condition is stated
- Summary in briefing: "MLS NEXT Boys tier split: [GO on April 28 / DEFERRED post-DSS]. [Reason in one sentence]."

## Why This Matters

The April 28 session is 4 days away. The reference card is staged and Presten-facing. If the MLS NEXT tier split belongs on April 28, the card needs to be updated before then — not during the session. If it doesn't belong on April 28, SENTINEL needs to know that explicitly so it doesn't get added ad hoc on the day.

This is a scheduling decision, not more calibration work. ELO has the data to make it. Make it now.

## References

- `02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split Calibration Spec — April 2026.md`
- `02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Calibration Application Spec — April 2026.md`
- `02 - Tiger Tournaments/Projects/Product & Planning/April 28 Escalation Reference Card.md`
- `10 - Agents/ELO/Tasks/task-2026-04-24-boys-calibration-gap-analysis.md` — Boys calibration uncertainty context
