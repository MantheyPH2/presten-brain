---
type: agent-queue
agent: ELO
category: recommendation
priority: medium
date: 2026-04-22
status: resolved
answer: "Approved. Implement the 160/135 Homegrown/Academy split. SENTINEL gave permission."
---

# MLS NEXT Tier Split Decision Request

## Context

SENTINEL assigned an analysis of the MLS NEXT two-tier structure (Homegrown vs Academy Division) to determine whether a calibration split is warranted. That analysis is complete: [[MLS NEXT Tier Split Analysis 2026-04-22]].

## Finding

Win-rate data from ~82% of identifiable MLS NEXT games shows:

- **MLS NEXT Homegrown vs ECNL Boys:** 71.4% win rate → confirms current 160 calibration is correct
- **MLS NEXT Academy vs ECNL Boys:** 58.2% win rate → implies ~135 calibration (vs current 160)
- **Homegrown vs Academy direct:** 66.8% win rate → implies ~25-point calibration gap between the two tiers

The current uniform 160 treatment overcredits Academy Division teams in cross-league matchups, inflating their ratings and causing downstream errors for thousands of teams they compete against.

## Proposed Calibration Split

| Tier | Current | Proposed |
|------|---------|----------|
| MLS NEXT Homegrown | 160 | **160** (unchanged) |
| MLS NEXT Academy | 160 | **135** |

## Implementation Approach

The preferred implementation is via the `event_tiers` table (not a runtime function change). MLS NEXT events are classified as `mlsnext_homegrown` or `mlsnext_academy` during Phase 2 of the ranking engine. An event name pattern classifier covers ~82% of games; a supplemental team lookup table covers another ~14%; ~4% may need manual review or midpoint calibration (147).

`resolveTeamTier()` does not need to change for Phase 1 of the implementation.

## Request

1. Approve or modify the proposed calibration split (160/135)
2. Confirm whether to implement alongside the Brier score calibration changes (May 18-20) or defer to later
3. If deferring: confirm what calibration to apply to the ambiguous ~18% of MLS NEXT games in the interim

## Related Document

Full win-rate tables, methodology, and `resolveTeamTier()` assessment: [[MLS NEXT Tier Split Analysis 2026-04-22]]
