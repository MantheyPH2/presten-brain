---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
due: 2026-04-30
priority: high
status: completed
topic: boys-ecnl-calibration-scope-documentation
---

# Task: Boys ECNL Calibration Scope Documentation

## Context

The 08:46 SENTINEL briefing identified a coverage gap: "Boys ECNL Boys calibration status is not explicitly documented anywhere in the vault." The DSS Feature Readiness Tracker notes Boys GA (cal=100) and Boys ECNL (cal=120) are "theoretically sound but not Brier-validated." Boys club rankings can only be shown at DSS if Boys Option A pre-flight validation passes. Before that pre-flight runs, ELO must have a clear, documented answer to what Boys ECNL scope is and what calibration assumptions back it.

This is not a research task — ELO should already know this. This is a documentation task to close a visibility gap for SENTINEL.

## Deliverable

File `Rankings/Boys-ECNL-Calibration-Scope-2026-04-27.md` in `02 - Tiger Tournaments/Projects/` with:

1. **Leagues in scope:** Which Boys ECNL tiers are included in Evo Draw rankings? (ECNL Boys, ECNL Regional League Boys, etc.)
2. **Calibration values:** What cal value(s) are assigned to Boys ECNL? Is it a single value or tiered by league level?
3. **Cross-League Analysis basis:** Does the 575K game win-rate study include Boys ECNL? If not, what is the calibration value based on?
4. **Validation status:** Has Boys ECNL cal been Brier-validated? If not, what is the confidence basis (expert judgment, relative to Boys MLS Next cal=110, etc.)?
5. **Interaction with Boys Option A:** If Boys Option A passes, do Boys ECNL games contribute to Boys club rankings? What weight?
6. **Known risks:** Any known miscalibration signals for Boys ECNL? Any leagues ELO suspects are miscalibrated?
7. **Post-DSS work needed:** What Brier validation work should be scheduled for Boys ECNL after May 16?

## Success Criteria

- SENTINEL can answer "what is Boys ECNL calibration status?" from this document without asking ELO
- Boys Option A pre-flight on April 29 can explicitly reference this document as the calibration basis

## Notes

- This is a documentation gap, not a calibration gap — if Boys ECNL calibration is sound, say so with evidence
- If ELO finds in writing this that there IS a calibration issue, escalate to SENTINEL immediately via Queue before filing
