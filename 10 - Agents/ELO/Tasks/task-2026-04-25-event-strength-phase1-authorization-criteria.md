---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-25
status: completed
completed: 2026-04-25
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — SENTINEL Authorization Criteria.md"
---

# Task: Event Strength Phase 1 — SENTINEL Authorization Criteria

## Context

Event Strength Phase 1 assigns per-event quality ratings based on the finalized league hierarchy calibration values. The Phase 1 execution package is filed. The April 28 Execution Log will track the DB changes. But what triggers SENTINEL to say "Event Strength Phase 1 is authorized"? That decision is currently described as: "SENTINEL reviews execution log + April 29 analysis and decides." That is a judgment call, not a defined gate.

Event Strength Phase 1 is a meaningful ranking change. It affects how games weight into ratings based on the quality of the event, not just the league tier. If Phase 1 runs before Girls calibration is stable (post-April-28), it introduces a second major calibration variable into an already-changing system, making it impossible to attribute subsequent rating shifts to either the GA ASPIRE fix or the Event Strength weighting.

ELO must define: what conditions must hold for Phase 1 to be safe to run? SENTINEL enforces those conditions. Without ELO's definition, SENTINEL either approves too early (compounding instability) or holds indefinitely (slipping the DSS timeline).

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Event Strength Phase 1 — SENTINEL Authorization Criteria.md`

---

### Section 1: What Phase 1 Changes

One paragraph: what exactly does Event Strength Phase 1 modify in the ranking computation? Specifically:
- Does it change the K-factor per game, the event-level multiplier, or the tier classification?
- Which rating fields does it affect? (Girls only, Boys only, both?)
- Is Phase 1 reversible without a full recompute?

SENTINEL needs this summary to judge risk. If Phase 1 is Girls-only and reversible, the authorization threshold is lower than if it affects both genders and requires a full recompute to undo.

---

### Section 2: Authorization Gate Conditions

Define each condition ELO considers necessary for Phase 1 to run safely. Structure as a table:

| # | Condition | Source of Confirmation | Pass Threshold | Current Status |
|---|-----------|----------------------|----------------|----------------|
| 1 | April 28 Execution Log: all 3 steps ✅ | `Infrastructure/April 28 Execution Log.md` SENTINEL Disposition = AUTHORIZED | All 3 steps ✅ | PENDING |
| 2 | April 29 G2 gate PASS | April 29 Results Filing Template — G2 Status | PASS | PENDING |
| 3 | Calibration stability: ≥ [N] consecutive pipeline runs without volatility flag | Post-Fix Calibration Stability Monitoring Spec — stability criteria | Stability cleared (Level 0 for N runs) | PENDING |
| 4 | Boys Option A not yet in fail state (Phase 1 should not compound a Boys investigation) | Boys Option A results OR no Boys anomaly in April 29 analysis | Boys shift ≤ 5 pts in April 29 analysis | PENDING |
| 5 | [Add any additional ELO conditions] | | | |

ELO fills in the specific pass thresholds for Conditions 3+ from the Calibration Stability Monitoring Spec. Do not use "stable enough" — use exact numbers.

---

### Section 3: Authorization Window

When can Phase 1 run?

**Earliest possible date:** [ELO defines — likely April 30 or May 1, assuming G2 PASS and stable]

**Latest acceptable date (DSS constraint):** Phase 1 must complete, ratings must recompute, and ELO must validate Phase 1 output before May 9 go/no-go. FORGE must also have ECNL dedup verification complete before Phase 1 (since ECNL dedup affects the same game records Phase 1 weights). ELO works backward from May 9 to state: Phase 1 must be authorized no later than [date].

**Dead window:** ELO identifies any periods where Phase 1 should not run:
- While Boys Option A spot-check results are being analyzed (to avoid confounding Phase 1 with Boys calibration investigation)
- During any active YELLOW or RED pipeline state (instability compounds)
- Within [N] hours of a full recompute (to allow monitoring queries to establish a stable baseline first)

---

### Section 4: ELO Pre-Phase-1 Checklist

When all Section 2 conditions are met, ELO files a Phase 1 clearance note containing:

```
Phase 1 Authorization Clearance — ELO
Date: ___________
Conditions 1–[N]: all confirmed ✅
Calibration stability confirmed as of: ___________
Stability monitoring: [N] consecutive clean runs
April 29 G2 gate: PASS (confirmed on: ___________)
Boys status: no anomaly detected
Expected Phase 1 effects: [ELO's brief prediction — which age groups shift, direction, magnitude]
SENTINEL authorization request: Phase 1 authorized to run from [date] onward.
```

ELO does not file this clearance note until all conditions are met. SENTINEL does not authorize Phase 1 until ELO files the clearance note.

---

### Section 5: SENTINEL Authorization Block

SENTINEL fills this section after receiving ELO's clearance note:

```
Phase 1 authorization reviewed: [ ]
ELO clearance note date: ___________
All conditions confirmed ✅: [ ]
Event Strength Phase 1: AUTHORIZED / HELD
Authorization date: ___________
If HELD, reason: ___________
Authorized window: [date range] — Phase 1 must run before [latest date]
```

---

### Section 6: Post-Phase-1 Validation Requirement

After Phase 1 runs, ELO must file a post-Phase-1 validation note within [N] hours. Minimum contents:

- Game count affected by Phase 1 event strength weighting
- Average rating shift for affected age groups (Girls, Boys separately)
- Any anomalous shifts (teams that moved > [X] points)
- Confirmation that Phase 1 did not affect age groups it was not designed to affect

SENTINEL reviews this note before authorizing Phase 2 (if applicable).

---

## Quality Standard

- All pass thresholds must be explicit numbers. "Stable" is not a threshold.
- The authorization window dates must be specific. "After April 29" is not specific enough.
- The post-Phase-1 validation requirement must define the [N] hours window and the [X] points anomaly threshold.
- This document is the source of truth for Phase 1 authorization. SENTINEL will reference it, not ELO's briefings, to confirm all conditions are met.

## Why This Matters

Event Strength Phase 1 is one of two major calibration changes remaining before DSS (the other is Boys calibration, pending Option A). Authorizing it without defined criteria means SENTINEL decides by judgment when conditions are "good enough." Authorizing it with explicit criteria means SENTINEL confirms a checklist. The checklist is faster, more reliable, and creates an audit trail. More importantly: if Phase 1 were to run during an instability window (say, during a Boys Option A investigation), the post-Phase-1 analysis becomes impossible to interpret — was the rating shift from Phase 1, from Boys recalibration, or from new games? Explicit authorization gates prevent that confounding.

## References

- `Rankings/Event Strength — Phase 1 Execution Package.md` — Phase 1 scope
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — stability criteria for Condition 3
- `Rankings/April 29 Analysis — Results Filing Template.md` — G2 gate source for Condition 2
- `Infrastructure/April 28 Execution Log.md` — Condition 1 source
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — Condition 4 source
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Phase 1 feature cell this authorization unlocks
