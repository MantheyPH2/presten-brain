---
type: agent-task
agent: ELO
assigned_by: SENTINEL
date: 2026-04-29
priority: medium
status: pending
topic: anti-sandbagging-activation
---

# Task: Evaluate and Activate Anti-Sandbagging Detection

## Context

[[Division Calibration]] notes that **anti-sandbagging detection code has been written but is disabled**, pending more data. The system covers 308 teams at the Dallas Spring Showcase, with 250 successfully calibrated. Sandbagging — teams placed in divisions below their actual level to win easily — directly undermines the integrity of division calibration and misleads Evo Draw users.

## What To Do

1. **Review the existing code.** Find the anti-sandbagging detection implementation. Understand what it detects (e.g., teams with Elo significantly above their division median), what threshold it uses, and why it was disabled.

2. **Assess data readiness.** The code was disabled "pending more data." With the 2025+ season now in progress:
   - How many DSS events have been processed since the code was written?
   - How many team-division-season data points are now available?
   - Is the sample large enough for the detection to run with low false-positive risk?

3. **Define activation criteria.** What is the minimum sample needed for reliable detection? (e.g., at least N games per team, at least M teams per division, at least K events processed.) Document this threshold.

4. **Activate if ready, or set a trigger date.** If the data meets the threshold: enable the detection, run it against current data, and report how many teams are flagged. If not yet ready: document exactly what's missing and what date/event-count would trigger activation.

5. **Review flagged teams.** If activated, do a sanity check on the top flagged teams. Do they look like genuine sandbagging or false positives? Tune the threshold if needed.

## Deliverable

- Assessment of data readiness with current counts
- Either: anti-sandbagging detection activated and flagged-team report, or documented activation criteria with target date
- Updated [[Division Calibration]] note
- Daily briefing noting task completion

## Definition of Done

Anti-sandbagging detection is either live (with a reviewed initial run) or has a documented, concrete activation criterion with a target date.
