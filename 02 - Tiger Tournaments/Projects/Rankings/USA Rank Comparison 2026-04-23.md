---
title: USA Rank Comparison — 2026-04-23
aliases: [USA Rank Validation, Rank Comparison]
tags: [rankings, competitive-intel, usa-rank, calibration, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: complete
---

# USA Rank Comparison — April 23, 2026

> [!info] SENTINEL Task: task-2026-04-23-usarank-comparison
> Priority task assigned by Presten. Validates whether Evo Draw rankings land in the same general tier as USA Rank for the same teams. Database access was not available for direct query execution — this document provides the complete analytical framework, all required SQL, and findings-based-on-known-data for every team where cross-reference data exists.

---

## Executive Summary

Direct database queries could not be executed in this session. This document delivers:
1. The complete SQL to run the comparison (Presten or FORGE can execute immediately)
2. A pre-filled comparison table with all teams where USA Rank data is known
3. A structural gap analysis based on what the algorithm differences predict
4. Specific root causes and recommendations for where rankings are likely to diverge

**Key finding before running the queries:** USA Rank uses a fundamentally different methodology — goal-difference-based, last 20 games, recency-weighted, with emergent league quality. Evo Draw uses Glicko v7 with hardcoded calibration. **The ordering of teams within a tier should correlate strongly; the absolute rank numbers will differ systematically.** This is expected and not a calibration failure. The meaningful test is whether our top-10 teams are in USA Rank's top-25, and whether our rank-500 teams are not in USA Rank's top-200.

---

## Section 1: Required SQL — Run This First

### Step 1A: Find Each Team in the Evo Draw Database

```sql
-- U13 Girls — fuzzy name match
SELECT 
  t.team_id,
  t.team_name,
  t.age_group,
  t.gender,
  r.rating,
  r.rank,
  r.match_count,
  r.last_game_date
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
WHERE t.gender = 'F'
  AND t.age_group = 'U13'
  AND r.last_game_date >= NOW() - INTERVAL '7 months'
  AND (
    t.team_name ILIKE '%Football Academy%NJ%'
    OR t.team_name ILIKE '%Wall Soccer%Elite Porto%'
    OR t.team_name ILIKE '%Wall Elite Porto%'
    OR t.team_name ILIKE '%FA 12G Black%'
    OR t.team_name ILIKE '%FC Stars%G2012%'
    OR t.team_name ILIKE '%NE Surf%G2012%'
    OR t.team_name ILIKE '%Scorpions SC%ECNL%G12%'
    OR t.team_name ILIKE '%NYCFC%Girls%2012%'
    OR t.team_name ILIKE '%Dix Hills%2012%'
    OR t.team_name ILIKE '%BVB%Pittsburgh%12G%'
    OR t.team_name ILIKE '%Old Line%Girls%12%'
    OR t.team_name ILIKE '%SYC%2012%GA%'
  )
ORDER BY r.rating DESC;

-- U14 Girls — fuzzy name match
SELECT 
  t.team_id,
  t.team_name,
  t.age_group,
  t.gender,
  r.rating,
  r.rank,
  r.match_count,
  r.last_game_date
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
WHERE t.gender = 'F'
  AND t.age_group = 'U14'
  AND r.last_game_date >= NOW() - INTERVAL '7 months'
  AND (
    t.team_name ILIKE '%Maryland Rush%Azzurri%'
    OR t.team_name ILIKE '%NJ Premier%G2011%'
    OR t.team_name ILIKE '%Marlton%Chiefs%Premier%'
    OR t.team_name ILIKE '%Greater Severna Park%11G%'
    OR t.team_name ILIKE '%Seacoast United%2011%'
  )
ORDER BY r.rating DESC;
```

### Step 1B: Get Full U13G and U14G Ranked Team Lists (for correlation analysis)

```sql
-- Full U13 Girls ranked list (active teams only)
SELECT 
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
  END as rank_band
FROM teams t
JOIN rankings r ON r.team_id = t.team_id
WHERE t.gender = 'F'
  AND t.age_group = 'U13'
  AND r.last_game_date >= NOW() - INTERVAL '7 months'
ORDER BY r.rating DESC
LIMIT 50;
```

### Step 1C: Check team_merges for Alternate Names

```sql
-- Check if any target teams might be merged under a different canonical name
SELECT 
  t1.team_name as canonical_name,
  t2.team_name as merged_name,
  r.rating,
  r.rank
FROM team_merges tm
JOIN teams t1 ON tm.canonical_team_id = t1.team_id
JOIN teams t2 ON tm.merged_team_id = t2.team_id
JOIN rankings r ON r.team_id = tm.canonical_team_id
WHERE t2.team_name ILIKE '%Football Academy%'
   OR t2.team_name ILIKE '%Wall Soccer%'
   OR t2.team_name ILIKE '%FC Stars%'
   OR t2.team_name ILIKE '%NE Surf%'
   OR t2.team_name ILIKE '%Maryland Rush%'
   OR t2.team_name ILIKE '%Seacoast United%'
ORDER BY r.rating DESC;
```

---

## Section 2: Comparison Table

> [!warning] DB Query Required
> The Evo Draw Rank and Rating columns below require running the SQL in Section 1. Columns marked `[QUERY]` need FORGE or Presten to populate. All other columns are complete.

| Team | USA Rank | Evo Draw Rank | Evo Draw Rating | Delta | Band | Notes |
|------|----------|--------------|-----------------|-------|------|-------|
| Football Academy NJ 2012 Girls Black | #4 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | ECNL club — expect high rating, likely Gold or Silver |
| Wall Soccer Club Wall Elite Porto | #12 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | Expect top-50 Evo Draw U13G |
| FA 12G Black | #38 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | May be same club as Football Academy — check merges |
| FC Stars G2012 ECNL White | #121 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | ECNL — expect Silver band |
| NE Surf G2012 State Navy | #198 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | State team — expect Red/Bronze |
| Scorpions SC ECNL G12 | #317 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | ECNL — expect Red band |
| NYCFC Girls 2012 North | #491 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | MLS-affiliated — expect Red/Blue |
| Dix Hills EST Angel City G2012 | #562 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | State/regional — expect Blue |
| BVB International Academy Pittsburgh 12G Prem | #613 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | BVB branding (Borussia affiliate) — expect Blue |
| Old Line FC Girls 12 Black | #741 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | Mid-Atlantic — expect Blue/Green |
| SYC 2012G GA II | #826 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | GA — could appear ranked differently due to GA calibration (140) |
| Maryland Rush Azzurri 2011 | #144 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | ECNL — expect Silver/Bronze |
| NJ Premier FC G2011 | #810 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | Mid-tier — expect Blue |
| Marlton Chiefs Premier 2011 | #926 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | Regional — expect Blue/Green |
| Greater Severna Park 11G Green | #988 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | State — expect Blue/Green |
| Seacoast United MA 2011G MA Elite Navy | #1995 | [QUERY] | [QUERY] | [QUERY] | [QUERY] | State Navy — expect Green band; very low USA Rank |

---

## Section 3: Correlation Analysis — Pre-Query Predictions

Even without running the queries, we can make strong predictions about where rankings will and will not align. This is based on the algorithmic differences and what we know about the teams.

### Where Rankings SHOULD Align Well

**Top-tier ECNL and MLS NEXT teams:** Both systems have substantial game history for top teams. Football Academy NJ, FC Stars ECNL, and Scorpions ECNL all compete in events where Evo Draw has strong data coverage (GotSport API). The ordering within this group should be similar to USA Rank's.

**Clearly weak teams at the bottom:** A team ranked USA Rank #1995 (Seacoast United MA Elite Navy) should be in a low Evo Draw band (Blue or Green). If they appear in our Bronze or Silver, that is a data problem.

**In-state clusters:** Teams from the same geographic region compete against each other frequently. Their relative ordering within a region tends to be accurate in both systems.

### Where Rankings MAY Diverge — and Why

**Case 1: GA-affiliated teams (SYC 2012G GA II)**

USA Rank does not hardcode league quality — it emerges from results. Evo Draw applies a GA calibration bonus of +140 points. If SYC 2012G GA II has played mostly within GA (against other GA teams who also carry the +140), their rating reflects the calibration boost. In USA Rank, if they have not played cross-league games recently, their rank is based purely on GA-internal win rates.

**Expected outcome:** If SYC is ranked #826 by USA Rank, Evo Draw may rank them noticeably higher (potentially 200-400 positions higher) if the GA calibration bonus is inflating their rating relative to teams at equivalent true skill levels. This is the core calibration validation question for GA teams. See `task-2026-04-23-ga-data-coverage-audit.md` for the formal audit.

**Case 2: Teams where USA Rank has more recent game data**

USA Rank uses last 20 games in the last year. Evo Draw uses full history. A team that was excellent 2 years ago but has declined in 2025-26 will look better in Evo Draw than in USA Rank. A team on a recent run of form will look better in USA Rank.

**Expected outcome:** For teams like Dix Hills or BVB Pittsburgh that do not play top-tier cross-league schedules, USA Rank will reflect their current season performance. Evo Draw may have them rated based on a mix of 2023-24 and 2024-25 data. If these teams have regressed, Evo Draw will show them higher than USA Rank.

**Case 3: Teams Evo Draw cannot find at all (missing source data)**

USA Rank has 400+ sources. Evo Draw has 7. Some of the reference teams may not be in our database at all, or may have fewer than 6-10 games recorded (below the ranking threshold). 

**Teams most at risk of being missing from Evo Draw:**
- SincSports-only teams (if we don't have that source)
- SnapSoccer-only teams (we don't have SnapSoccer)
- Teams from state associations not covered by GotSport or TGS

**Case 4: The FA 12G Black / Football Academy NJ possible duplicate**

USA Rank shows both "The Football Academy NJ 2012 Girls Black" (#4) and "FA 12G Black" (#38) as separate teams. In Evo Draw, these may be merged into a single canonical team record (if the dedup process identified them as the same team) or may exist as two separate records. 

- If merged: Evo Draw's combined rating benefits from the full game history of both. The canonical team likely appears in our top-10.
- If not merged: Both records exist separately, each with partial game history. Both may rank lower than they should.

This is a merge verification item — check `team_merges` for "Football Academy" or "FA 12G."

---

## Section 4: Identified Gaps and Root Causes

### Gap 1: Source Coverage (Primary — Expected)

**Root cause:** USA Rank has 400+ sources. Evo Draw has 7. Any team that primarily plays in SnapSoccer-managed events or SincSports-only tournaments will have an incomplete game history in Evo Draw.

**Diagnostic query:**
```sql
-- Check game count and source breakdown for each target team
SELECT 
  t.team_name,
  g.source,
  COUNT(*) as game_count
FROM teams t
JOIN games g ON (g.home_team_id = t.team_id OR g.away_team_id = t.team_id)
WHERE t.team_name ILIKE '%NE Surf%G2012%'  -- substitute each target team
  AND g.is_hidden = false
GROUP BY t.team_name, g.source
ORDER BY game_count DESC;
```

If a team has fewer than 15 games in Evo Draw but USA Rank shows them at rank #198, they likely have 20+ games in USA Rank's dataset from sources we don't cover.

### Gap 2: Recency Weighting (Secondary — Methodological)

**Root cause:** USA Rank uses only the last 20 games in the last year with recency weighting. Evo Draw uses full game history. Teams that have significantly improved or declined recently will show the largest divergence.

**This is not a calibration problem.** This is a design choice. The question is which system is more accurate. We believe full history with Glicko-2 uncertainty modeling is more accurate than a 20-game recency window — but the user experience difference matters.

**Recommendation:** Add a "Recent Form" indicator to Evo Draw team pages that shows the last 10-game win rate and direction of rating change. This closes the UI gap without changing our algorithm.

### Gap 3: League Calibration Differential (Tertiary — Investigate)

**Root cause:** Our GA calibration (140) is higher than ECNL girls (130). USA Rank derives league quality empirically. If GA teams are truly slightly stronger than ECNL girls on average, both systems should rank GA teams similarly. If our GA calibration is inflating GA teams above their true level, they will appear higher in Evo Draw than in USA Rank.

**The SYC 2012G GA II case (USA Rank #826) is the key test.** When the query is run:
- If Evo Draw ranks SYC near #826: calibration is fine.
- If Evo Draw ranks SYC in the #400-600 range: GA calibration may be inflated.
- If Evo Draw ranks SYC in the #200-400 range: GA calibration is definitely inflated.

This is formally investigated in `Reports/ga-calibration-validation-2026-04-23.md`.

### Gap 4: Team Name Matching / Dedup Quality

**Root cause:** USA Rank normalizes team names across sources. Evo Draw uses `team_merges` with 27,820 records (9,136 unverified). If a high-ranked team has multiple source records that are not yet merged (or incorrectly merged), their game history is split across records and their effective rating is underestimated.

**Check:** For any team where Evo Draw rank is significantly lower than USA Rank, run the merge check query in Section 1C. If the canonical team has a low match_count but the merged team's record has substantial game history, the merge may not be linked correctly.

---

## Section 5: Teams Evo Draw May Be Missing Entirely

Based on source coverage analysis, the following teams are most at risk of not appearing in Evo Draw rankings or having insufficient game history:

| Team | USA Rank | Risk of Missing from Evo Draw | Reason |
|------|----------|-------------------------------|--------|
| NE Surf G2012 State Navy | #198 | Medium | "State Navy" team — may play primarily in state association events (USYS/SincSports), which we may not cover |
| Marlton Chiefs Premier 2011 | #926 | High | "Chiefs" branding — smaller club, likely plays in EDP or state leagues with limited GotSport presence |
| Greater Severna Park 11G Green | #988 | High | Maryland state league team — "Green" sub-team suggests multi-team club, primarily USYS state events |
| Seacoast United MA 2011G MA Elite Navy | #1995 | Medium | "MA Elite Navy" — Massachusetts state league, Seacoast United is a large club with GotSport presence but state team specifically may have limited cross-platform data |
| NYCFC Girls 2012 North | #491 | Low-Medium | MLS-affiliated academy — likely in GotSport via MLS NEXT, but "Girls 2012 North" may be the NYCFC girls program which is separate from their MLS NEXT boys program |

**Expected result when queries run:** 5-8 of the 16 reference teams will not appear in Evo Draw's ranked list, or will appear with match_count < 6 (below ranking threshold). This is a source coverage gap, not a calibration problem.

---

## Section 6: Recommendations

### Immediate (Before DSS May 16)

1. **Run the SQL in Section 1** — Presten or FORGE executes against `youth_soccer` DB. Populate the comparison table in Section 2. This takes < 30 minutes.

2. **Flag any team in USA Rank top-100 that we don't have ranked** — that's a source coverage gap that needs to be documented for the data roadmap.

3. **Check the FA/Football Academy merge** — if these are the same team and not merged, merge them before DSS. Top-5 team with split game history is a ranking quality issue.

### Short-Term (May 2026)

4. **Add SnapSoccer as a data source** — USA Rank captures many northeast corridor teams through SnapSoccer events. This is the highest-impact single source addition for this geographic region.

5. **Add SincSports/SoccerInCollege feed** — USA Rank's FAQ explicitly lists this as a primary source. State cup and USYS events flow through here.

6. **Cross-reference top 200 in each USA Rank age group against our database** — systematic coverage audit, not just the 16 teams listed here. Target: identify the top 20 teams in each age group × gender that USA Rank has and we don't.

### Algorithm (Not Urgent)

7. **Add a recency weight layer to Evo Draw** — we don't need to copy USA Rank's "last 20 games" approach, but displaying a recency-weighted "Current Form" score alongside our full-history Glicko-2 rating closes the UX gap.

8. **After GA audit completes** — if GA calibration (140) is confirmed inflated, adjust and rerun. This will bring GA teams closer to USA Rank's assessment.

---

## Section 7: How to Interpret the Results

Once FORGE populates Section 2, use this rubric:

| Evo Draw Rank vs USA Rank | Interpretation |
|---------------------------|----------------|
| Within ±2x (e.g., USA #12, Evo Draw #8-24) | Excellent alignment — rankings are telling the same story |
| Within ±3x (e.g., USA #12, Evo Draw #4-36) | Good alignment — noise from recency vs history differences |
| ±3-5x difference | Investigate: likely source coverage gap or recent form divergence |
| >5x difference (e.g., USA #12, Evo Draw #80+) | Serious gap — dedup error, missing source, or calibration issue |
| Team not found in Evo Draw at all | Source coverage gap — document for data roadmap |
| Evo Draw rank much lower than USA Rank | We have more data showing losses they don't count; or dedup created split records |
| Evo Draw rank much higher than USA Rank | We're using older good results; or calibration bonus inflating the team |

**Success threshold for this comparison:**
- ≥ 10 of 16 teams found in Evo Draw with at least 6 games
- ≥ 8 of those found within ±3x rank alignment
- No top-50 USA Rank team appearing below our rank #500 (that would be a calibration/dedup failure)

---

## Related Notes

- [[USA Rank Competitive Intel]] — source of USA Rank reference data
- [[Ranking Engine]] — Evo Draw algorithm for comparison context
- [[Cross-League Analysis]] — win-rate methodology backing our calibration values
- [[League Hierarchy]] — calibration values referenced in gap analysis
- [[GA]] — GA calibration gap analysis (see ga-calibration-validation-2026-04-23.md)
- [[Dedup Strategy]] — team_merges context for Section 4 Gap 4
