---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-25
priority: high
due: 2026-05-07
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Club Rankings — Girls-Only Fallback Spec.md"
---

# Task: Club Rankings — Girls-Only Fallback Spec

## Context

The Club Rankings implementation plan covers both Boys and Girls. The DSS Feature Readiness Tracker flags the Boys cell as conditional: Boys club rankings require Boys Option A spot check to pass (run window: April 29 – May 5). If Boys Option A fails, the Club Rankings demo becomes Girls-only.

What does not exist: a Girls-only fallback spec. The full Club Rankings implementation plan assumes both genders. If Boys fails on May 9, ELO and FORGE will need to implement a narrowed version under time pressure (7 days before DSS). That is the wrong time to discover that the implementation plan needs scoping changes.

This task: specify the Girls-only Club Rankings now, while there is no time pressure. If Boys Option A passes, this spec is insurance that never gets used. If Boys Option A fails, this spec is what ELO hands to FORGE immediately after May 9 go/no-go.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Club Rankings — Girls-Only Fallback Spec.md`

---

### Section 1: Scope Delta from Full Implementation

Starting from `Rankings/Club Rankings — Implementation Specification.md`, document every change that applies when Boys are excluded:

- **Queries to remove or modify:** Any query that aggregates across Boys + Girls or uses gender as a parameter
- **Schema changes (if any):** Does the club_rankings table/view need a WHERE clause, or is gender already a filter parameter?
- **Calibration concerns specific to Girls-only scope:** Are there any Girls age groups where ELO has low confidence in club-level aggregation? (e.g., if a Girls age group has low game volume per club, the club average may be noisy)
- **Demo narrative change:** What does SENTINEL tell Presten to say instead of "Boys and Girls club rankings"? One sentence.

---

### Section 2: Girls Club Rankings Readiness Assessment

Using the same framework as the DSS Feature Readiness Tracker:

| Girls Age Group | Game Volume Sufficient? | Calibration Validated? | Club Aggregation Stable? | Notes |
|-----------------|------------------------|------------------------|--------------------------|-------|
| U13G | | | | |
| U14G | | | | |
| U15G | | | | |
| U16G | | | | |
| U17G | | | | |

For "Club Aggregation Stable?": does each Girls age group have enough teams per club to produce a meaningful club average? Minimum threshold: a club ranking based on < 3 teams per club is not stable. Define the threshold ELO considers acceptable.

---

### Section 3: Girls-Only Session Time Estimate

The full Club Rankings session is estimated at 10–11 hours (per the implementation spec). What is the Girls-only session estimate?

Break down:
- Hours saved by excluding Boys queries
- Hours added by any Girls-specific validation steps
- New total estimate

This sets Presten's expectation for a May 9 → May 16 compressed timeline if Boys fails.

---

### Section 4: Reference Clubs for Girls-Only Demo

Pull from `Rankings/Club Rankings — Reference Clubs.md` (filed 2026-04-25). If Boys fail, which reference clubs remain in the demo? Are there any clubs where the Girls rankings alone would tell a compelling story for the DSS audience?

List: top 5 Girls clubs by expected ranking, with a one-line description of why each is a strong demo example.

---

### Section 5: Trigger and Handoff Protocol

Define the exact conditions that activate this spec:

- **Trigger:** Boys Option A spot check returns Query 1 FAIL (Boys vs. Girls divergence > 100 pts for any age group)
- **ELO action:** File updated DSS Feature Readiness Tracker with Boys cell = ❌ and note "Girls-only fallback spec activated — see `Rankings/Club Rankings — Girls-Only Fallback Spec.md`"
- **FORGE action:** FORGE receives scope delta from Section 1 of this spec and adjusts implementation plan
- **Timeline:** If trigger fires by May 5 and Girls-only spec is ready, FORGE can still complete Girls-only implementation before May 9 go/no-go

---

## Definition of Done

- File written at `Rankings/Club Rankings — Girls-Only Fallback Spec.md`
- All 5 sections present
- Scope delta from full implementation is explicit (not "TBD" or "refer to full spec")
- Girls-only session estimate is a specific number, not a range
- Reference clubs section names at least 5 clubs
- Trigger and handoff protocol is a precise conditional: "If X, then ELO does Y, FORGE does Z"
- Summary in briefing: "Girls-only fallback spec written. Girls club rankings: [N/5] age groups ready. Session estimate: [N] hours. Top reference clubs: [list]."

## Why This Matters

The DSS demo is May 16. If Boys Option A fails on May 5, SENTINEL has 11 days to either fix Boys calibration (unlikely on that timeline) or complete Girls-only Club Rankings. Without a Girls-only spec, those 11 days start with a design session instead of an execution session. With this spec ready, FORGE can start implementation on May 5 if the trigger fires. The spec is low-cost to write now and high-value if the trigger ever fires.

## References

- `Rankings/Club Rankings — Implementation Specification.md` — full scope this spec narrows
- `Rankings/Club Rankings — Reference Clubs.md` — reference clubs (Boys + Girls)
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — the check that triggers this spec
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Boys cell this spec is fallback for
