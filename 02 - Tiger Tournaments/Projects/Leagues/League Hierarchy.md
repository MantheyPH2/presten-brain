---
title: League Hierarchy
aliases: [Tier System, League Tiers, Competitive Pathway]
tags: [leagues, hierarchy, tiers, calibration, evo-draw, critical]
created: 2026-04-21
updated: 2026-04-21
---

# League Hierarchy

> [!warning] CRITICAL REFERENCE
> This is the authoritative tier hierarchy for US youth soccer. The [[Ranking Engine]] uses these tiers for calibration. **Misclassifying a league tier will corrupt rankings.**

---

## Tier 1 — Elite

### [[MLS NEXT]]
- **Boys #1 league** in the US
- 273 clubs: 30 MLS academies + 243 Elite clubs
- Age groups: U13-U19
- Divisions: Homegrown (Tier 1) + Academy (Tier 2)
- **Calibration: 160 points**

### [[ECNL]]
- **#2 boys / #1 girls (tied with GA)**
- ~130 girls clubs, ~150 boys clubs
- Boys: 15 conferences | Girls: 10 conferences
- **Calibration: 120 boys, 130 girls**

### GA (Girls Academy)
- **#1 girls (tied with ECNL)**
- 120+ clubs
- ASPIRE tier launched Feb 2025
- **Calibration: 140 girls, 100 boys**

---

## Tier 2 — High Competitive

### [[ECNL RL]] (Regional League)
- ~700 clubs (300 girls + 400 boys)
- Feeder to [[ECNL]] — 24 clubs promoted for 2025-26
- 24 boys leagues, 23 girls leagues
- **Calibration: 55 points**

### NPL (National Premier League)
- Merit-based promotion system
- Integrating with USYS National League for 2026-27
- **Calibration: 55 points**

### DPL (Development Players League)
- Girls-focused, GA pathway
- **Calibration: 55 points**

### USL Academy
- 90 clubs with professional pathway
- Connected to USL professional system

### Others at Tier 2
- Elite 64
- EDP (Eastern Development Program)
- NAL (North American League)

---

## Tier 3 — Competitive

### [[Pre-ECNL]]
- U10-U12 developmental program
- 300+ clubs
- **Calibration: 25 points**

> [!warning] Pre-ECNL IS NOT ECNL
> Despite the name, Pre-ECNL is a Tier 3 developmental program for younger age groups. It is NOT the same competitive level as [[ECNL]]. The `resolveTeamTier()` function in [[Ranking Engine]] enforces this distinction.

### State Leagues and Cups
- Organized by state associations under [[Governing Bodies|USYS]]
- Highly variable quality

### ODP (Olympic Development Program)
- Talent identification pipeline
- State → Regional → National selection

---

## Competitive Pathway

```
Recreational → Select → Competitive → High Competitive → Elite → Professional
   (Tier 3)              (Tier 3)        (Tier 2)         (Tier 1)
```

---

## Calibration Values (used in [[Ranking Engine]])

### Boys
| League        | Calibration Points |
| ------------- | ------------------ |
| [[MLS NEXT]]  | 160                |
| [[ECNL]]      | 120                |
| GA             | 100                |
| NPL / DPL / [[ECNL RL]] | 55      |
| [[Pre-ECNL]]  | 25                 |

### Girls
| League        | Calibration Points |
| ------------- | ------------------ |
| GA             | 140                |
| [[ECNL]]      | 130                |
| NPL / DPL / [[ECNL RL]] | 55      |
| [[Pre-ECNL]]  | 25                 |

---

## Per-Team Tier Overrides

> [!warning] Critical Classification Rule
> The `resolveTeamTier()` function in [[Ranking Engine]] applies per-team overrides to prevent misclassification:
> - Teams with **"ECNL RL"** or **"ECRL"** in their name → capped at `ecrl` tier, even if the event is classified as `ecnl`
> - Teams with **"Pre-ECNL"** in their name → capped at `pre_ecnl` tier
> - Without these overrides, [[ECNL RL]] teams playing in ECNL-classified events would get ECNL-level calibration (120/130 instead of 55)

---

## Cross-League Validation

From our [[Cross-League Analysis]] of 575K games:
- [[ECNL RL]] beats NPL **55.1%** → confirms Tier 2 peer status
- [[Pre-ECNL]] beats NPL **68.4%** → surprisingly strong for Tier 3
- State Cup teams beat NPL **51.4%** → very competitive
- [[MLS NEXT]] dominates all tiers

---

## Related Notes
- [[Ranking Engine]] — how calibration values are applied
- [[Cross-League Analysis]] — empirical win-rate data
- [[Recent Changes 2024-2026]] — structural shifts in the landscape
- [[MLS NEXT]], [[ECNL]], [[ECNL RL]], [[Pre-ECNL]] — individual league details
