---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Club Rankings — Calibration Pre-Assessment.md"
---

# Task: Club Rankings — Calibration Pre-Assessment

## Context

ELO completed the Club Rankings design (`Rankings/Club Rankings Design.md`) and implementation plan (`Rankings/Club Rankings — Implementation Plan.md`). Presten's estimated time for the implementation session is 8–12 hours. DSS is May 14–15. For club rankings to be demo-ready, the implementation session must start by May 2.

Two calibration changes are in flight that directly affect what teams' ratings will look like when club rankings are computed:

1. **GA ASPIRE tier fix (April 28):** Moves GA ASPIRE events to the correct event_tier, triggering a ratings recompute. Teams that played GA ASPIRE events will see their ratings shift.
2. **U12/U13/U14/U19 calibration changes (May 18–20, pending Presten approval):** Changes K-factor and RD floor values for four age groups.

If club rankings are computed BEFORE the GA ASPIRE fix, the top club rankings in affected age groups (Girls especially) will be based on overcalibrated ratings. A club with several GA ASPIRE girls teams would rank artificially high — and then drop when the fix runs. If that happens mid-demo, or if rankings visibly shift after DSS, it undermines the accuracy claim.

ELO owns the calibration sequence. This task: assess the ordering dependency and write a clear recommendation for SENTINEL and Presten.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Rankings/Club Rankings — Calibration Pre-Assessment.md`

---

### Section 1: GA ASPIRE Dependency

**Q: Which age groups have GA ASPIRE events, and how significant is the rating impact of the fix?**

Using the GA ASPIRE Application Spec and the post-April-28 verification spec, estimate:
- Which gender/age group combinations have GA ASPIRE event coverage (likely U13G–U17G)?
- What is the expected direction and magnitude of rating change for teams that played GA ASPIRE events post-fix?
- For a club whose top U13G team played primarily GA ASPIRE events: will the club's aggregate rating increase or decrease after the fix? By how much (rough estimate)?

State whether the GA ASPIRE fix constitutes a **material rating shift** — defined as >30 points average change for top-10 teams in the affected age groups. If material: club rankings should NOT be computed until after the fix runs and ratings stabilize.

### Section 2: U12/U13/U14/U19 Calibration Dependency

**Q: Should the Calibration Production Runbook changes (K-factor, RD floor) be applied before or after the club rankings session?**

From the Calibration Production Runbook (`Rankings/Calibration Production Runbook — May 2026.md`), the proposed changes are:
- U12: RD floor 50 → 60
- U13: K-factor 32 → 24, RD floor 50 → 75
- U14: K-factor 32 → 28, RD floor 50 → 65
- U19: K-factor 32 → 24

These changes are scheduled for May 18–20 — after DSS. Club rankings implementation is scheduled for May 2 (latest). So by the timeline, club rankings would be computed before these changes apply.

Assess:
- Would the K-factor and RD floor changes cause a material shift in club rankings for the affected age groups?
- Since U12/U13/U14 are the core club ranking age groups: what is the expected direction and magnitude of change post-calibration?
- Is the May 18–20 timing a problem for DSS? The demo shows club rankings computed under pre-calibration values. After DSS, calibration runs, and rankings shift. Is this gap acceptable, or does it need to be disclosed?

State a recommendation: run calibration before or after club rankings, or accept the gap with disclosure.

### Section 3: Ordering Recommendation

Based on Sections 1 and 2, write an explicit implementation ordering recommendation:

**Recommended sequence:**
1. [Step]: [Reason]
2. [Step]: [Reason]
3. [Step]: [Reason]
...

This sequence should be concrete enough that SENTINEL can include it in a Presten-facing brief. Example:
> "Run GA ASPIRE fix and full ranking recompute (April 28). Wait 24 hours for ratings to stabilize. Run club rankings implementation session (May 2–3). Run U12/U13/U14/U19 calibration changes post-DSS (May 18–20). Club rankings will show one more recompute before DSS."

### Section 4: Demo Risk Assessment

Given the recommended sequence, what is the residual demo risk for club rankings?

- Will any top-10 clubs in Girls ranking likely shift position between now and DSS?
- Is there a known calibration issue in the Boys rankings that hasn't been addressed and could surface in club rankings?
- Is there an age group where club rankings are NOT reliable because game coverage is too thin (per FORGE's Source Gap Inventory)?

For each risk rated High: state the mitigation (show a different age group, add a disclosure, defer club rankings from the demo entirely).

## Definition of Done

- Assessment written at `Rankings/Club Rankings — Calibration Pre-Assessment.md`
- All four sections present
- GA ASPIRE dependency is assessed as Material or Not Material (not "unclear")
- Ordering recommendation is explicit: numbered sequence with named owners
- Demo risk assessment includes at least one age group flagged as risky and one flagged as safe to demo
- Summary in briefing: "Club rankings session dependency: [GA ASPIRE fix must run first / no dependency]. Recommended implementation date: [date]. Demo risk: [low/medium/high]. Flagged age groups: [list]."

## Why This Matters

The Club Rankings session is 8–12 hours of Presten's time. Running it before calibration is stable means the output will shift — possibly visibly — before or during DSS. That's the worst possible outcome: a feature that looks wrong to the exact audience we're trying to impress. Writing this assessment now ensures Presten schedules the session at the right time in the calibration sequence, not just at a convenient time.

## References

- `02 - Tiger Tournaments/Projects/Rankings/Club Rankings — Implementation Plan.md`
- `02 - Tiger Tournaments/Projects/Rankings/Calibration Production Runbook — May 2026.md`
- `02 - Tiger Tournaments/Projects/Rankings/GA ASPIRE Update SQL — 2026-04-24.md` (or equivalent)
- `02 - Tiger Tournaments/Projects/Infrastructure/Source Gap Inventory — April 2026.md` — coverage by league (affects club rankings reliability by age group)
- `10 - Agents/ELO/Tasks/task-2026-04-24-post-april28-verification-spec.md` — April 28 expected outputs
