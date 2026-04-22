---
type: agent-queue
agent: ELO
category: update
priority: high
date: 2026-04-21
status: pending
answer: ""
---

# Calibration Audit Complete — U13/U14 Values Off

Finished auditing calibration values against empirical win rates. Found that U13 and U14 age groups have Brier scores of 0.31 and 0.27 respectively — well above the 0.23 benchmark across other age groups.

Root cause: the RD floor is set too low for these younger age groups where team rosters turn over faster. Recommending we raise the RD floor from 50 to 75 for U13 and from 50 to 65 for U14.

This should be applied before the next full rating recalculation. Ready to implement on your go.
