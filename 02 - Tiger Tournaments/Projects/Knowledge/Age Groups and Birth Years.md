---
title: Age Groups and Birth Years
aliases: [Age Groups, Birth Years, U-Groups, Format Rules]
tags: [knowledge, age-groups, birth-years, formats, evo-draw]
created: 2026-04-21
updated: 2026-04-21
---

# Age Groups and Birth Years

---

## 2025-26 Season Mapping

| Age Group | Birth Year | Format          |
| --------- | ---------- | --------------- |
| U7        | 2019       | 4v4             |
| U8        | 2018       | 4v4             |
| U9        | 2017       | 7v7             |
| U10       | 2016       | 7v7             |
| U11       | 2015       | 9v9             |
| U12       | 2014       | 9v9             |
| U13       | 2013       | 11v11           |
| U14       | 2012       | 11v11           |
| U15       | 2011       | 11v11           |
| U16       | 2010       | 11v11           |
| U17       | 2009       | 11v11           |
| U18       | 2008       | 11v11           |
| U19       | 2007       | 11v11           |

> [!info] Format Progression
> - **4v4:** U7-U8 (youngest, small-sided)
> - **7v7:** U9-U10 (transitional)
> - **9v9:** U11-U12 (intermediate)
> - **11v11:** U13-U19 (full field)
>
> These formats affect game data scope — see [[Scope Rules]].

---

## Birth Year to Age Group Conversion

Formula: `Age Group = Current Year - Birth Year + 1`

Example for 2026: Birth year 2011 → 2026 - 2011 + 1 = 16 → **U16**

> [!warning] Wait — the table shows 2011 as U15
> The mapping depends on the **season year**, not calendar year. For the 2025-26 season, the base year is 2025, so: 2025 - 2011 + 1 = 15 → **U15**. The [[Parsing Rules]] use this season-aware calculation.

---

## 2026-27 Age Group Shift

> [!warning] Major Change Coming
> Starting 2026-27, most leagues shift from **birth year** to **school year** (Aug 1 - Jul 31) age group cutoffs. This affects:
> - [[ECNL]] — shifting to school year
> - [[ECNL RL]] — shifting to school year
> - NPL, DPL, and most others — shifting to school year
>
> **Exception:** [[MLS NEXT]] stays on birth year cutoff.
>
> This split creates complexity for the [[Ranking Engine]] — teams in the same age group bracket may have different age eligibility depending on their league.

---

## Parsing Formats

See [[Parsing Rules]] for full priority order. Quick reference:

| Format       | Example    | Parsed As |
| ------------ | ---------- | --------- |
| Standard     | U15        | U15       |
| Hyphenated   | U-15       | U15       |
| Reversed     | 15U        | U15       |
| Compact      | 15UB       | U15 Boys  |
| Spanish      | Sub 15     | U15       |
| Under        | Under 15   | U15       |
| Birth year   | B2011      | U15 (in 2025-26) |
| Multi-age    | U13/U14    | U13       |

---

## Related Notes
- [[Parsing Rules]] — full parsing priority and logic
- [[Scope Rules]] — U7-U19 only, format constraints
- [[League Hierarchy]] — which leagues use which age groups
- [[MLS NEXT]] — exception to the 2026-27 shift
- [[Recent Changes 2024-2026]] — age group shift details
- [[TGS AthleteOne]] — uses birth year division format
