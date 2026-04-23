---
title: USA Rank Ranking Accuracy Audit — April 2026
aliases: [Ranking Accuracy Audit, USA Rank Accuracy, ELO Accuracy Audit]
tags: [rankings, audit, usa-rank, calibration, competitive-intel, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: sql-ready-pending-db-run
related: "[[USA Rank Comparison 2026-04-23]]"
---

# USA Rank Ranking Accuracy Audit — April 2026

> SENTINEL-assigned audit. Cross-checks Evo Draw rankings against USA Rank's publicly known team positions to validate methodology alignment and identify quality problems. The companion document [[USA Rank Comparison 2026-04-23]] (Presten-assigned) contains the full comparison table template and preliminary gap analysis. This audit adds formal methodology divergence analysis and prioritized recommendations.

---

## Executive Summary

Database queries were not directly executable in this session. This document delivers:

1. **All SQL lookup queries** — ready for Presten or FORGE to execute against `youth_soccer` at localhost:5432
2. **Comparison table** — pre-populated with USA Rank reference data; Evo Draw columns require the DB run
3. **Delta analysis framework** — predicts which teams will diverge and explains why
4. **Missing team investigation** — risk-ranked by source coverage gap
5. **Methodology divergence assessment** — structural differences between Glicko v7 and USA Rank's goal-difference approach
6. **Recommendations** — three specific, prioritized follow-ups

**Preliminary finding (before DB run):** Evo Draw and USA Rank should broadly agree on elite team ordering (top-50 within an age group), with increasing divergence at lower ranks. The key failure modes to test for are: (a) source coverage gaps causing teams to be missing entirely, (b) GA calibration inflation causing GA teams to rank higher than USA Rank, and (c) team dedup errors splitting game history for high-rank teams.

---

## Section 1: SQL Lookup Queries

### Query 1A: All Target Teams — Broad Fuzzy Match

```sql
-- U13 Girls target teams
SELECT
  t.team_id,
  t.team_name,
  r.rating,
  r.rank,
  r.match_count,
  r.last_game_date,
  CASE
    WHEN r.rating >= 1500 THEN 'Gold'
    WHEN r.rating >= 1350 THEN 'Silver'
    WHEN r.rating >= 1200 THEN 'Bronze'
    WHEN r.rating >= 1050 THEN 'Red'
    WHEN r.rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS band
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
WHERE r.age_group = 'U13'
  AND r.gender = 'F'
  AND r.last_game_date >= NOW() - INTERVAL '7 months'
  AND (
       t.team_name ILIKE '%Football Academy%NJ%'
    OR t.team_name ILIKE '%Football Academy%12%'
    OR t.team_name ILIKE '%FA 12G%'
    OR t.team_name ILIKE '%Wall Soccer%Elite Porto%'
    OR t.team_name ILIKE '%Wall Elite Porto%'
    OR t.team_name ILIKE '%FC Stars%G2012%'
    OR t.team_name ILIKE '%FC Stars%2012%ECNL%'
    OR t.team_name ILIKE '%NE Surf%2012%'
    OR t.team_name ILIKE '%NE Surf%G12%'
    OR t.team_name ILIKE '%Scorpions%ECNL%G12%'
    OR t.team_name ILIKE '%Scorpions%G12%'
    OR t.team_name ILIKE '%NYCFC%Girls%2012%'
    OR t.team_name ILIKE '%Dix Hills%2012%'
    OR t.team_name ILIKE '%Dix Hills%Angel City%'
    OR t.team_name ILIKE '%BVB%Pittsburgh%12G%'
    OR t.team_name ILIKE '%BVB%Pitt%12%'
    OR t.team_name ILIKE '%Old Line%Girls%12%'
    OR t.team_name ILIKE '%Old Line%12 Black%'
    OR t.team_name ILIKE '%SYC%2012%GA%'
    OR t.team_name ILIKE '%SYC%2012%Girls%'
  )
ORDER BY r.rating DESC;
```

```sql
-- U14 Girls target teams
SELECT
  t.team_id,
  t.team_name,
  r.rating,
  r.rank,
  r.match_count,
  r.last_game_date,
  CASE
    WHEN r.rating >= 1500 THEN 'Gold'
    WHEN r.rating >= 1350 THEN 'Silver'
    WHEN r.rating >= 1200 THEN 'Bronze'
    WHEN r.rating >= 1050 THEN 'Red'
    WHEN r.rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS band
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
WHERE r.age_group = 'U14'
  AND r.gender = 'F'
  AND r.last_game_date >= NOW() - INTERVAL '7 months'
  AND (
       t.team_name ILIKE '%Maryland Rush%Azzurri%'
    OR t.team_name ILIKE '%Maryland Rush%2011%'
    OR t.team_name ILIKE '%NJ Premier%G2011%'
    OR t.team_name ILIKE '%NJ Premier%2011%'
    OR t.team_name ILIKE '%Marlton%Chiefs%Premier%'
    OR t.team_name ILIKE '%Marlton%2011%'
    OR t.team_name ILIKE '%Greater Severna Park%11G%'
    OR t.team_name ILIKE '%Severna Park%2011%'
    OR t.team_name ILIKE '%Seacoast United%2011%'
    OR t.team_name ILIKE '%Seacoast%MA Elite%Navy%'
  )
ORDER BY r.rating DESC;
```

### Query 1B: Team Merge Check (for split-history detection)

```sql
-- Check if any target team name appears as a MERGED (non-canonical) record
-- If found, the canonical team may have a different name
SELECT
  t_canonical.team_name  AS canonical_name,
  t_merged.team_name     AS merged_name,
  r.rating,
  r.rank,
  r.match_count
FROM team_merges tm
JOIN teams t_canonical ON tm.canonical_team_id = t_canonical.team_id
JOIN teams t_merged    ON tm.merged_team_id    = t_merged.team_id
JOIN rankings r        ON r.team_id = tm.canonical_team_id
WHERE
  t_merged.team_name ILIKE '%Football Academy%'
  OR t_merged.team_name ILIKE '%FA 12G%'
  OR t_merged.team_name ILIKE '%Wall Soccer%'
  OR t_merged.team_name ILIKE '%FC Stars%G2012%'
  OR t_merged.team_name ILIKE '%NE Surf%G2012%'
  OR t_merged.team_name ILIKE '%Maryland Rush%Azzurri%'
  OR t_merged.team_name ILIKE '%Seacoast United%2011%'
  OR t_merged.team_name ILIKE '%SYC%2012%GA%'
ORDER BY r.rating DESC;
```

### Query 1C: Source Coverage Check (for missing teams)

Run this for any team NOT found by Query 1A — checks if their games are in the system at all:

```sql
-- Template: substitute team name fragment
SELECT
  g.source,
  COUNT(*) AS game_count,
  MIN(g.match_date) AS earliest_game,
  MAX(g.match_date) AS latest_game
FROM games g
WHERE (g.home_team_name ILIKE '%Football Academy%NJ%'
    OR g.away_team_name ILIKE '%Football Academy%NJ%')
  AND g.is_hidden = false
GROUP BY g.source
ORDER BY game_count DESC;
```

Repeat substituting each missing team name. If game_count = 0 for all sources: complete source gap. If game_count > 0 but no ranking: below the 6-game threshold or dedup issue.

### Query 1D: Ranking List Context (for correlation analysis)

```sql
-- Full active ranked list for U13G — needed to compute Evo Draw rank of teams
-- even if they don't appear in the target team fuzzy matches
SELECT
  t.team_name,
  r.rating,
  r.rank,
  r.match_count,
  CASE
    WHEN r.rating >= 1500 THEN 'Gold'
    WHEN r.rating >= 1350 THEN 'Silver'
    WHEN r.rating >= 1200 THEN 'Bronze'
    WHEN r.rating >= 1050 THEN 'Red'
    WHEN r.rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS band
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
WHERE r.age_group = 'U13'
  AND r.gender = 'F'
  AND r.last_game_date >= NOW() - INTERVAL '7 months'
ORDER BY r.rating DESC
LIMIT 100;
```

---

## Section 2: Comparison Table

> **Status:** USA Rank columns are complete. Evo Draw columns require running Section 1 queries against `youth_soccer`. When run, populate Evo Draw Rank, Rating, Delta, and Band columns. Expected time to fill in: 15–20 minutes.

**Alignment rubric:**
- ✅ Within ±2× rank (e.g., USA #12 → Evo Draw #6–24): Excellent
- 🟡 Within ±3× rank (e.g., USA #12 → Evo Draw #4–36): Good
- 🔴 More than ±3× rank: Investigate (source gap, dedup, or calibration issue)
- ⬜ Not found: Source gap — document for data roadmap

### U13 Girls

| Team | USA Rank | Evo Draw Rank | Rating | Delta | Band | Status | Notes |
|------|----------|--------------|--------|-------|------|--------|-------|
| The Football Academy NJ 2012 Girls Black | #4 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | ECNL club; top-5 USA Rank — expect Gold or high Silver in Evo Draw. Also check "FA 12G Black" merge |
| Wall Soccer Club Wall Elite Porto | #12 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | Expect top-50 Evo Draw; Silver or Bronze |
| FA 12G Black | #38 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | May be same club as Football Academy NJ — check team_merges (Query 1B). If merged, this entry will resolve to the canonical team |
| FC Stars G2012 ECNL White | #121 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | ECNL team; expect Bronze band |
| NE Surf G2012 State Navy | #198 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | "State Navy" = state association team; source coverage risk (SnapSoccer/SincSports) |
| Scorpions SC ECNL G12 | #317 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | ECNL; expect Red/Bronze |
| NYCFC Girls 2012 North | #491 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | MLS-affiliated girls program; may be in GotSport |
| Dix Hills EST Angel City G2012 | #562 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | NY state/regional club; source coverage risk |
| BVB Int'l Academy Pittsburgh 12G Prem | #613 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | BVB branding; Pennsylvania; expect Blue |
| Old Line FC Girls 12 Black | #741 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | Mid-Atlantic; expect Blue |
| SYC 2012G GA II | #826 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | **GA calibration test case.** If Evo Draw ranks noticeably higher than #826, GA calibration (140 pts) is inflating. See ga-calibration-validation-2026-04-23.md |

### U14 Girls

| Team | USA Rank | Evo Draw Rank | Rating | Delta | Band | Status | Notes |
|------|----------|--------------|--------|-------|------|--------|-------|
| Maryland Rush Azzurri 2011 | #144 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | ECNL; expect Silver/Bronze |
| NJ Premier FC G2011 | #810 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | Mid-tier; expect Blue |
| Marlton Chiefs Premier 2011 | #926 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | Regional; high source coverage risk (EDP Soccer, SincSports) |
| Greater Severna Park 11G Green | #988 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | Maryland state league; high source coverage risk |
| Seacoast United MA 2011G MA Elite Navy | #1995 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | [QUERY] | Very low USA Rank. If Evo Draw ranks this team in the top 1000, we may have insufficient data to distinguish their true skill level |

---

## Section 3: Delta Analysis

### Predicted Agreement Zones

**Zone A — Strong alignment expected (USA Rank #1–100):**
Both systems have dense game history for elite teams. Football Academy NJ (#4), Wall Soccer (#12), FC Stars ECNL (#121), Maryland Rush Azzurri (#144) all compete in ECNL and GotSport-covered events. Evo Draw should place these teams within ±2× of their USA Rank position.

**Zone B — Moderate alignment (USA Rank #100–500):**
NE Surf State Navy (#198), Scorpions ECNL (#317), NYCFC Girls (#491). Some source coverage risk at #198 for state teams. ECNL teams should still align well.

**Zone C — Expected divergence (USA Rank #500–1000):**
BVB Pittsburgh (#613), Old Line FC (#741), SYC GA (#826), NJ Premier (#810). These teams are more likely to have incomplete Evo Draw game histories. Divergence of ±5× expected and explainable.

**Zone D — Likely missing from Evo Draw (#1000+):**
Seacoast United (#1995), Greater Severna Park (#988), Marlton Chiefs (#926). Teams ranked this low in USA Rank likely have minimal GotSport presence. Expect either "not found" or "below ranking threshold" (< 6 games).

### Systematic Bias Check

After running queries, calculate:
```
avg_delta = AVG( |evo_rank - usa_rank| / usa_rank )  -- normalized delta
```

- If avg_delta < 0.3 across found teams: **no systematic bias** — methodology is sound
- If Evo Draw ranks consistently lower than USA Rank: likely **missing source data** (our game counts are lower, suppressing ratings)
- If Evo Draw ranks consistently higher than USA Rank: likely **calibration bonus inflation** (league calibration values are boosting teams beyond empirical cross-league win rates)

### Key Test Cases

| Test | Teams | What Finding Means |
|------|-------|--------------------|
| GA calibration test | SYC 2012G GA II (#826) | Evo Draw rank ≫ #826 → GA calibration inflated → awaiting `ga-calibration-validation-2026-04-23.md` |
| Top ECNL alignment | Football Academy NJ (#4), Maryland Rush (#144) | Evo Draw near these positions → ECNL calibration (130) is correct |
| Bottom-tier presence | Seacoast United (#1995), Marlton Chiefs (#926) | Not in Evo Draw → expected source gap; found with high rank → miscalibration |
| Dedup / merge integrity | FA 12G Black (#38) vs Football Academy NJ (#4) | Single merged record in Evo Draw → dedup working; two separate low-count records → merge missing |

---

## Section 4: Missing Team Investigation

For teams NOT found by Query 1A, run Query 1C on each and classify:

| Team | USA Rank | Expected Reason Missing | Source to Check |
|------|----------|------------------------|-----------------|
| Marlton Chiefs Premier 2011 | #926 | EDP Soccer events not in our data | EDP Soccer / SincSports |
| Greater Severna Park 11G Green | #988 | Maryland state league (MSYSA via SincSports) | SincSports |
| Seacoast United MA 2011G MA Elite Navy | #1995 | NH/MA state league events | SnapSoccer, USYS/NHLSA |
| Dix Hills EST Angel City G2012 | #562 | Long Island regional events | SnapSoccer |
| NE Surf G2012 State Navy | #198 | New England state cup (NEYSA) | SincSports / NEYSA |

**If Query 1C returns game_count > 0 but team not in rankings:**

```sql
-- Check if they're below the ranking threshold
SELECT COUNT(*) as game_count, COUNT(DISTINCT home_team_id) as in_home, COUNT(DISTINCT away_team_id) as in_away
FROM games
WHERE (home_team_name ILIKE '%Marlton%Chiefs%'
    OR away_team_name ILIKE '%Marlton%Chiefs%')
  AND is_hidden = false
  AND age_group = 'U14'
  AND gender = 'F';
```

If game_count < 6: they're in the system but below the ranking threshold — likely only a handful of GotSport games while most of their schedule is on SincSports/SnapSoccer.

---

## Section 5: Methodology Divergence Assessment

### USA Rank's Last-20-Games Recency Window

**Effect:** Favors teams with strong recent form. Penalizes teams that were dominant 18+ months ago but have declined.

**Who this affects most for our sample:**
- Teams in the 6th-grade age bracket (born 2012 for U13G) are in a formative year where quality changes rapidly. A team that was elite in fall 2024 but plateaued in spring 2026 will look worse in USA Rank than in Evo Draw.
- Conversely, a team that broke through mid-season 2026 will rank higher in USA Rank (recent 20 games) than in Evo Draw (full history dilutes the improvement).

**Evo Draw response:** Add a "Form Indicator" to team pages showing the last-10-game win rate and rating trend direction (↑↓→). This doesn't require changing the algorithm; it adds context for recency differences.

### Goal Difference as the Rating Signal

**USA Rank's approach:** Rating is "measured in goals." A 5–0 win moves the rating more than a 1–0 win.

**Evo Draw's approach:** Glicko v7 uses win/loss/draw (binary outcome) as the signal. Goal difference is not used.

**Who ranks differently between the two systems:**
- **High-scoring dominant teams** (win 4–0 regularly): USA Rank will rate these higher because each blowout win provides more signal. Evo Draw treats it the same as a 1–0 win.
- **Teams that win ugly** (1–0, 2–1 grind-it-out style): Evo Draw rates these based on the win; USA Rank may rank them lower because the narrow margins signal uncertainty.
- **Teams that draw frequently against strong opponents** (competitive 0–0): Evo Draw may rate these better (draws against strong teams contribute Glicko partial credit); USA Rank uses goal difference, so a 0–0 draw is a weak signal.

**Practical implication for our sample:** A team like SYC 2012G GA II (USA Rank #826) that plays in GA primarily against similarly-rated GA teams may generate low goal differences in balanced games. In Evo Draw, those same draws and narrow wins still count. This is part of why GA teams may appear inflated in Evo Draw relative to USA Rank.

### Emergent vs. Hardcoded League Calibration

**USA Rank's approach:** League quality emerges from cross-league win rates. No hardcoded bonus.

**Evo Draw's approach:** Calibration values hardcoded per league (MLS NEXT = 160, ECNL = 130, GA girls = 140, etc.). These are based on the 575K-game cross-league win rate analysis, but they are static until manually updated.

**The calibration divergence test:** If the GA calibration (140) is empirically correct, teams like SYC 2012G GA II should appear at similar positions in both systems. If GA teams systematically appear higher in Evo Draw than USA Rank, the calibration is inflated. This is the primary calibration question for this audit.

**Current status:** The GA calibration validation (`ga-calibration-validation-2026-04-23.md`) is running in parallel and will confirm whether adjustment is needed.

---

## Section 6: Recommendations

Ranked by impact on ranking accuracy, given what's known prior to running the queries:

### Recommendation 1 (High Impact): Run the SQL and complete the comparison table

Assign to FORGE or Presten. Estimated time: 15–20 minutes against `youth_soccer`. This converts the framework in this document into concrete findings. Without running the queries, we are reasoning about expected outcomes — the actual data may surprise us.

**Specific deliverable:** Populate all `[QUERY]` cells in Section 2. Add a `Status` annotation (✅/🟡/🔴/⬜) per the alignment rubric. Report the count of teams found vs. missing.

### Recommendation 2 (High Impact): Verify the Football Academy NJ / FA 12G Black merge

USA Rank shows these as two separate teams at #4 and #38 in U13G. This is almost certainly the same organization (Football Academy NJ). In Evo Draw, if these exist as two separate records with partial game histories, neither may rank as high as they should.

**Action:** Run Query 1B for "Football Academy" and "FA 12G." If two separate unmerged records are found, merge them via `team_merges` before DSS. A correctly merged top-5 team appearing in our top-20 is a stronger product story than a fragmented record appearing at #100.

**Assign to:** FORGE (database access) with ELO confirming which record should be canonical.

### Recommendation 3 (Medium Impact): Add SnapSoccer and SincSports as data sources

5 of the 16 reference teams are at high risk of being absent from Evo Draw entirely because they primarily play in SnapSoccer-managed or SincSports-managed events. These are northeast corridor youth soccer leagues that USA Rank explicitly lists as primary sources.

**Action:** FORGE to evaluate SnapSoccer and SincSports/SoccerInCollege API availability. Even a partial integration of northeast corridor state league results would close a significant portion of the top-1000 source gap.

**Priority:** After DSS (May 16). This is infrastructure work, not a quick fix. Scope it for summer 2026.

---

## Success Threshold for This Audit

This audit is **successful** if, after running the queries, we can make at least one of these concrete statements:

- **"Rankings broadly agree":** ≥ 8 of the found teams fall within ±3× of their USA Rank position. This validates Glicko v7 methodology.
- **"Specific divergence identified and explained":** Even if alignment is weak, we can name the cause (source gap, GA calibration, FA/NYCFC merge issue) and have a fix path.
- **"Top-tier teams are not broken":** No team in USA Rank's top-50 appears at Evo Draw rank #500+. That would signal a structural dedup or calibration failure requiring urgent attention before DSS.

**Failure condition:** Evo Draw rank diverges by >5× from USA Rank for the majority of ECNL-affiliated teams (Football Academy, FC Stars, Maryland Rush). That would indicate either a systematic calibration error or missing ECNL game data — neither is expected but both are actionable.

---

## Related Notes

- [[USA Rank Comparison 2026-04-23]] — companion document with full comparison table template (Presten-assigned)
- [[USA Rank Competitive Intel]] — source of all USA Rank reference data used here
- [[Ranking Engine]] — Elo/Glicko v7 methodology for comparison context
- [[Cross-League Analysis]] — 575K-game win-rate study backing calibration values
- [[League Hierarchy]] — current calibration values including GA (140) and ECNL (130)
- [[Dedup Strategy]] — team_merges context for Football Academy merge investigation
- Reports/ga-calibration-validation-2026-04-23.md — GA calibration empirical validation (parallel work)
