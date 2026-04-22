---
title: Ranking Engine
aliases: [compute-rankings.js, Rankings v7, Elo Engine, Glicko]
tags: [rankings, elo, glicko, algorithm, compute-rankings, evo-draw, critical]
created: 2026-04-21
updated: 2026-04-21
version: 7
file: compute-rankings.js
---

# Ranking Engine (v7)

The ranking engine (`compute-rankings.js`) computes national youth soccer rankings using an **Elo/Glicko hybrid** algorithm with extensive post-hoc adjustments. Version 7 introduced stronger H2H enforcement and transitive H2H — a major accuracy improvement.

---

## Core Philosophy

> [!tip] The Golden Rule
> **"If you BEAT them, you RANK above them."** Head-to-head results are the ultimate tiebreaker. The H2H enforcement phase (Phase 10) iteratively fixes any rank-order violations where a lower-ranked team has beaten a higher-ranked team.

---

## The 9+1 Phases

### Phase 1: Load Team Merges
- Loads **18,684 verified** team merge records from [[Database Schema|team_merges table]]
- Links the same team across different platforms/sources
- Total merges in DB: 27,820 (includes unverified)
- See [[Dedup Strategy]] for merge logic

### Phase 2: Load Event Tiers
- Loads tier classifications for **10,342 events**
- Maps events to [[League Hierarchy]] tiers
- Used for K-factor multipliers and calibration

### Phase 3: Elo Computation
- Processes **570K+ games** (visible only, `is_hidden=false`)
- Standard Elo with Glicko-inspired rating deviation
- K-factor varies by tier (higher tiers = more informative games)
- Games filtered by [[Scope Rules]]

### Phase 4: H2H Adjustments (3x stronger in v7)
- Direct head-to-head record comparison
- v7 tripled the weight of H2H adjustments vs v6
- Only applies when teams have played each other directly

### Phase 5: Transitive H2H (NEW in v7)
- If A beat B and B beat C, A gets a transitive boost over C
- Chain length limited to prevent noise amplification
- Major v7 addition that improved rank accuracy

### Phase 6: Common Opponent Adjustments
- Teams that share opponents get compared via relative performance
- Strengthens rankings for teams with limited direct matchups

### Phase 7: SOS Bonuses (Strength of Schedule)
- Teams facing stronger opponents get rating bonuses
- Prevents strong teams in weak divisions from being penalized

### Phase 8: League Calibration (Win-Rate Scaled)
- Applies [[League Hierarchy|calibration values]] per league tier
- Boys: [[MLS NEXT]] 160, [[ECNL]] 120, GA 100, NPL/DPL/[[ECNL RL]] 55, [[Pre-ECNL]] 25
- Girls: GA 140, [[ECNL]] 130, NPL/DPL/[[ECNL RL]] 55, [[Pre-ECNL]] 25
- Scaled by win rate within the league

### Phase 9: Division-Tier Calibration
- Applied to managed events (e.g., [[Dallas Spring Showcase]])
- Uses [[Division Calibration]] rules: Gold > Silver Elite > Silver > Bronze > Bronze II > Rec
- Adjusts ratings based on division placement

### Phase 10: H2H ENFORCEMENT (The Closer)
- **Iterative bubble-sort** fixing rank-order violations
- Scans all direct H2H matchups
- If team A beat team B but ranks lower, swaps them
- **250-point gap threshold** — won't force a swap if the Elo gap is > 250 points (prevents one upset from radically reshuffling rankings)
- Runs multiple passes until no more violations exist (or max iterations reached)

---

## Tier Overrides: resolveTeamTier()

> [!warning] Critical Function
> `resolveTeamTier()` prevents tier misclassification:
> - Teams with **"Pre-ECNL"** in name → `pre_ecnl` tier (25 cal, not 120/130)
> - Teams with **"ECNL RL"** or **"ECRL"** in name → `ecrl` tier (55 cal, not 120/130)
>
> Without this, [[Pre-ECNL]] and [[ECNL RL]] teams appearing in ECNL-classified events would receive inflated calibration. See [[League Hierarchy]] for why this matters.

---

## K-Factor Multipliers

K-factor determines how much a single game result moves a team's rating. Higher-tier games carry more weight:

| Tier             | K-factor Multiplier |
| ---------------- | ------------------- |
| Tier 1 (Elite)   | Highest             |
| Tier 2 (High Comp) | Medium            |
| Tier 3 (Competitive) | Standard        |
| Unknown/Untiered | Lowest              |

---

## Input/Output

- **Input:** `games` table filtered by `is_hidden=false` + `team_merges` + `event_tiers`
- **Output:** `rankings` table with computed Elo ratings per team
- **Run:** `node compute-rankings.js` (part of [[Data Pipeline]])
- **Duration:** Several minutes for 570K+ games

---

## Related Notes
- [[Data Pipeline]] — where rankings fit in the processing chain
- [[League Hierarchy]] — calibration values and tier definitions
- [[Division Calibration]] — DSS division-specific adjustments
- [[Cross-League Analysis]] — empirical validation of calibration
- [[Dedup Strategy]] — team merges that feed Phase 1
- [[Database Schema]] — rankings table structure
- [[Dallas Spring Showcase]] — primary managed event using these rankings
