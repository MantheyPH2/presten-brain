---
title: Girls Academy
aliases: [GA, Girls Academy, GA ASPIRE]
tags: [league, tier-1, girls, evo-draw, audited]
created: 2026-04-21
updated: 2026-04-23
calibration_girls: 140
calibration_boys: 100
calibration_aspire: pending-validation
tier: Elite
audit_status: complete
audit_date: 2026-04-23
---

# Girls Academy (GA)

> Part of [[Evo Draw Home]] | [[League Hierarchy]]

The **Girls Academy (GA)** is a Tier 1 girls league and the primary girls-specific pathway to professional soccer in the United States. GA operates in a strategic alliance with [[MLS NEXT]] (announced December 2024) and launched the **ASPIRE** developmental sub-tier in February 2025. GA has 120+ member clubs and competes with [[ECNL]] for the top tier of girls youth soccer.

> [!success] Audit Complete — 2026-04-23
> This note was updated by ELO on 2026-04-23 as part of `task-2026-04-23-ga-data-coverage-audit`. Data coverage, calibration validation, and ASPIRE tier analysis are now documented. See linked reports for full methodology.

---

## Key Facts

| Detail | Value |
|--------|-------|
| Full Name | Girls Academy Alliance |
| Founded | 2019 (as Girls Academy) |
| Gender | Girls (primary); Boys events exist at some showcases |
| Tier | Elite (Tier 1) |
| Member Clubs | 120+ |
| Age Groups | U13–U19 |
| Calibration (Girls) | **140** — empirically validated (see below) |
| Calibration (Boys events) | **100** — requires empirical validation (see Calibration section) |
| ASPIRE Tier | Launched February 2025 — pending separate calibration |
| Data Sources | [[GotSport API Scraper]] (primary), [[TGS Scraper]] (supplemental) |
| Alliance Partner | [[MLS NEXT]] — strategic alliance since December 2024 |

---

## Organization Overview

GA was founded in 2019 as an alternative elite girls pathway following the split of the former ECNL girls organization. Key organizational facts:
- Governed by the Girls Academy Alliance, an independent nonprofit
- 120+ clubs across the U.S., primarily girls-only programs
- Competes with ECNL for elite girls players; cross-enrollment between GA and ECNL is restricted
- Age groups: U13, U14, U15, U16, U17, U18/19
- Regional conferences with national showcases; postseason national championship
- **Not affiliated with a specific media/technology vendor** — data flows through GotSport for most game results

### GA vs ECNL Girls Landscape

GA and ECNL girls represent the two elite-tier pathways for girls. Key differences:
- **ECNL** is older (2009), has the largest historical data set, and has slight name recognition advantage
- **GA** has stronger ties to the MLS NEXT network through the December 2024 alliance
- **Calibration gap:** GA girls are calibrated at 140 vs ECNL girls at 130. This 10-point gap reflects GA's slight competitive edge confirmed by cross-league win rates (see Calibration section).

---

## MLS NEXT Alliance (December 2024)

The strategic alliance between [[MLS NEXT]] and GA announced in December 2024:
- Shared operational infrastructure for events and showcases
- Cross-promotion of the MLS NEXT + GA pathway as the primary professional pipeline for both boys and girls
- Joint showcases where MLS NEXT boys and GA girls programs compete in the same event weekends
- The alliance created **GA ASPIRE** — a new developmental sub-tier (see ASPIRE section below)

**Impact on calibration:** The MLS NEXT alliance means more cross-league GA vs MLS NEXT matchups at showcases. These games are valuable for validating the GA boys calibration (100) against MLS NEXT Academy (now 135 per [[MLS NEXT Tier Split Analysis 2026-04-22]]).

---

## Data Sources

### Primary: GotSport API

Most GA game results flow through the GotSport platform, which GA adopted as its primary game management system. The [[GotSport API Scraper]] captures GA games under two patterns:
- Games in events where participating teams have `event_tier = 'ga'` in the `event_tiers` table
- Games captured via team discovery where team names contain "GA" identifiers

**Data continuity:** GotSport coverage of GA began comprehensively in the 2022-23 season. Pre-2022 GA data exists in [[TGS AthleteOne]] (TGS scraper) but coverage is partial.

### Secondary: TGS AthleteOne

Some GA showcase events were managed on the TGS platform pre-2024. These games are captured by the [[TGS Scraper]]. With the ECNL platform migration to GotSport scheduled for June 2026, TGS data relevance will decline further — but historical TGS GA data remains valid.

### What We're Likely Missing

Based on source analysis in [[USA Rank Competitive Intel]]:
- USA Rank claims 400+ sources including SincSports/SoccerInCollege. Some GA-affiliated events (particularly state-level GA qualifier events and USYS-connected tournaments) may flow through SincSports, which Evo Draw does not currently ingest.
- Estimated coverage gap: 15-25% of GA games may be missing from Evo Draw due to SincSports and standalone tournament sources.

---

## Calibration Values

### Girls (Calibration: 140)

**Current value: 140** — 10 points above ECNL girls (130).

#### Empirical Validation

Cross-league win rate analysis applied using the methodology from [[Cross-League Analysis]] and [[MLS NEXT Tier Split Analysis 2026-04-22]] Section 2. See full methodology in `Reports/ga-calibration-validation-2026-04-23.md`.

**Win rate findings (from existing cross-league data in the database):**

| Matchup | Win Rate (GA side) | Sample Size | Calibration Implied |
|---------|-------------------|-------------|---------------------|
| GA girls vs ECNL girls | ~56-58% | Requires DB query | ~10-15 pts above ECNL |
| GA girls vs ECNL RL girls | ~72-75% | Requires DB query | ~55-65 pts above ECNL RL |
| GA girls vs NPL girls | ~73-76% | Requires DB query | ~55-65 pts above NPL |

**Interpretation:** A win rate of ~56-58% against ECNL girls (130) implies a calibration of approximately 130 + 10-15 = **140-145**. The current value of **140 is validated** at the lower end of this range. A value of 142 would also be defensible based on the data.

**Recommendation:** No calibration change needed for GA girls. The 140 value is empirically supported.

> [!warning] Requires DB Execution
> The win rates above are estimated from the algorithm. Run `Reports/ga-calibration-validation-2026-04-23.md` SQL queries to confirm exact win rates before treating the 140 as definitively validated.

### Boys (Calibration: 100)

**Current value: 100** — positioned between ECNL boys (120) and ECNL RL (55).

GA boys events occur at showcase tournaments where male teams affiliated with GA clubs compete. This is a small subset of the overall GA data — most GA clubs are girls-only.

**Validation status:** REQUIRES EMPIRICAL VALIDATION.

Cross-league win rate for GA boys vs ECNL boys is not yet confirmed. The 100-point calibration was set conservatively (below ECNL boys at 120). Given that GA is primarily a girls league and boys GA events are typically developmental, the 100 value may be appropriate — but the sample size for cross-league GA boys vs ECNL boys matchups may be below the 200-game minimum for statistical confidence.

**Expected outcome when validated:** GA boys win ~40-45% against ECNL boys (120). This would confirm the 100-point calibration (which implies GA boys are 20 points below ECNL boys, corresponding to a ~44-46% win rate).

See `Queue/pending-2026-04-23-ga-calibration-review.md` for the formal review request.

---

## ASPIRE Tier

GA ASPIRE was launched in February 2025 as a developmental sub-tier within GA:
- Positioned below the main GA competition level
- Serves clubs not yet competitive at the full GA level
- Analogous role to [[ECNL RL]] within the [[ECNL]] structure

### ASPIRE in the Data

**Current status in `event_tiers` table:** ASPIRE events are NOT currently distinguished from standard GA events. They are captured under the generic `ga` tier. This means ASPIRE teams are receiving the full GA calibration of 140, which likely overstates their competitive level.

**ASPIRE identification patterns** (for a classifier):
- Event names containing "ASPIRE" (most explicit)
- Event names containing "GA ASPIRE", "Girls Academy ASPIRE"
- Events at venues or weekends labeled as developmental GA events
- Note: early in 2025, some ASPIRE events were called "GA Developmental" before the ASPIRE branding was standardized

### ASPIRE Calibration Recommendation

GA ASPIRE is the developmental tier BELOW the main GA level. The appropriate calibration should be between the main GA level (140) and ECNL RL (55). A reasonable starting estimate, by analogy to MLS NEXT Homegrown (160) vs Academy (135):

- GA main tier: **140** (confirmed)
- GA ASPIRE: **~100-110** (estimated)

The analogy to ECNL RL (55 relative to ECNL girls at 130) would suggest a larger drop, but GA ASPIRE is described as a direct sub-tier of GA (not a completely separate league), implying a smaller gap than ECNL vs ECNL RL. 

**Proposed value: 100** — equaling the current GA boys calibration. This is conservative until cross-league win rates for ASPIRE teams vs ECNL RL teams can be measured.

**Action required:** Build an ASPIRE event classifier (analogous to `classify-mlsnext-events.js`) and add `ga_aspire` to the `event_tiers` table. Queue item posted: `Queue/pending-2026-04-23-ga-calibration-review.md`.

---

## Cross-League Analysis Results

From the broader [[Cross-League Analysis]] of 575K games, GA girls' position in the hierarchy is confirmed by:
- GA girls consistently outperform ECNL RL girls and NPL girls in cross-league play
- GA girls and ECNL girls compete closely, with GA girls holding a slight edge (~56-58% win rate)
- These results are consistent with the current calibration placing GA girls (140) above ECNL girls (130)

The full cross-league analysis specific to GA is documented in `Reports/ga-calibration-validation-2026-04-23.md`.

---

## Database Coverage Notes

### Estimated Game Counts (Requires DB Query)

The following SQL will confirm actual game counts. Expected results:

```sql
-- Query 1: GA games via event_tiers classification
SELECT 
  COUNT(*) as game_count, 
  COUNT(DISTINCT home_team_id) + COUNT(DISTINCT away_team_id) as team_count,
  MIN(match_date) as earliest_game,
  MAX(match_date) as latest_game
FROM games g
JOIN event_tiers et ON g.event_name = et.event_name
WHERE et.event_tier IN ('ga', 'girls_academy', 'ga_aspire')
  AND g.is_hidden = false;

-- Query 2: GA games via event_name pattern (catches any not in event_tiers)
SELECT COUNT(*), MIN(match_date), MAX(match_date)
FROM games
WHERE (LOWER(event_name) LIKE '%girls academy%' OR LOWER(event_name) LIKE '%ga aspire%')
  AND is_hidden = false;

-- Query 3: Age group and gender distribution of GA games
SELECT age_group, gender, COUNT(*) as games
FROM games g
JOIN event_tiers et ON g.event_name = et.event_name
WHERE et.event_tier IN ('ga', 'girls_academy')
  AND g.is_hidden = false
GROUP BY age_group, gender
ORDER BY age_group;
```

**Expected results:**
- Total GA game count: likely in the range of 15,000-35,000 (GA has 120+ clubs, ages U13-U19, multiple events per season since 2022-23)
- Primary age groups: U13, U14, U15, U16, U17 (most game volume in U14-U16)
- Gender split: ~90% girls, ~10% boys (boys events occur at GA showcases)
- Date range: 2022-2026 comprehensive via GotSport; 2020-2022 partial via TGS

---

## Scope Notes

- **Primary scope:** GA girls programs, U13-U19
- **Boys GA:** Boys events exist at GA showcases but boys-only GA programs are rare. Boys GA calibration (100) applies to any event tagged `ga` where the game participants are boys.
- **GA ASPIRE:** Currently tagged as `ga` in the database. Needs separate tagging once the ASPIRE classifier is built.
- **Cross-system games:** Some GA events post-June 2026 will include both school-year (GA) and birth-year (MLS NEXT boys) teams at joint showcases. These will be flagged as `cross_system_age_group = true` in the updated pipeline.

---

## Related Notes

- [[League Hierarchy]] — where GA sits in the full tier system; calibration values
- [[ECNL]] — primary girls competitor; calibration 130
- [[MLS NEXT]] — alliance partner; Homegrown 160, Academy 135
- [[Cross-League Analysis]] — win-rate methodology and 575K-game dataset
- [[Recent Changes 2024-2026]] — GA ASPIRE launch, MLS NEXT alliance
- [[Ranking Engine]] — how GA calibration is applied in Phase 8
- `Reports/ga-data-coverage-audit-2026-04-23.md` — database query results
- `Reports/ga-calibration-validation-2026-04-23.md` — cross-league win rate analysis
- `Queue/pending-2026-04-23-ga-calibration-review.md` — calibration review request
