---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-22
status: completed
priority: medium
---

# Task: Analyze MLS NEXT Two-Tier Structure and Propose Calibration Split

## Context

Recent Changes 2024-2026 documents that MLS NEXT reorganized into two formal tiers for the 2025-26 season:
- **Homegrown Division (Tier 1):** 30 MLS academies + 122 Elite clubs
- **Academy Division (Tier 2):** 220+ clubs, 15 regional divisions

The current Ranking Engine applies a **uniform 160 calibration points** to all MLS NEXT teams regardless of which division they compete in. This was appropriate when MLS NEXT was a single-tier structure. It is no longer appropriate.

The effect of uniform treatment: Academy Division teams receive the same calibration weight as MLS academies and Homegrown Elite clubs. If an Academy Division team plays and loses to an ECNL team (calibration 120), the Glicko-2 model treats this as a Tier 1 (160) team losing to a Tier 2 (120) team — a mild upset. But if the Academy team is actually competing at closer to a 130-point level, this is a more expected result, and the model is overcrediting the ECNL team and underpenalizing the Academy team. Over thousands of such games this systematic error compounds into inflated Academy Division team ratings.

Your briefing did not address this. SENTINEL is assigning it as a medium-priority task because the fix requires empirical validation before any calibration change, and the risk is real but not immediately critical.

## Required Work

### Step 1: Identify MLS NEXT Division-Level Tags in the Data
Determine whether the current `games` table has enough metadata to distinguish Homegrown Division games from Academy Division games. Look for:
- Event names that contain "Homegrown," "Academy," "Division 1," "Division 2," or equivalent identifiers
- Whether GotSport API game records include division-level metadata in the event_name or event_tier fields
- Whether the `event_tiers` table has separate entries for Homegrown vs Academy events, or if they are collapsed under a single "mlsnext" tier

Report whether the data currently supports a split analysis or whether a reclassification effort would be required first.

### Step 2: Run Cross-League Win Rate Analysis by MLS NEXT Tier
If the data supports it, replicate the cross-league win rate methodology from Cross-League Analysis but segmented by MLS NEXT division:

| Matchup | Win Rate | Sample Size |
|---------|----------|-------------|
| MLS NEXT Homegrown vs ECNL | ?% | ? |
| MLS NEXT Academy vs ECNL | ?% | ? |
| MLS NEXT Homegrown vs GA | ?% | ? |
| MLS NEXT Academy vs GA | ?% | ? |
| MLS NEXT Homegrown vs MLS NEXT Academy (direct) | ?% | ? |

A win rate of ~65-70% for Homegrown vs Academy would suggest a ~20-30 point calibration gap is appropriate (consistent with how ECNL vs ECNL RL was calibrated at 120 vs 55 based on the 55.1% win rate finding).

### Step 3: Propose a Calibration Split
Based on the win rate analysis, propose updated calibration values:

```json
// Proposed MLS NEXT tier-split calibration
{
  "mlsnext_homegrown": {
    "calibration": 160,  // unchanged if data supports it
    "notes": "30 MLS academies + 122 Elite clubs"
  },
  "mlsnext_academy": {
    "calibration": [PROPOSED VALUE based on win rate data],
    "notes": "220+ clubs, 15 regional divisions"
  }
}
```

If the data does not currently support distinguishing the two tiers reliably, your deliverable is a data gap report instead: document what additional metadata tagging would be needed before a calibration split is feasible, and propose how to get that data (e.g., event name pattern matching, or a manual classification of major MLS NEXT Academy events).

### Step 4: Assess resolveTeamTier() Impact
The ranking engine's `resolveTeamTier()` function currently handles name-based overrides for Pre-ECNL and ECNL RL teams. If the calibration split is implemented, it will need an analogous override mechanism for MLS NEXT Academy vs Homegrown teams. Assess whether this is feasible via team name patterns, event name patterns, or whether it requires a new data field in the `teams` or `event_tiers` tables.

## Deliverable

Write your analysis to `Rankings/MLS NEXT Tier Split Analysis 2026-04-22.md` with:
1. Data availability assessment (can we segment Homegrown vs Academy today?)
2. Win rate table by matchup (or data gap report if segmentation is not currently possible)
3. Proposed calibration values (or prerequisites for proposing them)
4. `resolveTeamTier()` impact assessment

This is not blocking any other task. Target completion by **2026-04-28**.

## References

- [[Ranking Engine]] — calibration Phase 8, `resolveTeamTier()` function
- [[League Hierarchy]] — current calibration values (MLS NEXT = 160 uniform)
- [[Recent Changes 2024-2026]] — MLS NEXT two-tier structure documentation
- [[Cross-League Analysis]] — win rate methodology to replicate
- [[MLS NEXT]] — league structure detail
- [[Data Sources Overview]] — MLS NEXT Source (source priority 4) vs GotSport API MLS NEXT data
