---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: urgent
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — CP1 Fail Escalation Protocol.md"
---

# Task: ECNL Migration — CP1 Fail Escalation Protocol

## Context

CP1 (ECNL RL staleness check) runs as Step 1 of the April 29 ELO Session Master Script. It is the highest-stakes checkpoint in the entire ECNL migration decision. ELO flagged it explicitly in the April 26 briefing:

> "If ECNL RL > 30 days stale, Option 1 is disqualified immediately, and the June 1 migration plan shifts to Option 2 — which requires Presten sign-off on the rating cliff. That escalation path needs to be processed before the May 10 option selection deadline."

What does not exist: the protocol for what happens when CP1 fails. The Option Comparison Matrix (filed April 26) records the disqualification mechanically. The April 29 Master Script says "flag to SENTINEL." Neither document describes what ELO, SENTINEL, and Presten actually do from the moment CP1 returns a FAIL result.

This document must exist before April 29 so that if CP1 fails, the escalation runs from a defined playbook rather than an improvised response under time pressure.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — CP1 Fail Escalation Protocol.md`

---

### Section 1: CP1 Fail Definition

State precisely what constitutes a CP1 FAIL:

- **FAIL:** ECNL Regional League teams have `last_game_date` > 30 days before the query date — meaning ECNL RL data is stale by more than 30 days in the system
- **PASS:** ECNL RL data is ≤ 30 days stale — Option 1 (No Re-tag) remains in contention

Include the exact query from the Option Comparison Matrix or Master Script that produces this result. This section is not a description — it is the operative definition. ELO should be able to read Section 1, run one query, and know immediately whether CP1 PASSED or FAILED with no ambiguity.

---

### Section 2: Immediate Actions (Within the April 29 Session)

If CP1 FAILS during the April 29 session, ELO takes these steps **before continuing with any other April 29 work**:

1. **Record the result.** Fill in CP1 row of `Rankings/ECNL Migration — Option Comparison Matrix.md` Running Verdict Tracker: CP1 = FAIL. Option 1 status = DISQUALIFIED. Date = April 29.

2. **File a same-session Queue item.** Write to `10 - Agents/ELO/Queue/pending-2026-04-29-ecnl-cp1-fail.md`:
   - Category: alert
   - Priority: urgent
   - Content: "CP1 FAIL confirmed. ECNL RL staleness > 30 days. Option 1 disqualified. Migration decision now between Option 2 (Re-tag + rating cliff) and Option 3 (ecnl_legacy). Presten sign-off required on Option 2 rating cliff before May 10 option selection deadline. Request: Presten session to review Option 2 implications before May 5."

3. **Update the April 29 briefing.** Note CP1 FAIL in the briefing flags section: "ECNL CP1 FAIL — Option 1 disqualified. Migration on Option 2 / 3 track. See Queue item pending-2026-04-29-ecnl-cp1-fail."

4. **Continue with remaining April 29 steps.** CP1 FAIL does not halt the entire session. Steps 2–6 (calibration baseline, Decision Tree G1–G4, Girls Calibration Narrative, Prediction vs. Actual, CP2) proceed normally. Only Steps 7–8 conditional work adjusts based on CP1 result.

---

### Section 3: Option 2 Implications (ELO's Pre-Written Analysis)

Write this section now, before April 29, so it exists when CP1 FAIL is confirmed.

Option 2 (Re-tag) means:
- ECNL Regional League teams get re-tagged with the new ECNL RL league code on June 1
- All historical ECNL RL games re-compute under the new tier/calibration value
- Teams that transition from "ECNL" (old tag) to "ECNL RL" (new tag) may experience a rating cliff at the transition point

ELO must state:
1. **Estimated rating cliff magnitude:** Based on the calibration difference between the old ECNL tag and the new ECNL RL tag, what is the expected rating shift for teams transitioning? Provide a range (e.g., "estimated 20–60 pts downward for ECNL RL teams, depending on historical game volume under old tag").
2. **Which teams are most exposed:** Are there age groups or team types (high game volume, ECNL RL primary vs. occasional) that face a larger cliff?
3. **What Presten needs to decide:** Option 2 is viable but creates a visible jump in historical rankings. Presten's sign-off should acknowledge: (a) the cliff is expected and acceptable, or (b) Option 3 (ecnl_legacy) is preferable to avoid the cliff. ELO's recommendation: [state ELO's current lean and why].

---

### Section 4: Option 3 Comparison

State what Option 3 (ecnl_legacy) means vs. Option 2, and what would make ELO recommend Option 3 over Option 2:

- **Option 3:** Tag all historical ECNL RL games as `ecnl_legacy` — they keep their old calibration value permanently. No rating cliff, but legacy games are computed differently than current ECNL RL games going forward. Creates a permanent two-track system for ECNL RL historical data.
- **When to prefer Option 3:** If the Option 2 rating cliff exceeds [N] points for more than [X]% of affected teams, the distortion is large enough that the two-track system is preferable to the cliff.
- **ELO's current lean:** [State preference and condition for switching]

---

### Section 5: May 10 Deadline Implications

If CP1 FAILS on April 29:
- May 10 is the option selection deadline (13 days from April 29 session)
- Presten sign-off on Option 2 or Option 3 must occur before May 10
- ELO needs Presten session confirmation by **May 5** (5 days before deadline) to have time to implement the selected option's migration spec before June 1
- If no Presten sign-off by May 5: ELO defaults to Option 3 (ecnl_legacy) as the safer non-cliff option and files this in the Queue as a decision-by-default

State this timeline explicitly with calendar dates.

---

## Definition of Done

- Protocol written at `Rankings/ECNL Migration — CP1 Fail Escalation Protocol.md`
- All 5 sections present
- CP1 FAIL definition includes the operative query (copy-paste ready)
- Option 2 rating cliff magnitude is a specific estimate, not "TBD"
- ELO's current lean between Option 2 and Option 3 is stated with a condition for switching
- May 10 deadline implications include calendar dates

## Why This Matters

This document must exist before CP1 runs. Writing the escalation protocol after CP1 FAILS requires ELO to design it under time pressure while Presten is waiting for the next step. Writing it now means that if CP1 fails, ELO takes 4 pre-defined actions, files 2 documents, and moves on — total time cost: 10 minutes. Without the protocol, CP1 FAIL becomes a session-stopping decision exercise that consumes the 130–170 minute April 29 session budget.

## References

- `Rankings/ECNL Migration — Option Comparison Matrix.md` — Running Verdict Tracker to update
- `Rankings/April 29 ELO Session Master Script.md` — Step 1 context
- `Rankings/ECNL Migration — Rating Continuity Spec.md` — Option 2/3 details
- `task-2026-04-26-ecnl-migration-option-comparison-matrix.md` — Matrix this protocol feeds into
