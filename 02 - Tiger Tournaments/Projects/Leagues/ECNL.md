---
title: ECNL
aliases: [ECNL, Elite Clubs National League]
tags: [league, ecnl, tier-1, elite, boys, girls, evo-draw]
created: 2026-04-21
updated: 2026-04-21
tier: 1
calibration_boys: 120
calibration_girls: 130
---

# ECNL (Elite Clubs National League)

ECNL is the **#2 boys league** and **#1 girls league (tied with GA)** in US youth soccer. It is a national competition platform for elite youth clubs, operated by [[Governing Bodies|US Club Soccer]].

---

## Structure

### Boys
- **~150 clubs** across **15 conferences**
- Calibration: **120 points** (see [[Ranking Engine]])
- Second only to [[MLS NEXT]] (160) for boys

### Girls
- **~130 clubs** across **10 conferences**
- Calibration: **130 points** (see [[Ranking Engine]])
- Tied with GA (140) as top girls competition

---

## Feeder System

- [[ECNL RL]] (Regional League) serves as the direct feeder
- **24 clubs promoted** from ECNL RL for the 2025-26 season
- See [[ECNL RL]] for details on the promotion pathway

> [!warning] ECNL vs ECNL RL — Different Tiers
> [[ECNL RL]] is Tier 2 (calibration 55), NOT Tier 1. The [[Ranking Engine]] uses `resolveTeamTier()` to enforce this distinction. Teams with "ECNL RL" or "ECRL" in their name are capped at the `ecrl` tier. See [[League Hierarchy]].

---

## Upcoming Changes

### Age Group Shift (2026-27)
Moving from **birth year** to **school year** (Aug 1 - Jul 31) age groups starting 2026-27. See [[Age Groups and Birth Years]] and [[Recent Changes 2024-2026]].

### Platform Migration (June 2026)

> [!warning] Critical Migration
> ECNL is migrating from [[TGS AthleteOne]] to [[GotSport]] in **June 2026**. This has significant implications:
> - All new ECNL game data will flow through [[GotSport]] (source priority 1) instead of TGS (priority 3)
> - Historical TGS data remains in the database
> - The [[TGS Scraper]] may become obsolete for ECNL
> - [[GotSport API Scraper]] will automatically capture new ECNL games as teams get GotSport IDs
> - Watch for [[Dedup Strategy]] edge cases during the transition

---

## Data Coverage

Current ECNL data primarily comes from:
1. [[TGS AthleteOne]] — league play (source priority 3)
2. [[GotSport]] — ECNL tournaments and showcases (source priority 1-2)

After June 2026, all data will flow through [[GotSport]].

---

## Related Notes
- [[League Hierarchy]] — tier system and calibration values
- [[ECNL RL]] — regional feeder league
- [[Pre-ECNL]] — U10-U12 developmental (NOT the same level)
- [[TGS AthleteOne]] — current data source
- [[GotSport]] — future data source (post-June 2026)
- [[Ranking Engine]] — calibration application
- [[Recent Changes 2024-2026]] — platform migration details
- [[Tournament Landscape]] — ECNL showcases and national events
