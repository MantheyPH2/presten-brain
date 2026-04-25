---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: high
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/May 9 DSS Readiness Final Assessment Spec.md"
---

# Task: May 9 DSS Readiness Final Assessment Spec

## Context

May 9 is the final DSS go/no-go date — one week before the demo on May 16. ELO is responsible for updating the DSS Feature Readiness Tracker on May 9 and filing a final readiness summary to SENTINEL. Without a spec for what to check and how to decide, the May 9 review will be a high-pressure judgment call under time constraints.

This task makes May 9 a 30-minute mechanical check with pre-defined pass/fail criteria. ELO writes the spec now (before May 9), so that on May 9 the session is just execution, not design.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/May 9 DSS Readiness Final Assessment Spec.md`

### Sections Required

#### Section 1: Per-Feature Final Check

For each of the 5 DSS features in the Readiness Tracker, specify:
- The exact query or check to run (copy-paste SQL or step)
- The PASS threshold that upgrades ⚠️ → ✅
- The FAIL condition that confirms ❌ (and which demo plan it forces: primary vs. fallback)

Features to cover: Rank Bands, Event Strength Phase 1, Club Rankings, USA Rank Comparison, USYS/GotSport Coverage.

#### Section 2: Decision Table

A single table: "If [feature] is ❌ on May 9, then [action]."

| Feature | ❌ on May 9 → Action |
|---------|---------------------|
| Rank Bands | Fallback demo only. Notify SENTINEL immediately. |
| Event Strength Phase 1 | Drop from demo. Update DSS Demo Spec Section 5 fallback slide. |
| Club Rankings | Drop from demo. Girls-only if Boys Option A failed; if Girls also ❌, drop entirely. |
| USA Rank Comparison | Fallback demo only. This is non-negotiable; if ❌, contact Presten directly. |
| USYS/GotSport Coverage | Drop coverage claim from script. Do not improvise live. |

(Adjust based on actual DSS Demo Spec and Readiness Tracker — ELO should not copy this table verbatim but derive it from those documents.)

#### Section 3: ELO Output Format for May 9

Specify exactly what ELO's May 9 briefing should include: updated Readiness Tracker columns, a summary line for SENTINEL ("Data ready: N/5. Calibration ready: N/5. Primary demo features: [list]. Fallback features: [list]."), and a flag if any non-negotiable item (USA Rank Comparison) is ❌.

#### Section 4: May 14 Pre-Demo Final Check

Short section: on May 14, ELO runs the `DSS Data Readiness Validation Plan — May 14-15.md` queries. If any result changes between May 9 and May 14, ELO flags to SENTINEL by 11 AM. No changes to demo plan after noon May 14.

## Quality Standard

On May 9, Presten should be able to hand ELO this spec and say "run it." ELO returns with a completed checklist and a one-paragraph SENTINEL summary within one session. Every SQL query must be copy-paste ready. Every decision must be explicit — no "use your judgment" language in the decision table.
