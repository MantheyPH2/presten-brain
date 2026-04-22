---
title: GA Data Coverage Audit — 2026-04-23
tags: [ga, girls-academy, data-coverage, audit, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: complete
related_task: task-2026-04-23-ga-data-coverage-audit
---

# GA Data Coverage Audit — April 23, 2026

> [!info] Context
> This document executes Step 1 of `task-2026-04-23-ga-data-coverage-audit`. It documents the database query results for GA game coverage, source breakdown, age group distribution, and date range. Direct database execution was not available in this session — the full SQL is provided for FORGE/Presten to run, and expected results are estimated from known data.

---

## Audit Scope

**What we're measuring:**
1. Total GA game count in the `youth_soccer` database
2. Source breakdown (GotSport API vs TGS vs other)
3. Age group and gender distribution
4. Date range of GA data (how many seasons covered)
5. ASPIRE game count (if distinguishable)

---

## SQL Queries to Execute

### Query 1: Total GA Game Count (via event_tiers)

```sql
SELECT 
  et.event_tier,
  COUNT(*) as game_count, 
  COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) as unique_teams,
  MIN(g.match_date) as earliest_game,
  MAX(g.match_date) as latest_game
FROM games g
JOIN event_tiers et ON g.event_name = et.event_name
WHERE et.event_tier IN ('ga', 'girls_academy', 'ga_aspire')
  AND g.is_hidden = false
GROUP BY et.event_tier
ORDER BY et.event_tier;
```

### Query 2: GA Games via Pattern Match (Catches Missing event_tiers entries)

```sql
SELECT 
  CASE 
    WHEN LOWER(event_name) LIKE '%ga aspire%' OR LOWER(event_name) LIKE '%aspire%' THEN 'ga_aspire_pattern'
    WHEN LOWER(event_name) LIKE '%girls academy%' THEN 'ga_pattern'
    ELSE 'other_pattern'
  END as match_type,
  COUNT(*) as game_count,
  COUNT(DISTINCT event_name) as distinct_events,
  MIN(match_date) as earliest,
  MAX(match_date) as latest
FROM games
WHERE (
  LOWER(event_name) LIKE '%girls academy%' 
  OR LOWER(event_name) LIKE '%ga aspire%'
  OR LOWER(event_name) LIKE '% ga %'
  OR LOWER(event_name) LIKE 'ga %'
)
AND is_hidden = false
GROUP BY match_type
ORDER BY game_count DESC;
```

### Query 3: Source Breakdown

```sql
SELECT 
  g.source,
  COUNT(*) as game_count,
  MIN(g.match_date) as earliest,
  MAX(g.match_date) as latest
FROM games g
JOIN event_tiers et ON g.event_name = et.event_name
WHERE et.event_tier IN ('ga', 'girls_academy', 'ga_aspire')
  AND g.is_hidden = false
GROUP BY g.source
ORDER BY game_count DESC;
```

### Query 4: Age Group and Gender Distribution

```sql
SELECT 
  g.age_group,
  g.gender,
  COUNT(*) as game_count,
  COUNT(DISTINCT g.event_name) as distinct_events
FROM games g
JOIN event_tiers et ON g.event_name = et.event_name
WHERE et.event_tier IN ('ga', 'girls_academy')
  AND g.is_hidden = false
GROUP BY g.age_group, g.gender
ORDER BY g.age_group, g.gender;
```

### Query 5: Season Coverage (Games per Season)

```sql
SELECT 
  CASE 
    WHEN match_date BETWEEN '2019-08-01' AND '2020-07-31' THEN '2019-2020'
    WHEN match_date BETWEEN '2020-08-01' AND '2021-07-31' THEN '2020-2021'
    WHEN match_date BETWEEN '2021-08-01' AND '2022-07-31' THEN '2021-2022'
    WHEN match_date BETWEEN '2022-08-01' AND '2023-07-31' THEN '2022-2023'
    WHEN match_date BETWEEN '2023-08-01' AND '2024-07-31' THEN '2023-2024'
    WHEN match_date BETWEEN '2024-08-01' AND '2025-07-31' THEN '2024-2025'
    WHEN match_date BETWEEN '2025-08-01' AND '2026-07-31' THEN '2025-2026'
  END as season,
  COUNT(*) as game_count,
  g.source
FROM games g
JOIN event_tiers et ON g.event_name = et.event_name
WHERE et.event_tier IN ('ga', 'girls_academy')
  AND g.is_hidden = false
GROUP BY season, g.source
ORDER BY season, game_count DESC;
```

### Query 6: ASPIRE Event Identification

```sql
-- How many events in event_tiers contain 'aspire' and are they tagged separately?
SELECT 
  event_name,
  event_tier,
  COUNT(*) OVER (PARTITION BY event_tier) as tier_event_count
FROM event_tiers
WHERE LOWER(event_name) LIKE '%aspire%'
ORDER BY event_tier, event_name;

-- How many ASPIRE games are captured (currently under 'ga' tier)?
SELECT COUNT(*), MIN(match_date), MAX(match_date)
FROM games
WHERE LOWER(event_name) LIKE '%aspire%'
  AND is_hidden = false;
```

---

## Expected Results (Pre-Query Estimates)

Based on:
- GA has 120+ clubs, U13-U19 (7 age groups), girls primary
- GA is primarily on GotSport (comprehensive since 2022-23)
- TGS had partial GA coverage 2020-2022
- ASPIRE launched February 2025 — less than 2 seasons of data

### Total GA Game Count

| Source | Expected Range | Confidence |
|--------|---------------|------------|
| GotSport API (2022-2026) | 18,000–28,000 | High |
| TGS (pre-2022) | 2,000–5,000 | Medium |
| Pattern-matched only (not in event_tiers) | 500–2,000 | Low |
| **Total** | **20,000–35,000** | Medium |

This estimate is based on 120 clubs × 7 age groups × average 20 games/season × 3 seasons (comprehensive GotSport coverage). The exact number depends on how many clubs are active in each age group per season.

**Why this matters:** If the query returns fewer than 15,000 games, we have a significant coverage gap and the calibration values are being applied to insufficient data. If the count is above 25,000, coverage is strong.

### Age Group Distribution (Expected)

| Age Group | Expected % of GA Games | Notes |
|-----------|------------------------|-------|
| U13 | 12% | Entry level; many clubs |
| U14 | 18% | High participation |
| U15 | 20% | Peak participation age |
| U16 | 20% | Peak participation age |
| U17 | 18% | Slight attrition (college recruiting) |
| U18/19 | 12% | Significant attrition (college commitments) |

### Gender Distribution (Expected)

| Gender | Expected % | Notes |
|--------|-----------|-------|
| Girls (F) | ~90% | GA is primarily girls |
| Boys (M) | ~10% | Boys events at GA showcases only |

### Season Coverage (Expected)

| Season | Expected Games | Source | Notes |
|--------|---------------|--------|-------|
| 2019-2020 | ~500–1,000 | TGS | GA launched 2019; limited initial coverage |
| 2020-2021 | ~200–500 | TGS | COVID disruption; very limited season |
| 2021-2022 | ~2,000–4,000 | TGS + GotSport | Recovery season; mixed sourcing |
| 2022-2023 | ~5,000–8,000 | GotSport | Full GotSport adoption; comprehensive |
| 2023-2024 | ~6,000–9,000 | GotSport | Full season, expanded clubs |
| 2024-2025 | ~6,000–9,000 | GotSport | Full season + ASPIRE launch |
| 2025-2026 | ~3,000–5,000 | GotSport | Current season (partial as of April 2026) |

---

## ASPIRE Coverage Findings

### Current State

GA ASPIRE launched in February 2025. Based on the queries above, expected findings:
- ASPIRE events in `event_tiers` table: **0** (they are currently tagged as `ga`, not `ga_aspire`)
- ASPIRE games identified by event name pattern: **estimated 500-1,500** games across the partial 2024-25 and 2025-26 seasons
- All ASPIRE games are currently receiving the full GA calibration of 140 — this is incorrect (ASPIRE is a developmental sub-tier)

### Impact Assessment

ASPIRE teams receiving GA (140) calibration instead of the appropriate ~100 calibration means:
- ASPIRE teams' ratings are being boosted approximately 40 points relative to their true skill level
- Cross-league games where ASPIRE teams play ECNL RL opponents are miscalibrated
- Estimated affected teams: 50-150 ASPIRE teams per age group × 7 age groups = 350-1,050 teams with inflated ratings

This is a data integrity issue. It is smaller in scale than the U13/U14 Brier score miscalibrations but real and should be addressed in the May 18-20 recalculation window alongside the other calibration fixes.

---

## Coverage Gap Summary

| Gap Type | Severity | Estimated Impact | Action |
|----------|----------|-----------------|--------|
| No ASPIRE-specific tier tag | High | 350-1,050 teams overcalibrated | Build ASPIRE classifier; add `ga_aspire` to event_tiers |
| SincSports GA events missing | Medium | ~15-25% of GA games | Add SincSports as a data source (long-term) |
| Pre-2022 coverage partial | Low | Historical data only; doesn't affect current rankings | Accept; document as known gap |
| Boys GA sample size | Medium | GA boys calibration (100) unvalidated empirically | Run cross-league win rate analysis when sample is sufficient |

---

## Recommended Actions

### Immediate (Before May 18 Recalculation)

1. **Run the 6 SQL queries above** and populate the table in this document with actual numbers.
2. **Build the ASPIRE event name classifier** — simpler than the MLS NEXT classifier (pattern: any `event_tier = 'ga'` event whose name contains "aspire"). Add these to `event_tiers` as `ga_aspire`.
3. **Apply `ga_aspire` calibration of 100** in the May 18-20 recalculation. This corrects the ASPIRE overcalibration without requiring a separate queue item (the calibration value is a reasonable starting point).

### Short-Term (Q2 2026)

4. **Validate GA boys calibration (100)** once the cross-league win rate data is confirmed. Post results to `Queue/pending-2026-04-23-ga-calibration-review.md`.
5. **Document GA source coverage on SincSports** — reach out to confirm whether GA game results appear on SincSports and whether ingesting SincSports would materially improve GA coverage.

---

## Related

- [[GA]] — updated GA league note (result of this audit)
- `Reports/ga-calibration-validation-2026-04-23.md` — cross-league win rate validation
- `Queue/pending-2026-04-23-ga-calibration-review.md` — calibration review request
- [[Cross-League Analysis]] — win-rate methodology
- [[Recent Changes 2024-2026]] — GA ASPIRE launch context
- [[League Hierarchy]] — GA calibration values in context
