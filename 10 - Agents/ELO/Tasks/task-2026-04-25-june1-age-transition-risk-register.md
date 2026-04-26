---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-25
priority: medium
due: 2026-05-07
deliverable: "02 - Tiger Tournaments/Projects/Rankings/June 1 Age Transition — Risk Register.md"
---

# Task: June 1 Age Transition — Risk Register

## Context

The June 1 school-year age group transition is the largest structural change Evo Draw will undergo in 2026. Teams shift age groups based on birth year to school year mapping. The transition affects team identity (which age group a team competes in), historical game attribution (do past games follow the team or stay with the old age group?), and rating continuity (does a team's rating carry forward, reset, or blend?).

ELO has filed various transition-adjacent documents across the April sprint:
- Age group transition June 1 risk (`task-2026-04-25-age-group-transition-june1-risk.md`, completed)
- June 2 ECNL migration verification spec (`task-2026-04-24-june2-ecnl-migration-verification-spec.md`, completed)
- June 1 transition Presten brief (`task-2026-04-24-june1-transition-presten-brief.md`, completed)

What does not exist: a single risk register. The individual documents address specific aspects of the transition. The risk register answers: what are all known risks, what is the probability and impact of each, what is the mitigation plan, and what is the residual risk?

A risk register serves two functions SENTINEL cares about:

1. **DSS preparation**: If Presten makes commitments at DSS (May 16) about age group handling, those commitments must be consistent with the transition risks. A DSS audience asking "how do you handle age group transitions?" needs an honest answer. If ELO hasn't systematically catalogued the risks, the answer may be incomplete.

2. **June 1 readiness**: SENTINEL needs a single reference to judge whether the transition is ready to execute on June 1 vs. needs delay. Without a risk register, that judgment requires synthesizing 5+ documents.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/June 1 Age Transition — Risk Register.md`

---

### Section 1: Transition Summary

One paragraph per transition type:

**What changes June 1:**
- Birth year → school year mapping for all teams (affected age groups, affected team count estimate)
- ECNL migration begins June 2 (ecnl_verified column, teams moving to new event structure)
- Rating carryforward rule: what is ELO's policy for each transition type?

**What does NOT change June 1:**
- Historical game records remain attributed to their original age group context
- Calibration values remain in effect until next full calibration session (unless a specific calibration fix is triggered by transition)

---

### Section 2: Risk Register

ELO catalogs all known risks. For each risk:

| # | Risk | Category | Probability | Impact | Mitigation | Residual Risk | Status |
|---|------|----------|-------------|--------|------------|--------------|--------|
| R1 | Teams assigned to wrong age group post-transition due to ambiguous birth year edge cases | Data quality | [Low/Med/High] | [Low/Med/High] | [ELO mitigation] | [ELO assessment] | Open |
| R2 | Rating discontinuity: team rated U14 all season suddenly competes as U15 with no rating bridge | Rating accuracy | | | | | Open |
| R3 | Cross-league win-rate data becomes invalid: win rates computed across age groups that no longer exist post-transition | Algorithm | | | | | Open |
| R4 | ECNL migration (June 2) conflicts with or compounds the June 1 transition processing | Data quality | | | | | Open |
| R5 | High-volume teams with many June 1 games (tournaments) are re-rated mid-tournament | Data quality | | | | | Open |
| R6 | `ecnl_verified` backfill (April 28 → May pipeline) is incomplete when June 2 migration runs | Dependency | | | | | Open |
| R7 | [ELO identifies additional risks from prior transition documents] | | | | | | Open |

**Probability scale:** Low (< 20%), Medium (20–60%), High (> 60%)
**Impact scale:** Low (cosmetic, no SENTINEL action needed), Medium (requires ELO fix within 1 week), High (requires immediate action, may affect DSS demo claims)

---

### Section 3: Pre-Transition Readiness Checklist

What must be confirmed before June 1?

| Condition | Owner | Deadline | Status |
|-----------|-------|----------|--------|
| Age group mapping table verified for all affected teams | ELO/Presten | May 24 | Pending |
| Rating carryforward rule documented and tested | ELO | May 24 | Pending |
| ecnl_verified backfill ≥ 90% complete before June 2 | FORGE | June 1 | Pending (May 1 pipeline will begin populating) |
| June 2 ECNL migration verification spec reviewed by SENTINEL | SENTINEL | May 30 | Pending |
| Cross-league win-rate recalibration scheduled post-transition | ELO | June 7 | Pending |
| [Additional conditions ELO identifies] | | | |

---

### Section 4: DSS Implications

If Presten is asked about age group transitions at DSS on May 16:

**Safe to say:** [ELO writes what is definitively true and defensible — e.g., "transitions are handled with rating carryforward at [X]% of the team's prior rating" or "historical games are retained under original age group context"]

**Do not commit to:** [ELO writes what is not yet resolved — e.g., "exact handling of teams with games in both old and new age group during the transition week" — areas where a specific commitment could be incorrect]

**If asked about ECNL migration specifically:** [ELO drafts a one-sentence DSS-appropriate response]

---

### Section 5: SENTINEL Monitoring Triggers

Conditions that would prompt SENTINEL to delay June 1 transition:

- Any High-probability + High-impact risk without a confirmed mitigation (as of June 1 readiness check)
- ecnl_verified backfill < [N]% complete (FORGE confirms threshold)
- June 2 ECNL migration verification spec not reviewed by SENTINEL by May 30

Conditions that allow June 1 to proceed with monitoring:
- All Medium risks have confirmed mitigations
- No High-impact risks are Open as of May 31 SENTINEL review

---

## Quality Standard

- Every risk in Section 2 must have a Probability and Impact score — not left blank.
- Section 4 "safe to say" language must be specific and ELO-authorized — not "we handle transitions carefully."
- Section 3 deadlines must be specific dates, not relative ones.
- The register must synthesize information from the prior transition documents (June 1 risk task, June 2 ECNL migration spec, Presten brief). Do not repeat those documents — reference them and extract the risks they identified.

## Why This Matters

The June 1 transition is larger in scope than the April 28 calibration session, but it has less preparation infrastructure. April 28 has an Execution Log, a Reference Card, a Pre-April-28 Baseline Snapshot package, and a pipeline validation spec. June 1 currently has three separate documents none of which is a risk register. If something goes wrong in June, SENTINEL will need to diagnose quickly — and a risk register is the fastest way to identify which risk materialized and what the pre-planned mitigation is. Writing it now, in May, ensures the mitigations are thought through before June 1, not improvised after.

## References

- `Rankings/Age Group Transition — June 1 Risk.md` — prior ELO risk assessment to synthesize
- `Rankings/June 1 Transition — Presten Brief.md` — Presten-facing context document
- `Rankings/June 2 ECNL Migration Verification Spec.md` — June 2 dependency this register tracks
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — the ecnl_verified dependency this register tracks
