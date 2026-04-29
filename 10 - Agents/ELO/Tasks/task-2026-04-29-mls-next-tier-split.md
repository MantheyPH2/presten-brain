---
type: agent-task
agent: ELO
assigned_by: SENTINEL
date: 2026-04-29
priority: urgent
status: pending
topic: mls-next-tier-split
---

# Task: Implement MLS NEXT Tier Split (Homegrown vs Academy)

## Context

[[League Hierarchy]] documents a **pending MLS NEXT tier split** tied to the May 18–20 event. Currently all MLS NEXT teams share a single calibration value (160). The split will differentiate:
- **Homegrown programs:** calibration 160 (top tier)
- **Academy programs:** calibration 135 (below ECNL Girls at 130/120)

This is a material ranking change that affects all teams competing against or compared to MLS NEXT. It must be ready before the May 18–20 event processes through the pipeline.

## What To Do

1. **Identify the distinguishing data.** How do we know if a team is Homegrown vs Academy? Review [[MLS NEXT]] and the data sources. Is there a league name suffix, division field, or GotSport tag that separates them?

2. **Update the tier assignment logic.** In the ranking engine (Phase 2: event tier assignment), update the MLS NEXT league handling to route teams to the correct calibration bucket based on the Homegrown/Academy distinction.

3. **Update [[League Hierarchy]].** Replace the single 160 entry with two entries: Homegrown (160) and Academy (135). Remove the "pending" annotation.

4. **Validate with historical data.** Re-run rankings for a sample of MLS NEXT teams under the new split. Do the Homegrown teams rank as expected vs ECNL? Do Academy teams land in the right zone between ECNL and Tier 2?

5. **Document the change.** Add an entry to [[Recent Changes 2024-2026]] describing the split, the calibration values, and the effective date.

## Deliverable

- Updated tier assignment code (document file path)
- Updated [[League Hierarchy]] and [[Recent Changes 2024-2026]]
- Validation summary (sample rankings before/after)
- Daily briefing noting task completion

## Definition of Done

Ranking engine correctly routes MLS NEXT Homegrown to 160 and Academy to 135, validated against sample data, ready before May 18.
