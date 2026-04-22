---
title: Competitors
aliases: [Competitive Landscape, Ranking Systems, Market Gaps]
tags: [knowledge, competitors, rankings, market-analysis, evo-draw]
created: 2026-04-21
updated: 2026-04-21
---

# Competitors

The youth soccer ranking and data landscape is fragmented. Evo Draw competes with several existing systems, each with significant limitations.

---

## Existing Ranking Systems

### GotSport Rankings
- **Strengths:** Largest data set, official tournament platform
- **Weaknesses:** Only ranks teams within [[GotSport]] ecosystem, no cross-platform data
- **Data:** Rankings endpoint at `/api/v1/team_ranking_data` (used by our [[GotSport API Scraper]])

### TopDrawerSoccer
- **Strengths:** Strong brand recognition, college recruiting focus
- **Weaknesses:** Limited methodology transparency, subjective elements, primarily editorial
- **Focus:** College recruiting content and rankings

### YouthSoccerRankings.us
- **Strengths:** Dedicated youth focus
- **Weaknesses:** Limited data sources, less comprehensive coverage
- **Focus:** Team rankings by age group

### SoccerWire
- **Strengths:** News and content ecosystem
- **Weaknesses:** Rankings are secondary to news coverage
- **Focus:** Youth soccer news, club directories

### PitchRank
- **Strengths:** Modern interface
- **Weaknesses:** Newer entrant, building data coverage
- **Focus:** Team and player analytics

### Others
- State association rankings (fragmented by state)
- League-specific rankings ([[ECNL]], [[MLS NEXT]] internal)
- Coach/media polls (subjective)

---

## The 7 Major Gaps (Our Opportunity)

> [!tip] These gaps define Evo Draw's competitive advantage

### 1. Cross-League Comparison
**Gap:** No existing system compares teams across [[MLS NEXT]], [[ECNL]], GA, NPL, and state leagues on the same scale.
**Evo Draw:** Our [[Ranking Engine]] processes 570K+ games across all leagues with calibrated tier weights. See [[Cross-League Analysis]].

### 2. Data Fragmentation
**Gap:** Game data is siloed across [[GotSport]], [[TGS AthleteOne]], [[Sinc Sports]], and proprietary league platforms.
**Evo Draw:** We aggregate 7 sources into a unified [[Database Schema|database]] with 1.65M games. See [[Data Sources Overview]].

### 3. Accuracy (Dedup + Merging)
**Gap:** Existing rankings don't handle duplicate games or cross-platform team identity.
**Evo Draw:** Our [[Dedup Strategy]] resolves cross-source duplicates, and 27,820 [[Dedup Strategy|team merges]] link teams across platforms.

### 4. Player Analytics
**Gap:** No system connects team performance to individual player development trajectories.
**Evo Draw:** Future roadmap — foundation is the team-level data.

### 5. Methodology Transparency
**Gap:** Most ranking systems are black boxes. Coaches and parents don't know how rankings are computed.
**Evo Draw:** Our [[Ranking Engine]] methodology is documented. H2H enforcement is visible via the `/api/h2h-violations` endpoint.

### 6. Regional Strength Mapping
**Gap:** No system maps competitive strength geographically to help with travel planning.
**Evo Draw:** Our data supports regional analysis — see [[Cross-League Analysis]] for league-level comparisons.

### 7. Development Tracking
**Gap:** No longitudinal view of how a team/club improves over seasons.
**Evo Draw:** Historical data in the database enables season-over-season tracking.

---

## Competitive Positioning

```
                    Data Coverage
                    High ──────────── Low
                    │                  │
Cross-League   High ┤  EVO DRAW        │  TopDrawerSoccer
Comparison         │                  │
               Low ┤  GotSport        │  State Rankings
                    │  Rankings        │
```

Evo Draw is uniquely positioned in the **high data coverage + high cross-league comparison** quadrant.

---

## Related Notes
- [[Ranking Engine]] — our algorithmic advantage
- [[Cross-League Analysis]] — empirical cross-league data
- [[Data Sources Overview]] — our data breadth
- [[Dedup Strategy]] — our accuracy advantage
- [[College Recruiting]] — the end users who need this data
