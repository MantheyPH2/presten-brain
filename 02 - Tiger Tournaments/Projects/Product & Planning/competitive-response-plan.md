---
title: Competitive Response Plan — USA Rank
tags: [strategy, competitive, evo-draw, roadmap]
created: 2026-04-23
updated: 2026-04-23
author: SENTINEL
status: active
priority: high
---

# Competitive Response Plan: Match → Beat → Dominate USA Rank

> Part of [[Evo Draw Home]] | See also [[Competitors]], [[USA Rank Competitive Intel]]

**Directive issued:** 2026-04-23 by Presten Manthey
**Context:** USA Rank / USA Sport Statistics is our top competitor. They have 400+ sources, daily updates, rank bands, club rankings, and event strength ratings. We have better algorithms (Glicko v7, 9 phases), more raw data (1.65M games vs their unknown count), and 27,820 team merges as a moat. But we are behind on breadth and visibility. This plan closes the gap before DSS (May 16, 2026) and beyond.

---

## Gap Analysis: Critical Gaps to Close First

| Gap | USA Rank | Evo Draw | Priority |
|-----|----------|----------|----------|
| Source count | 400+ | 7 | CRITICAL |
| Update cadence | Daily | Overnight batch | CRITICAL |
| Rank bands | Gold/Silver/Bronze/Red/Blue/Green | None | HIGH |
| Club rankings | Yes (U11–U17, 5+ age groups) | None | HIGH |
| Event strength ratings | High/Avg/Low, Spread, Upsets | None | HIGH |
| User-submitted sources | Yes | None | HIGH |
| Cross-league analysis | Not public | 575K games (built, not exposed) | MEDIUM |
| Match predictions | Implied internal only | Capability exists, no UI | MEDIUM |
| Methodology transparency | Opaque | Partial (H2H endpoint) | MEDIUM |
| Mobile / notifications | Unknown | None | LOW |

---

## Phase 1: Match (Target: DSS — May 16, 2026)

**Goal:** Close the most visible gaps before our highest-stakes event. Coaches and club directors attending DSS will compare us to USA Rank. We must not look inferior on the basics.

### 1.1 Rank Bands — UI Only (2–3 days)
**What:** Display Gold/Silver/Bronze/Red/Blue/Green badges on team ranking pages.
**Why first:** Zero backend work if bands are computed from existing Elo ratings. Pure frontend. Highest visual impact for lowest effort.
**Deliverable:** Rank band badges live on ranking pages before May 10.
**Owner:** ELO designs thresholds (task assigned). Presten implements.
**Reference:** [[Rank Bands Design]] (ELO task: `task-2026-04-23-rank-bands-design.md`)

### 1.2 Event Strength Ratings — Display Only (3–4 days)
**What:** Show High/Average/Low strength label + spread + upset rate on event pages.
**Why second:** Event pages are what coaches browse during and after tournaments. Strength ratings make those pages useful, not just data dumps.
**Deliverable:** Event strength labels on all events with 4+ teams before May 12.
**Owner:** ELO designs computation (task assigned). Presten implements SQL + frontend.
**Reference:** [[Event Strength Ratings Design]] (ELO task: `task-2026-04-23-event-strength-ratings.md`)

### 1.3 Daily Pipeline — Active Teams Only (2–3 days)
**What:** Run pipeline daily against active teams (last 90 days) instead of full overnight batch.
**Why third:** Daily cadence is table stakes. "Rankings updated daily" is a one-line marketing claim that USA Rank makes and we currently cannot.
**Deliverable:** Daily ranking updates running on cron by May 8.
**Owner:** FORGE scopes (task assigned). Presten implements.
**Reference:** [[Daily Pipeline Cadence Scope]] (FORGE task: `task-2026-04-23-daily-pipeline-cadence.md`)

### Phase 1 Success Criteria
- [ ] Rank bands visible on ranking pages (with correct Elo thresholds)
- [ ] Event strength labels visible on event pages
- [ ] Pipeline updating rankings daily (not just overnight)
- [ ] All three live before DSS (May 16, 2026)

---

## Phase 2: Beat (Target: July 31, 2026)

**Goal:** Activate the advantages USA Rank cannot match. Our moat is 1.65M games, 27,820 team merges, Glicko v7 accuracy, and cross-league analysis. Phase 2 makes these visible and useful to users.

### 2.1 Club Rankings (3–5 days implementation)
**What:** Average of top team ratings, U11–U17, 5+ age groups required. Separate boys/girls.
**Why:** Club directors are a primary customer segment for DSS and future managed events. A club ranking page drives organic search traffic and gives clubs a reason to care about their teams' data quality. It also creates a sales angle: "your club is ranked #47 nationally — let us run your division placement."
**Deliverable:** Club rankings live, browsable by state and gender.
**Owner:** ELO designs (task assigned). Presten implements.
**Reference:** [[Club Rankings Design]] (ELO task: `task-2026-04-23-club-rankings.md`)

### 2.2 SnapSoccer Data Source (1–2 weeks total)
**What:** Add SnapSoccer as source #8. Research first (FORGE task), then build scraper.
**Why:** SnapSoccer is explicitly confirmed in USA Rank's data. Adding it closes a named gap and expands coverage for age groups where SnapSoccer runs Playdates events (typically U8–U14).
**Deliverable:** SnapSoccer scraper live, games ingested, dedup verified.
**Owner:** FORGE researches (task assigned). Presten builds scraper after research memo delivered.
**Reference:** [[SnapSoccer Source Research]] (FORGE task: `task-2026-04-23-snapsoccer-source-research.md`)

### 2.3 Source Submission Framework (1 week design + 3–5 days implementation)
**What:** Web form for tournament directors to submit missing event URLs. Queue + triage system. Routing to existing scrapers where possible.
**Why:** We cannot build 400 scrapers. But tournament directors can bring events to us — especially once we have managed event relationships (DSS → more events). USA Rank built their long tail via user submissions. We build ours the same way.
**Deliverable:** Submission form live at `/submit-source`. Queue visible to Presten. At least 10 submissions processed by July 31.
**Owner:** FORGE designs (task assigned). Presten implements.
**Reference:** [[Source Submission Framework Design]] (FORGE task: `task-2026-04-23-source-submission-framework.md`)

### 2.4 Cross-League Analysis — Public Exposure (2–3 days)
**What:** Publish the 575K-game cross-league win rate study as a public page. We have this data. USA Rank does not expose this publicly.
**Why:** This is a genuine differentiator. A public cross-league comparison page (MLS NEXT vs ECNL vs GA vs NPL win rates) drives organic traffic, establishes credibility, and gives us something to point at that USA Rank simply cannot match.
**Deliverable:** Public `/cross-league` page with interactive filters by age group and gender.
**Reference:** [[Cross-League Analysis]]

### 2.5 Methodology Transparency Page (1–2 days)
**What:** Public page explaining Elo/Glicko v7, what data goes in, how ratings update, why cross-league calibration matters.
**Why:** USA Rank is a black box. "We explain our math and they don't" is a trust differentiator that matters to serious coaches and college scouts.
**Deliverable:** `/methodology` page live. Link it from ranking pages and the H2H endpoint.

### Phase 2 Success Criteria
- [ ] Club rankings live and browsable
- [ ] SnapSoccer scraper live with games ingested
- [ ] Source submission form live and accepting submissions
- [ ] Cross-league analysis publicly accessible
- [ ] Methodology page live
- [ ] Source count at 9+ (up from 7)

---

## Phase 3: Dominate (Target: Q4 2026 and beyond)

**Goal:** Build products USA Rank structurally cannot build. Our algorithmic advantage, team merge moat, and division placement capability are hard to replicate. Phase 3 converts them into defensible, revenue-generating products.

### 3.1 Division Placement as a Service (Ongoing — starting Q3 2026)
**What:** Pitch DSS division calibration system to 2–3 external tournament directors. Data-backed placement for any event.
**Why:** This is the fastest path to recurring revenue and data moat expansion. Each managed event generates team merges, calibration data, and a marketing case study. Target from Tournament Landscape: Jefferson Cup (700+ teams), Weston Cup (300+ teams), major Texas events.
**Reference:** [[Division Calibration]], [[Dallas Spring Showcase]], [[Tournament Landscape]]

### 3.2 Recruiting Scout View (Q3 2026)
**What:** Filter rankings by U16–U18, add cross-league context, target college coaches as free users and parents as paying subscribers.
**Why:** College coaches make 60%+ of initial contacts at tournaments. They use rankings to identify teams to scout. TopDrawerSoccer is the incumbent but has subjective methodology. We have the empirical data.
**Reference:** [[College Recruiting]], [[Cross-League Analysis]]

### 3.3 Public API (Q4 2026)
**What:** Authenticated API for accessing team ratings, event data, and rankings.
**Why:** Enables third-party integrations (recruiting platforms, tournament software, state associations) and creates a B2B revenue stream. USA Rank does not appear to offer this.

### 3.4 Automated Event Detection (Q4 2026)
**What:** Monitor GotSport's events endpoint for new events; auto-trigger scraping without manual cron.
**Why:** As event count grows, manual pipeline management doesn't scale. Event-triggered scraping is the architecture that enables near-real-time rankings.

### 3.5 Mobile Notifications (2027)
**What:** Push notifications when a team's ranking changes, when a rival plays, when a new event is posted.
**Why:** Rankings become habitual once teams have a notification hook. This is the retention mechanism.

### Phase 3 Success Criteria
- [ ] 3+ managed events beyond DSS generating revenue
- [ ] Recruiting Scout view live with 500+ unique monthly users
- [ ] Public API with at least 3 paying integrations
- [ ] Source count at 50+ via submission framework growth
- [ ] Rankings updated hourly during peak showcase season

---

## Timeline Summary

| Phase | Target Date | Key Deliverables |
|-------|------------|-----------------|
| Phase 1: Match | May 16, 2026 (DSS) | Rank bands, event strength, daily updates |
| Phase 2: Beat | July 31, 2026 | Club rankings, SnapSoccer, submission framework, cross-league public, methodology page |
| Phase 3: Dominate | Q4 2026–2027 | Division placement scale, recruiting scout, public API, event detection |

---

## Immediate Next Actions (This Week)

1. **ELO:** Start rank bands design — query rating distribution, set thresholds (task: `task-2026-04-23-rank-bands-design.md`)
2. **ELO:** Start event strength ratings design — confirm events/games schema, write computation SQL (task: `task-2026-04-23-event-strength-ratings.md`)
3. **FORGE:** Start daily pipeline scope — analyze active team count, model Option A/B/C (task: `task-2026-04-23-daily-pipeline-cadence.md`)
4. **Presten:** Review design docs as they arrive. Target: implement all Phase 1 items by May 10.
5. **FORGE + ELO:** Remaining tasks (SnapSoccer, source submission, club rankings) — design docs due by May 1.

---

## Why We Win

USA Rank has more sources. We have better algorithms and more validated data.

Their 400+ sources are self-reported. Our 1.65M games include 27,820 verified cross-platform team identity links — something no competitor can replicate without rebuilding our entire merge pipeline from scratch. Every tournament we manage adds to this moat.

Their ranking methodology is opaque ("measured in goals," last 20 games, recency weighting). Our Glicko v7 with 9 phases, cross-league calibration, and H2H violation enforcement is demonstrably more sophisticated — and we can prove it publicly.

The plan is simple: match their visible features fast (Phase 1), then activate our structural advantages (Phase 2 and 3) before they can build what we already have.

---

*Generated 2026-04-23 by SENTINEL. Based on: [[USA Rank Competitive Intel]], [[Strategic Insights]], [[Competitors]], [[Data Sources Overview]].*
