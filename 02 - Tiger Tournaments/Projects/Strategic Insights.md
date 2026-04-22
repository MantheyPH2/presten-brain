---
title: Strategic Insights
aliases: [Strategy, Knowledge Graph Analysis, Cross-Domain Insights]
tags: [strategy, analysis, evo-draw, tiger-tournaments, insights]
created: 2026-04-21
updated: 2026-04-21
author: Presten Manthey
status: active
---

# Strategic Insights — Cross-Domain Analysis

> Part of [[Evo Draw Home]] | Back to [[000 - Presten Manthey]]

This note traces the wikilink connections across all 38 notes in the vault, identifies non-obvious intersections between knowledge domains, and extracts actionable strategic insights for Evo Draw and [[Tiger Tournaments]].

---

## Connection Map

### Cluster 1: Data Ingestion Core
The densest cluster in the vault. Nearly every note connects back here.
```
[[GotSport]] ←→ [[GotSport API Scraper]] ←→ [[GotSport HTML Scraper]]
     ↕                    ↕                           ↕
[[Data Sources Overview]] ←→ [[Dedup Strategy]] ←→ [[Data Pipeline]]
     ↕                    ↕
[[TGS AthleteOne]] ←→ [[TGS Scraper]]
     ↕
[[Sinc Sports]]  [[MLS NEXT Source]]
```
**7 notes, 15+ links.** This is the operational heart of the platform.

### Cluster 2: League Intelligence
```
[[League Hierarchy]] ←→ [[MLS NEXT]] ←→ [[ECNL]] ←→ [[ECNL RL]] ←→ [[Pre-ECNL]]
         ↕                                   ↕
[[Cross-League Analysis]] ←→ [[Ranking Engine]] ←→ [[Division Calibration]]
         ↕
[[Governing Bodies]]
```
**9 notes, 20+ links.** The intellectual backbone -- domain knowledge that makes the algorithm work.

### Cluster 3: Pipeline & Quality
```
[[Data Pipeline]] ←→ [[Parsing Rules]] ←→ [[Age Groups and Birth Years]]
       ↕                    ↕
[[Data Quality]] ←→ [[Scope Rules]]
       ↕
[[Database Schema]] ←→ [[Server and Frontend]]
```
**7 notes, 12+ links.** Operational plumbing -- less glamorous but failure here breaks everything.

### Cluster 4: Market & Go-to-Market
```
[[Competitors]] ←→ [[College Recruiting]] ←→ [[Tournament Landscape]]
       ↕                    ↕
[[Dallas Spring Showcase]] ←→ [[Tiger Tournaments]]
       ↕
[[Team Name Matching]] ←→ [[Division Calibration]]
```
**6 notes, 10+ links.** This cluster is notably thinner than the others. More on this below.

### Cluster 5: Change Drivers
```
[[Recent Changes 2024-2026]] ←→ [[Age Groups and Birth Years]]
         ↕                              ↕
[[ECNL]] migration ←→ [[TGS AthleteOne]] ←→ [[GotSport]]
         ↕
[[MLS NEXT]] birth year exception
```
**5 notes, 8+ links.** Time-sensitive cluster -- these are the notes that will force action in the next 2-14 months.

### Key Gaps in the Link Graph
- **[[Tiger Tournaments]] is an orphan.** Only 2 inbound links (from [[000 - Presten Manthey]] and [[Dallas Spring Showcase]]). The business entity is disconnected from the technical platform -- no notes on revenue, customers, growth strategy, or other events beyond DSS.
- **[[Personal]] has zero outbound links.** Expected for a personal area, but worth noting there's no connection between Presten's goals/skills and the business.
- **No notes on users, customers, or parents.** The vault is deeply technical but has nothing on who uses the rankings or what they do with them.
- **No note on GA (Girls Academy).** Referenced 10+ times across [[League Hierarchy]], [[Recent Changes 2024-2026]], [[Cross-League Analysis]], etc., but has no dedicated note despite being a Tier 1 league with calibration value 140.
- **No pricing, revenue, or business model notes.** Evo Draw's monetization is invisible in the knowledge graph.

---

## Strategic Insights

### Insight 1: The ECNL Migration Is a One-Time Data Consolidation Windfall
**Notes:** [[ECNL]], [[TGS AthleteOne]], [[GotSport]], [[GotSport API Scraper]], [[Dedup Strategy]], [[Recent Changes 2024-2026]]

When ECNL moves from [[TGS AthleteOne]] to [[GotSport]] in June 2026, ~91K games worth of ECNL data will shift from source priority 3 to source priority 1. This is not just a scraper change -- it is a **data quality upgrade**. The [[GotSport API Scraper]] provides richer structured data (team IDs, explicit match metadata) than TGS's birth-year-based division format that requires [[Parsing Rules]] conversion. Post-migration, ECNL game data will have:
- Stable GotSport team IDs (eliminating the birth-year parsing step for new data)
- Better gender metadata (reducing the ~12K null-gender problem in [[Data Quality]])
- Automatic cross-platform team linking (GotSport IDs already power 107,528 team discoveries)

> [!tip] Action
> **Do not just passively let the scraper pick up ECNL data.** Proactively build a migration validation script that compares historical TGS ECNL records against the new GotSport ECNL records to verify continuity. The [[Dedup Strategy]] will need a one-time reconciliation pass for the transition period where both sources emit ECNL games.

### Insight 2: The Team Name Problem Is Really a "No Universal ID" Problem -- and Solving It Is the Moat
**Notes:** [[Team Name Matching]], [[Dedup Strategy]], [[Database Schema]], [[Competitors]], [[Dallas Spring Showcase]]

The 24 manual name aliases in `dss-name-aliases.json` are a symptom of the deeper issue: US youth soccer has **no standardized team identity system**. Each platform ([[GotSport]], [[TGS AthleteOne]], [[Sinc Sports]]) assigns its own IDs. The 27,820 entries in the `team_merges` table represent Evo Draw's attempt to solve this at scale. This is directly tied to Competitive Gap #3 ([[Competitors]]: "Accuracy -- Dedup + Merging").

Here is why this matters strategically: **the team_merges table IS the moat.** Any competitor can scrape GotSport's public API. Nobody else has 27,820 verified cross-platform team identity links built from coach names, fuzzy matching, and manual review. Every new event Evo Draw manages (like [[Dallas Spring Showcase]]) generates more verified merges, which improves rankings for ALL teams, which makes the platform more valuable.

> [!warning] Risk
> The moat is only as strong as the 18,684 verified merges (out of 27,820 total). The 9,136 unverified merges could contain errors that silently corrupt rankings. A batch verification campaign would strengthen the moat significantly.

### Insight 3: The Age Group Shift Will Break Cross-Season Comparisons -- Build the Bridge Now
**Notes:** [[Age Groups and Birth Years]], [[Recent Changes 2024-2026]], [[MLS NEXT]], [[ECNL]], [[Ranking Engine]], [[Parsing Rules]]

Starting 2026-27, most leagues shift to school-year cutoffs (Aug 1 - Jul 31) while [[MLS NEXT]] stays on birth year. This creates a scenario where a player born July 15, 2011 is U16 in ECNL but U15 in MLS NEXT for the same season. The [[Ranking Engine]] currently treats age groups as uniform buckets. When a "U16 ECNL" team plays a "U15 MLS NEXT" team at a showcase, which age group does the game belong to? The current [[Parsing Rules]] have no mechanism for this.

Worse: **historical season-over-season tracking (Competitive Gap #7) becomes unreliable.** A team that was "U15" in 2025-26 might be "U16" in 2026-27 under the new system, but an MLS NEXT team from the same birth cohort stays "U15." The [[Cross-League Analysis]] methodology depends on consistent age group definitions.

> [!tip] Action
> Add a `birth_year_cohort` column to the [[Database Schema|games table]] that normalizes all age groups to birth year, regardless of league cutoff system. This preserves cross-league comparability even as labeling conventions diverge.

### Insight 4: DSS Is the Revenue Proof-of-Concept -- Scale It Before Perfecting the Platform
**Notes:** [[Dallas Spring Showcase]], [[Tiger Tournaments]], [[Division Calibration]], [[Ranking Engine]], [[Team Name Matching]], [[Competitors]]

The [[Dallas Spring Showcase]] has 308 teams (targeting 390) and is the only managed event in the system. It exercises [[Division Calibration]] (Phase 9), the team history API, bracket generation, and the name-matching pipeline. But the vault reveals something important: **DSS is the only revenue-generating use case documented.** The 7 competitive gaps in [[Competitors]] describe platform capabilities, but only DSS actually delivers value to paying customers (clubs and coaches).

The anti-sandbagging code in [[Division Calibration]] is disabled because there is only one season of data. The [[Ranking Engine]] has 9+1 phases but the DSS-specific division calibration (Phase 9) only covers 308 teams out of 144,676. This suggests the platform is over-engineered for a single event and under-leveraged commercially.

> [!tip] Action
> Before investing more in algorithm refinement, use the DSS model to onboard 2-3 more managed events in different regions. Each event generates: (a) revenue, (b) more team_merges, (c) more division calibration data, (d) a marketing proof point. The [[Tournament Landscape]] lists Jefferson Cup (700+ teams), Surf Cup (500+ teams), Weston Cup (300+ teams) as potential partners.

### Insight 5: The College Recruiting Pipeline Is the Highest-Value Unbuilt Product
**Notes:** [[College Recruiting]], [[Tournament Landscape]], [[Competitors]], [[Ranking Engine]], [[Cross-League Analysis]], [[League Hierarchy]]

College coaches make 60%+ of initial contacts at tournaments ([[College Recruiting]]). They check rankings to identify which teams to scout. Evo Draw's [[Cross-League Analysis]] (575K games) provides the only empirical cross-league comparison in the market -- exactly what coaches need to evaluate out-of-region talent. Yet there is no product built around this.

The recruiting timeline shows peak activity at ages 15-17 (U16-U18), which are fully within the 11v11 scope ([[Age Groups and Birth Years]]). The D1 scholarship expansion to 28 ([[Recent Changes 2024-2026]]) increases demand for scouting data. TopDrawerSoccer ([[Competitors]]) is the current leader in college recruiting content but has "limited methodology transparency" and "subjective elements."

The specific product opportunity: **a "Recruiting Scout" view that filters [[Ranking Engine]] output by age group (U16-U18), gender, and region, with cross-league performance context from [[Cross-League Analysis]].** Parents would pay for visibility; coaches would use it for free (which drives parent demand).

### Insight 6: GotSport's Public API Is a Strategic Vulnerability, Not Just a Data Source
**Notes:** [[GotSport]], [[GotSport API Scraper]], [[Competitors]], [[Data Sources Overview]]

The entire platform depends on GotSport's unauthenticated public API. 90% of all game data (1.49M of 1.65M games) comes from [[GotSport]]. The API requires no auth, no API key, and the only rate limiting is self-imposed (1200ms delay). This is an extraordinary situation that could change at any time.

GotSport already has its own rankings system ([[Competitors]]). If GotSport decides to lock down the API or build a competitive cross-league product, Evo Draw loses 90% of its data pipeline overnight. The [[Sinc Sports]] situation (Cloudflare-blocked, no live scraper) shows what happens when a platform decides to restrict access.

> [!warning] Risk
> There is no documented contingency plan for GotSport API lockdown. The [[GotSport HTML Scraper]] is a partial fallback (event-centric vs team-centric) but would not replace the API's team discovery capability (107,528 IDs from the rankings endpoint).

### Insight 7: Sinc Sports' Cloudflare Block Reveals a Data Source Acquisition Pattern
**Notes:** [[Sinc Sports]], [[Data Sources Overview]], [[GotSport]], [[TGS AthleteOne]], [[Dedup Strategy]]

[[Sinc Sports]] has 34,961 games but is permanently frozen -- no live scraper due to Cloudflare protection. The sub-sources (Coast: 5,934, Heartland: 1,291, Squadi: 84) are similarly static. Meanwhile, [[TGS AthleteOne]] (91,401 games) is about to lose its primary use case when ECNL migrates.

Pattern: **data sources have a lifecycle.** They emerge, get scraped, and eventually either get locked down (Sinc) or become irrelevant (TGS post-migration). The sustainable strategy is not to build permanent scrapers for each source but to maintain a **source-agnostic ingestion framework** where new sources can be onboarded quickly.

The current [[Data Pipeline]] is already source-agnostic from Step 1 onward. The bottleneck is scraper creation (Step 0). Consider whether the GA ASPIRE launch ([[Recent Changes 2024-2026]]) and the USYS National League/NPL integration create new data sources that need scrapers before the existing ones decay.

### Insight 8: The Pre-ECNL Anomaly Points to a Missed Market Segment
**Notes:** [[Pre-ECNL]], [[Cross-League Analysis]], [[League Hierarchy]], [[Age Groups and Birth Years]], [[College Recruiting]]

[[Pre-ECNL]] beats NPL 68.4% of the time despite being classified as Tier 3 (calibration 25 vs NPL's 55). The explanation -- "organizational quality, not tier equivalence" -- is correct for ranking purposes but reveals a market insight: **Pre-ECNL parents are elite-club families who care deeply about competitive trajectory.** These are U10-U12 families at ECNL member clubs who are 3-5 years from the [[College Recruiting|recruiting window]].

These families are the highest-LTV customer segment for a rankings platform. They are emotionally and financially invested in their child's soccer development, affiliated with elite clubs, and years away from their data needs peaking (college recruiting at U16-U18). Capturing them at U10-U12 means 5-7 years of engagement.

Yet [[Pre-ECNL]] only has a 25-point calibration and no dedicated coverage in the platform. The [[Scope Rules]] include U7-U19, and [[Age Groups and Birth Years]] covers 4v4 through 11v11 formats, so the data infrastructure supports it.

### Insight 9: MLS NEXT's Birth Year Exception Creates a Cross-League Arbitrage Opportunity
**Notes:** [[MLS NEXT]], [[Age Groups and Birth Years]], [[Recent Changes 2024-2026]], [[Cross-League Analysis]], [[Ranking Engine]]

When the age group shift happens in 2026-27, MLS NEXT (birth year) and ECNL (school year) will define "U16" differently. A player born in January plays in a different competitive cohort depending on the league. This creates legitimate confusion for parents choosing between leagues -- and **Evo Draw is the only platform positioned to explain the difference with data.**

No competitor ([[Competitors]]) tracks both systems simultaneously. The [[Cross-League Analysis]] already compares MLS NEXT vs ECNL performance. Adding age-cohort-aware comparisons ("here's how 2010-born players perform in MLS NEXT U16 vs ECNL U17") would be a completely unique product feature.

### Insight 10: The 7 Competitive Gaps vs What's Actually Built Shows a Prioritization Problem
**Notes:** [[Competitors]], [[Ranking Engine]], [[Cross-League Analysis]], [[Dedup Strategy]], [[Server and Frontend]], [[Data Sources Overview]]

The 7 gaps from [[Competitors]]:

| # | Gap | Status | Evidence |
|---|-----|--------|----------|
| 1 | Cross-League Comparison | **Built** | [[Cross-League Analysis]] -- 575K games analyzed |
| 2 | Data Fragmentation | **Built** | 7 sources, 1.65M games in [[Data Sources Overview]] |
| 3 | Accuracy (Dedup/Merge) | **Built** | [[Dedup Strategy]] -- 27,820 merges |
| 4 | Player Analytics | **Not started** | "Future roadmap" per [[Competitors]] |
| 5 | Methodology Transparency | **Partially built** | H2H violations endpoint exists, but no public methodology page |
| 6 | Regional Strength Mapping | **Not started** | Data supports it but no feature built |
| 7 | Development Tracking | **Not started** | Historical data exists but no UI |

Gaps 1-3 are the foundation and they are solid. But gaps 4-7 are where the user-facing value lives, and none are shipped. The platform is a powerful data engine with a thin UI layer ([[Server and Frontend]]: plain Node.js, 5 API endpoints). The risk is building a technically superior product that nobody outside of DSS participants ever sees.

### Insight 11: The Division Calibration System Is a Portable Product for Any Tournament
**Notes:** [[Division Calibration]], [[Dallas Spring Showcase]], [[Ranking Engine]], [[Tiger Tournaments]], [[Tournament Landscape]]

The Phase 9 division calibration (Gold > Silver Elite > Silver > Bronze > Bronze II > Rec) is specific to [[Dallas Spring Showcase]], but the concept is universal. Every tournament in the [[Tournament Landscape]] places teams into divisions. Most do it subjectively -- "this team feels like Gold level." Evo Draw can offer **data-backed division placement as a service** to any tournament director.

The 308-team DSS dataset proves the concept. The disabled anti-sandbagging code shows the roadmap. The pitch to tournament directors: "We'll place your teams into the right divisions using Elo ratings from 1.65M games across every league in the country. No more sandbagging. No more blowout games."

This is a much easier sale than "use our rankings" because it solves an immediate operational pain point that tournament directors face every registration cycle.

### Insight 12: The Overnight Pipeline Is a Scaling Bottleneck That Becomes Critical at 3+ Events
**Notes:** [[Data Pipeline]], [[GotSport API Scraper]], [[Database Schema]], [[Server and Frontend]], [[Dallas Spring Showcase]]

The current pipeline runs via `run-overnight.sh` on a single machine. The [[GotSport API Scraper]] crawls 107,528 team IDs at 1200ms delay with 3 workers -- that is ~12 hours for a full crawl. The 9-step [[Data Pipeline]] runs sequentially. The [[Server and Frontend]] is a plain Node.js server on port 3000.

This works for one event with 308 teams. It will not work when:
- Multiple managed events need fresh data simultaneously
- The team_merges table grows past 50K+ entries (slowing Phase 1 of [[Ranking Engine]])
- College recruiting users need real-time or near-real-time rankings during showcase season (March-June per [[Tournament Landscape]])

The [[Database Schema]] uses schema-adaptive insertion (scrapers discover columns dynamically), which is clever for development but adds overhead at scale.

---

## Risks & Blind Spots

### Risk 1: GotSport API Dependency
**Severity: Critical** | **Likelihood: Medium** | **Notes:** [[GotSport]], [[GotSport API Scraper]], [[Data Sources Overview]]

90% of data comes from one unauthenticated API. No partnership agreement documented. GotSport has its own rankings. If they add authentication or rate-limit aggressively, the platform loses its data supply chain.

**Mitigation:** Pursue a formal data partnership or affiliate agreement with GotSport. Position Evo Draw as a value-add to GotSport's ecosystem (driving traffic to their events), not a competitor. Alternatively, build a data export pipeline so tournament directors voluntarily share their GotSport event data.

### Risk 2: Single-Person Technical Dependency
**Severity: High** | **Likelihood: High** | **Notes:** [[Evo Draw Home]], [[Tiger Tournaments]]

Presten builds, maintains, and operates every component: scrapers, pipeline, ranking engine, frontend, database, and tournament operations. The vault documents the system well, but there is no team. The [[TGS Scraper]] null-name bug and its "significant production" impact illustrates what happens when one person manages everything.

### Risk 3: Age Group Shift Transition Period
**Severity: Medium-High** | **Likelihood: Certain** | **Notes:** [[Age Groups and Birth Years]], [[Recent Changes 2024-2026]], [[Ranking Engine]], [[Parsing Rules]]

The 2026-27 birth-year-to-school-year shift will happen. The [[Parsing Rules]] and [[Ranking Engine]] are not yet updated for dual-system handling. This will cause data quality degradation precisely when the platform should be demonstrating reliability to potential new customers.

### Risk 4: No Girls Academy (GA) Note or Data Source
**Severity: Medium** | **Notes:** [[League Hierarchy]], [[Competitors]], [[Recent Changes 2024-2026]]

GA is a Tier 1 league with 140 calibration points (girls) and 120+ clubs. The MLS NEXT-GA alliance and ASPIRE launch make it increasingly important. Yet there is no dedicated [[GA]] note, no identified GA-specific data source, and no GA scraper. The [[Cross-League Analysis]] presumably includes GA data via [[GotSport]], but this is not explicitly tracked.

### Risk 5: Revenue Concentration
**Severity: Medium** | **Notes:** [[Dallas Spring Showcase]], [[Tiger Tournaments]]

All documented revenue comes from a single event (DSS) in a single market (Dallas). The target is 390 teams; current registration is 308 (79% of target). No other events or revenue streams are documented.

### Blind Spot: No User Research
The vault has deep technical and domain knowledge but zero notes on user behavior, customer feedback, or market validation. [[College Recruiting]] describes how coaches find players but not whether coaches have ever used or been shown Evo Draw. [[Competitors]] lists market gaps but does not document whether users perceive these as problems worth paying to solve.

### Blind Spot: GA Data Coverage
GA games presumably flow through [[GotSport]], but without a dedicated tracking note, it is unknown how many GA games are in the database, whether the calibration values (140 girls, 100 boys) are empirically validated the way ECNL RL's 55.1% win rate validates its calibration, or whether the ASPIRE tier needs its own calibration.

---

## Action Items (Ordered by Impact)

> [!tip] Prioritization Logic
> Items are ordered by: (immediate revenue impact) > (data moat strengthening) > (risk mitigation) > (future optionality)

### Immediate (Before DSS -- by May 16, 2026)

- [ ] **1. Verify DSS team data coverage for the 13 zero-game teams.** Even at 95.8% coverage, 13 teams with no history means 13 teams whose brackets are guesswork. Cross-reference with [[GotSport]] registration API to confirm these are genuinely new teams. ([[Dallas Spring Showcase]], [[Team Name Matching]], [[Data Quality]])

- [ ] **2. Build the ECNL migration validation script.** June 2026 is 2 months away. Write a script that can compare TGS historical ECNL records against incoming GotSport ECNL records to verify team continuity during the transition. ([[ECNL]], [[TGS AthleteOne]], [[GotSport]], [[Dedup Strategy]])

### Short-Term (May-August 2026)

- [ ] **3. Pitch division placement as a service to 2-3 tournament directors.** Use DSS results as the case study. Target events in the [[Tournament Landscape]]: Jefferson Cup, Weston Cup, or a major Texas event. This is the fastest path to recurring revenue and data moat expansion. ([[Division Calibration]], [[Dallas Spring Showcase]], [[Tournament Landscape]], [[Tiger Tournaments]])

- [ ] **4. Add `birth_year_cohort` column to the games table.** Normalize all age groups to birth year before the 2026-27 shift makes cross-league comparisons ambiguous. This is a one-time schema change with a backfill script. ([[Age Groups and Birth Years]], [[Database Schema]], [[Recent Changes 2024-2026]], [[Parsing Rules]])

- [ ] **5. Create a GA (Girls Academy) knowledge note and audit GA data coverage.** Count GA games in the database, verify calibration values against empirical data (like the [[Cross-League Analysis]] methodology), and add ASPIRE tier handling. ([[League Hierarchy]], [[Recent Changes 2024-2026]], [[Cross-League Analysis]])

- [ ] **6. Run a team_merges verification campaign.** Prioritize the 9,136 unverified merges. Even verifying the top 2,000 (by game volume) would significantly improve ranking accuracy. ([[Dedup Strategy]], [[Ranking Engine]])

### Medium-Term (Fall 2026)

- [ ] **7. Build the "Recruiting Scout" view.** Filter rankings by U16-U18, add cross-league context from [[Cross-League Analysis]], and target college coaches as free users and parents as paying users. ([[College Recruiting]], [[Ranking Engine]], [[Tournament Landscape]], [[Competitors]])

- [ ] **8. Publish methodology transparency page.** Gap #5 from [[Competitors]] is partially built (H2H violations endpoint) but not externalized. A public page explaining the Elo/Glicko v7 approach, the 575K-game cross-league study, and the calibration values would differentiate Evo Draw from every competitor. ([[Ranking Engine]], [[Cross-League Analysis]], [[Competitors]])

- [ ] **9. Update [[Parsing Rules]] and [[Ranking Engine]] for dual age-group systems.** The 2026-27 season starts in August. Both the parsing pipeline and the ranking calibration need to handle school-year and birth-year age groups simultaneously. ([[Age Groups and Birth Years]], [[Recent Changes 2024-2026]], [[MLS NEXT]], [[ECNL]])

- [ ] **10. Investigate GotSport partnership or data agreement.** Even an informal conversation reduces the API dependency risk. Position the value as: Evo Draw drives engagement with GotSport-hosted events and provides tournament directors with a reason to use GotSport for registration. ([[GotSport]], [[Competitors]], [[Data Sources Overview]])

### Long-Term (2027+)

- [ ] **11. Build regional strength mapping (Gap #6).** The database has geographic data from team names, club locations, and event locations. A heat map of competitive density by region would serve both the recruiting market and tournament directors planning expansion. ([[Competitors]], [[Cross-League Analysis]], [[Tournament Landscape]])

- [ ] **12. Evaluate season-over-season development tracking (Gap #7).** With 2+ seasons of data post-2026, this becomes viable. Track Elo trajectories by team/club across seasons. Particularly powerful for the [[Pre-ECNL]] → [[ECNL RL]] → [[ECNL]] promotion pathway. ([[Competitors]], [[Ranking Engine]], [[Pre-ECNL]], [[ECNL RL]])

---

> [!summary] The Core Thesis
> Evo Draw's data engine (1.65M games, 27,820 team merges, 575K-game cross-league analysis) is **genuinely differentiated**. The 7 competitive gaps are real. But the platform is 80% data infrastructure and 20% user-facing product. The next phase is not about collecting more data or refining the algorithm further -- it is about **converting the data moat into products that paying users can see and touch.** Division placement as a service, recruiting scout views, and methodology transparency are the bridges from "best data" to "best product."

---

*Generated 2026-04-21 from analysis of 38 vault notes and their wiki link graph.*
