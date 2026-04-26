---
title: USA Rank Reference Data — Sourcing Spec
type: data-sourcing-spec
author: ELO
date: 2026-04-26
status: ready
due: 2026-04-30
related: "[[USA Rank Post-April-28 Comparison — Execution Package]]", "[[USA Rank Delta Interpretation Spec — April 2026]]", "[[DSS Ranking Accuracy Claims — Authorized Spec]]"
tags: [usa-rank, comparison, data-sourcing, girls, dss, validation, evo-draw]
---

# USA Rank Reference Data — Sourcing Spec

> **Purpose:** Define exactly what USA Rank data ELO needs, how Presten collects it, and in what format to provide it — so the May 3 comparison session begins with data in hand, not with a 30–60 minute collection detour.
>
> **Context:** `Rankings/USA Rank Post-April-28 Comparison — Execution Package.md` has a standing open item: "SENTINEL provides USA Rank reference data." That data is not in the vault. This spec closes the gap: Presten collects it before May 3 using the method and template below.

---

## Section 1: Data Requirements

### Which age groups

All five Girls age groups covered by the execution package:

- U13G (2012 birth year cohort)
- U14G (2011)
- U15G (2010)
- U16G (2009)
- U17G (2008)

Boys age groups are out of scope for this comparison. April 28 was a Girls-only fix; Boys gap with USA Rank is pre-existing and not part of the April 29 analysis.

### Which teams

ELO uses an **anchor-on-Evo-Draw** approach (Option C — see Section 2). Presten does not browse USA Rank broadly. Instead, ELO identifies the top-10 Evo Draw Girls teams per age group after the April 28 recompute, and Presten looks up those specific teams in USA Rank.

**Why 10 teams per age group:** The execution package's comparison works at the cohort-average level, not team-by-team. 10 teams per age group gives a meaningful sample for computing the USA Rank top-10 average — enough to detect a systematic gap, not so many that collection becomes burdensome.

**Total data points to collect:** 10 teams × 5 age groups = 50 team lookups. Estimated collection time: 25–35 minutes.

### What data fields per team

| Field | Required? | Notes |
|-------|-----------|-------|
| USA Rank ranking position (rank number) | **Required** | Primary comparison field |
| USA Rank rating score | Optional | Only if USA Rank displays a numeric score alongside the rank. Many pages only show rank — that is sufficient. |
| Date USA Rank data was captured | **Required** | Add to template header. Must be April 28 – May 3 to be contemporaneous with post-recompute Evo Draw data. |
| Season displayed on USA Rank page | **Required** | Confirm it is the 2025–2026 season, not a prior season. |

---

## Section 2: Sourcing Options

### Option A: Vault Note Search

**Status: No actionable USA Rank data found in vault.**

The `Knowledge/Competitors.md` note covers USA Rank only structurally (it is named as a competitor) with no ranking data. Prior comparison work (`Rankings/USA Rank Comparison 2026-04-23.md`) contains reference team entries for a small number of teams:

| Team | USA Rank | Age Group | Date captured |
|------|----------|-----------|---------------|
| Football Academy NJ 2012 Girls Black | #4 | U13G | ~April 2026 |
| FC Stars G2012 ECNL White | #121 | U13G | ~April 2026 |
| SYC 2012G GA II | #826 | U13G | ~April 2026 |
| Maryland Rush Azzurri 2011 | #144 | U14G | ~April 2026 |

This is insufficient for the comparison: it covers only U13G and U14G partially, does not cover U15G–U17G, and the date is not confirmed post-April-28. Option A cannot satisfy the requirement alone.

### Option B: USA Rank Website (direct browse)

Presten navigates to USA Rank and browses each age group's ranking page. This produces full data but requires finding the right pages and manual entry for each team. Estimated time: 45–60 minutes for 50 team lookups across 5 age groups.

Not recommended as primary because it is slower than Option C and risks collecting data for teams ELO doesn't need.

### Option C: Anchor on Evo Draw First — ELO Recommended

**ELO recommends Option C.**

Presten runs Evo Draw's top-10 Girls teams per age group first (Query 1 from the execution package, or a simplified version below). That list becomes the lookup list for USA Rank. Presten only searches USA Rank for those specific 50 teams — not broad page browsing.

**Advantage:** Focused lookup, minimal over-collection, produces exactly the crossover teams the comparison needs.

**Prerequisite:** Query 1 from the execution package must be run first (requires post-April-28 recompute state). If recompute is confirmed April 29, this can run April 29 or 30.

**Evo Draw top-10 pull query:**

```sql
-- Evo Draw top 10 Girls teams per age group (post-April-28)
-- Run this first, then use team names to look up USA Rank
SELECT
  age_group,
  team_name,
  rating AS evo_draw_rating,
  rank   AS evo_draw_rank
FROM rankings r
JOIN teams t ON t.team_id = r.team_id
WHERE t.gender = 'F'
  AND t.age_group IN ('U13', 'U14', 'U15', 'U16', 'U17')
  AND r.rank <= 10
  AND r.last_game_date >= NOW() - INTERVAL '7 months'
ORDER BY t.age_group, r.rank;
```

Save the output as a reference list. Then search USA Rank for each team name by age group.

**How to look up on USA Rank:**
1. Navigate to the USA Rank Girls rankings for each age group (use the age group filter on the USA Rank search/rankings page)
2. Search the team name (or club name fragment) in the rankings page search
3. Record the team's USA Rank position and score (if shown)
4. If team does not appear in USA Rank, enter "not listed" in the template

---

## Section 3: Input Template

Pre-built template for Presten to fill after collecting USA Rank data. File this at `Rankings/Reports/usarank-reference-data-[date].md` and send to ELO.

```
---
USA Rank Reference Data — Captured [date]
Source: USA Rank website, captured by: Presten
Evo Draw query run: [date of top-10 query]
USA Rank season confirmed: 2025-2026 [ ]
---

Age group: U13G

| Team Name (Evo Draw) | Evo Draw Rating | Evo Draw Rank | USA Rank Position | USA Rank Score (if shown) | Notes |
|---------------------|-----------------|---------------|-------------------|--------------------------|-------|
| [from Evo Draw query] | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |

Age group: U14G

| Team Name (Evo Draw) | Evo Draw Rating | Evo Draw Rank | USA Rank Position | USA Rank Score (if shown) | Notes |
|---------------------|-----------------|---------------|-------------------|--------------------------|-------|
| [from Evo Draw query] | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |

Age group: U15G

| Team Name (Evo Draw) | Evo Draw Rating | Evo Draw Rank | USA Rank Position | USA Rank Score (if shown) | Notes |
|---------------------|-----------------|---------------|-------------------|--------------------------|-------|
| [from Evo Draw query] | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |

Age group: U16G

| Team Name (Evo Draw) | Evo Draw Rating | Evo Draw Rank | USA Rank Position | USA Rank Score (if shown) | Notes |
|---------------------|-----------------|---------------|-------------------|--------------------------|-------|
| [from Evo Draw query] | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |

Age group: U17G

| Team Name (Evo Draw) | Evo Draw Rating | Evo Draw Rank | USA Rank Position | USA Rank Score (if shown) | Notes |
|---------------------|-----------------|---------------|-------------------|--------------------------|-------|
| [from Evo Draw query] | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |
| | | | | | |

Teams not found in USA Rank (list any from above marked "not listed"):
- [team name] — [age group]
```

---

## Section 4: Comparison Validity Notes

### Rating vs. rank

If USA Rank only shows rankings (not scores), the comparison is Evo Draw rating vs. USA Rank rank — a rank-to-rating cross-check, not a score-to-score comparison.

**ELO assessment: A rank-only comparison is sufficient for the DSS narrative claim.** The claim is "our rankings correlate with USA Rank on the teams we share." Showing that Evo Draw's top-10 Girls teams by rating map to USA Rank positions in the top 50 (or top 100) is meaningful correlation evidence, even without score-to-score matching. The claim does not require ELO and USA Rank to produce identical numbers — it requires that our high-rated teams are also recognized as strong by the external benchmark.

### Staleness

USA Rank data captured April 28 – May 3 is contemporaneous with the post-April-28 recompute. This is acceptable.

**Older USA Rank data (pre-April-28) is not acceptable for this comparison.** The comparison's purpose is to confirm the GA ASPIRE fix moved Evo Draw Girls rankings in the right direction relative to USA Rank. Using pre-fix USA Rank data would compare post-fix Evo Draw against a USA Rank snapshot that was not contemporaneous — the comparison would conflate the effect of the fix with natural USA Rank drift. Capture fresh data.

### Coverage gap caveat and minimum overlap requirement

USA Rank leans GotSport-adjacent. Teams that compete exclusively in ECNL (which is not a GotSport-primary league) may not appear in USA Rank. Some Evo Draw top-10 teams per age group may be absent from USA Rank.

**Minimum valid overlap: 6 teams per age group.** If fewer than 6 of the 10 Evo Draw top-10 teams appear in USA Rank for a given age group, the comparison for that age group is not statistically meaningful and ELO will not include it in the DSS narrative claim. If 3 or more age groups fall below 6-team overlap, the entire comparison claim should be weakened or dropped from the DSS script.

If overlap is consistently low (< 6 per age group across multiple age groups), the interpretation is that Evo Draw and USA Rank rank different team populations — which is itself a claim about coverage breadth. ELO will note this in the comparison verdict.

---

## Section 5: Briefing Summary for SENTINEL

To run the USA Rank comparison before May 3, Presten needs to collect USA Rank positions for up to 10 teams × 5 age groups (50 team lookups), estimated 25–35 minutes at the USA Rank Girls rankings search pages. Recommended method: run Evo Draw's top-10 Girls per age group first using the query in Section 2, then look up those specific team names in USA Rank. Data entry template is at `Rankings/USA Rank Reference Data — Sourcing Spec.md` Section 3; once filled, save to `Rankings/Reports/usarank-reference-data-[date].md` and send to ELO. ELO returns comparison results within the same session or within 24 hours of receiving the completed template.

---

## References

- `Rankings/USA Rank Post-April-28 Comparison — Execution Package.md` — the execution package this spec unblocks
- `Rankings/USA Rank Delta Interpretation Spec — April 2026.md` — ELO's framework for interpreting comparison results
- `Knowledge/Competitors.md` — checked; no usable USA Rank data found
- `Rankings/USA Rank Comparison 2026-04-23.md` — prior comparison work; partial reference team list for U13G/U14G only
- `Product & Planning/DSS Demo Spec — May 2026.md` — the coverage claim this comparison supports

*ELO — 2026-04-26*
