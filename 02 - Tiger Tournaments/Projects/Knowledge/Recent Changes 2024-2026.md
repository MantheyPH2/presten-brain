---
title: Recent Changes 2024-2026
aliases: [Industry Changes, Landscape Shifts, What's New]
tags: [knowledge, changes, timeline, industry, evo-draw]
created: 2026-04-21
updated: 2026-04-21
---

# Recent Changes 2024-2026

Major structural changes in the US youth soccer landscape that affect Evo Draw's data, rankings, and strategy.

---

## MLS NEXT-GA Strategic Alliance (December 2024)

[[MLS NEXT]] and GA (Girls Academy) announced a strategic alliance:
- Closer integration between the two organizations
- Potential for shared events and cross-league play
- Strengthens both as top-tier competitors to [[ECNL]]
- Impact on [[Ranking Engine]]: may need to adjust GA calibration as alliance evolves

---

## MLS NEXT Two-Tier Structure (2025-26)

[[MLS NEXT]] reorganized into two formal tiers:
- **Homegrown Division (Tier 1):** 30 MLS academies + 122 Elite clubs
- **Academy Division (Tier 2):** 220+ clubs, 15 regional divisions
- Impact: Previously a single tier, now needs differentiated calibration consideration
- Current [[Ranking Engine]] calibration: 160 points for all MLS NEXT (may need splitting)

---

## GA ASPIRE Launch (February 2025)

Girls Academy launched the **ASPIRE** tier:
- New entry-level tier below the main GA competition
- Broadens GA's reach to more clubs
- Similar concept to [[ECNL RL]] for [[ECNL]]
- Impact on [[League Hierarchy]]: adds another tier to track

---

## Age Group Shift (2026-27 Season)

> [!warning] Major Ecosystem Change
> Most leagues shifting from **birth year** to **school year** (Aug 1 - Jul 31) age group cutoffs:
> - **Shifting:** [[ECNL]], [[ECNL RL]], NPL, DPL, most state leagues
> - **NOT shifting:** [[MLS NEXT]] (stays on birth year)
>
> **Impact on Evo Draw:**
> - [[Parsing Rules]] need updating for dual-system age groups
> - [[Ranking Engine]] must handle teams in the same age bracket with different eligibility
> - Cross-league comparisons become more complex
> - See [[Age Groups and Birth Years]] for the mapping table

---

## ECNL RL Massive Expansion

[[ECNL RL]] has grown dramatically:
- Now ~700 clubs (300 girls + 400 boys)
- 24 boys leagues, 23 girls leagues
- **24 clubs promoted** to [[ECNL]] for 2025-26
- Becoming the dominant Tier 2 pathway
- Impact: Significantly more data flowing through the system

---

## ECNL Platform Migration (June 2026)

> [!warning] Critical Data Source Change
> [[ECNL]] migrating from [[TGS AthleteOne]] to [[GotSport]] in **June 2026**:
> - All new ECNL game data will flow through [[GotSport]] (source priority 1)
> - [[TGS Scraper]] may become obsolete for ECNL
> - [[GotSport API Scraper]] will automatically pick up ECNL games
> - Watch for [[Dedup Strategy]] edge cases during transition
> - Historical TGS data remains in [[Database Schema|database]]

---

## NCAA D1 Scholarship Expansion

Men's soccer D1 scholarships expanded to **28**:
- Previously limited to a lower number
- Slightly improves odds for male players (still only 0.9% of HS boys make D1)
- Increases the value of [[College Recruiting|recruiting data]] and visibility
- Reinforces Evo Draw's value proposition for families

---

## Timeline Summary

| When         | What                                    | Impact Level |
| ------------ | --------------------------------------- | ------------ |
| Dec 2024     | MLS NEXT-GA Strategic Alliance          | Medium       |
| Feb 2025     | GA ASPIRE Launch                        | Low-Medium   |
| 2025-26      | MLS NEXT Two-Tier Structure             | Medium       |
| 2025-26      | ECNL RL Massive Expansion              | Medium       |
| 2025-26      | NCAA D1 Scholarship Expansion           | Low          |
| **June 2026**| **ECNL Platform Migration**             | **High**     |
| **2026-27**  | **Age Group Shift (birth→school year)** | **High**     |

---

## Related Notes
- [[League Hierarchy]] — tier structure affected by all these changes
- [[MLS NEXT]] — two-tier structure and birth year exception
- [[ECNL]] — platform migration and age group shift
- [[ECNL RL]] — expansion details
- [[TGS AthleteOne]] — becoming obsolete for ECNL
- [[GotSport]] — gaining ECNL data
- [[Age Groups and Birth Years]] — shift details
- [[College Recruiting]] — scholarship expansion
- [[Ranking Engine]] — calibration may need updates
