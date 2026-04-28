---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: medium
due: 2026-05-05
deliverable: "02 - Tiger Tournaments/Projects/Rankings/June 1–2 Sequencing Risk Register.md"
---

# Task: June 1–2 Sequencing Risk Register

## Context

ELO's 16:21 session surfaced a sequencing dependency SENTINEL had not previously tracked: **the June 1 age group transition migration must complete before the June 2 ECNL platform migration (TGS → GotSport).** If June 1 slips, June 2 must slip accordingly. The acceptable slip window is June 7–10 per the existing spec.

This dependency is now visible but not formally documented. It exists in the June 1 procedure document as an assertion, not in a risk register with slip scenarios, impact analysis, and escalation criteria.

The risk is real: the Boys Brier analysis window runs May 17–30. If it surfaces a calibration revision that takes more than a few days to implement, the June 1 transition date could slip. If June 1 slips past June 10 (the acceptable slip window), the ECNL migration must also be re-planned. If both slip, the July 1 season start — which depends on the new team IDs being live — is at risk.

SENTINEL is flagging this to Presten now. ELO must produce the risk register so SENTINEL has a document to reference when making escalation decisions.

---

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/June 1–2 Sequencing Risk Register.md`

---

### Section 1: Dependency Map

Diagram (text-based) of the full dependency chain:

```
Boys Brier verdict (by May 30)
        |
        v
June 1 age group transition (Presten, ~3 hrs)
        |
        v
June 2 ECNL platform migration (TGS → GotSport)
        |
        v
July 1 season start (new team IDs live)
```

For each step, state: who owns it, estimated time, latest acceptable date, and what event triggers it.

---

### Section 2: Slip Scenario Analysis

Analyze three slip scenarios for June 1:

#### Scenario A: June 1 → June 7 (1-week slip)

- **Cause:** Brier results require a targeted calibration adjustment; 3–5 day implementation window pushes June 1 task to June 7.
- **Impact on June 2 ECNL migration:** June 2 becomes June 8. Is June 8 within the acceptable ECNL migration window?
- **Impact on July 1 season start:** [ELO states — does 1 week slip create a risk for July 1?]
- **SENTINEL action:** [What should SENTINEL do if June 1 is on track for June 7?]

#### Scenario B: June 1 → June 10 (1.5-week slip — edge of acceptable window)

- **Cause:** Brier results require a more significant calibration revision; 7–8 day implementation + validation window.
- **Impact on June 2 ECNL migration:** June 2 becomes June 11. Is this within the acceptable window?
- **Impact on July 1 season start:** [ELO states — is this still safe?]
- **SENTINEL action:** [What should SENTINEL do if June 1 looks like it will slip to June 10?]

#### Scenario C: June 1 → June 14+ (slip beyond acceptable window)

- **Cause:** Brier results reveal a structural calibration problem requiring a full recalibration cycle.
- **Impact on June 2 ECNL migration:** June 2 cannot maintain its current date. New date must be negotiated.
- **Impact on July 1 season start:** [ELO states — is July 1 at risk?]
- **SENTINEL action:** [This is an escalation scenario. What does SENTINEL tell Presten?]

---

### Section 3: Acceptable Window Definition

ELO pulls the acceptable slip window from the existing spec and states it explicitly:

- Latest acceptable June 1 date: [date from spec or ELO's analysis]
- Latest acceptable June 2 date: [date]
- Why these are the limits: [reasoning from the July 1 dependency — how much processing/validation time does the ECNL migration need before July 1?]

If ELO cannot determine the July 1 deadline reasoning from existing vault documents, flag it as a question for Presten in Section 5.

---

### Section 4: Escalation Criteria

Define the conditions under which SENTINEL escalates to Presten proactively:

| Condition | SENTINEL Action | Timing |
|-----------|----------------|--------|
| Boys Brier verdict indicates > 5-day implementation window | Alert Presten — June 1 slip likely | Same session as verdict |
| June 1 execution confirmed delayed by > 3 days | Alert Presten — June 2 must be rescheduled | As soon as delay is confirmed |
| Slip exceeds acceptable window | Escalate — July 1 season start at risk | Immediately |
| June 1 execution started but not completed in one session | Alert Presten — confirm time to completion | Next session after June 1 begins |

---

### Section 5: Open Questions for Presten

If ELO cannot answer any of the following from existing vault documents, list them:

1. What is the minimum processing/validation time between June 2 ECNL migration completion and July 1 season start? (This sets the hard outer limit on the slip window.)
2. Is the June 1 age group transition a hard-coded annual schedule, or can it be executed on any date within a window?
3. If the June 1 → June 2 dependency breaks (both slip to late June), is there any option to run them in the same session?

ELO should attempt to answer these from the vault before posting as open questions.

---

## Quality Standard

- All three slip scenarios must have specific impact statements — not "possible risk to July 1" but "July 1 is at risk if June 2 does not complete by [date]."
- Section 4 escalation criteria must be triggerable by SENTINEL without additional deliberation — SENTINEL reads the criteria, identifies the condition, takes the action.
- Acceptable window in Section 3 must be sourced from existing vault documents or explicitly flagged as "ELO's estimate — requires Presten confirmation."

## Why This Matters

SENTINEL discovered this dependency two days before it becomes time-critical (Boys Brier results arrive May 17–30; June 1 is only 2–3 weeks later). If SENTINEL waits to formalize the risk register until June 1 is actually slipping, there will be no time to manage the downstream effects on ECNL migration and July 1. With this document in place before May 17, SENTINEL can monitor the Brier results specifically for their June 1 slip implications and escalate to Presten with a specific scenario analysis rather than a vague warning.

## References

- `Rankings/June1-Age-Transition-Procedure-2026-04-27.md` — source of June 1 / June 2 ordering dependency assertion
- `Rankings/ECNL Migration — Option B Technical Risk Deep Dive.md` — June 1 simultaneous-change risk analysis
- `Rankings/Post-DSS Sprint — Week 1 Execution Calendar.md` — Boys Brier window (May 17–30)
- `Rankings/ECNL Rating Continuity Spec.md` or equivalent — ECNL migration timing requirements
