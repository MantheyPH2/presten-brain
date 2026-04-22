---
title: ECNL RL
aliases: [ECNL RL, ECNL Regional League, ECRL]
tags: [league, ecnl-rl, ecrl, tier-2, high-competitive, evo-draw]
created: 2026-04-21
updated: 2026-04-21
tier: 2
calibration: 55
---

# ECNL RL (Regional League)

ECNL RL is the **regional feeder league** to [[ECNL]], sitting at **Tier 2 — High Competitive** in the [[League Hierarchy]]. It provides a pathway for clubs to earn promotion into the elite ECNL platform.

---

## Structure

| Attribute     | Value                  |
| ------------- | ---------------------- |
| Total clubs   | ~700                   |
| Girls clubs   | ~300                   |
| Boys clubs    | ~400                   |
| Boys leagues  | 24                     |
| Girls leagues | 23                     |
| Tier          | 2 — High Competitive   |
| Calibration   | **55 points**          |

---

## Promotion to ECNL

- **24 clubs promoted** from ECNL RL to [[ECNL]] for the 2025-26 season
- Massive expansion underway — see [[Recent Changes 2024-2026]]
- Performance in ECNL RL is the primary pathway to ECNL membership

---

## Calibration and Tier Override

> [!warning] Critical: Per-Team Tier Override
> ECNL RL shares the same calibration (55 points) as NPL and DPL. However, ECNL RL games sometimes appear in events classified as "ECNL" in the data. Without intervention, these teams would receive ECNL-level calibration (120/130 instead of 55).
>
> The `resolveTeamTier()` function in [[Ranking Engine]] handles this:
> - Teams with **"ECNL RL"** in their name → capped at `ecrl` tier
> - Teams with **"ECRL"** in their name → capped at `ecrl` tier
>
> **This override is essential for ranking accuracy.**

---

## Cross-League Performance

From [[Cross-League Analysis]] (575K games):
- ECNL RL beats NPL **55.1%** of the time
- This confirms ECNL RL's placement as a **Tier 2 peer** alongside NPL
- The win rate is decisive enough to validate the tier, but not so dominant as to warrant a higher calibration

---

## Related Notes
- [[ECNL]] — the elite league ECNL RL feeds into
- [[League Hierarchy]] — full tier system
- [[Ranking Engine]] — `resolveTeamTier()` override logic
- [[Cross-League Analysis]] — empirical win-rate validation
- [[Pre-ECNL]] — U10-U12 developmental (different from both ECNL and ECNL RL)
- [[Recent Changes 2024-2026]] — expansion details
