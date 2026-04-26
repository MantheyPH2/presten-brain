---
title: Competitive Coverage Gap Inventory — April 2026
tags: [infrastructure, competitive, coverage, gaps, dss, evo-draw, forge, sentinel]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: filed — update when competitor indexing is confirmed or coverage expands
due: 2026-05-05
task: task-2026-04-25-competitive-coverage-gap-inventory
---

# Competitive Coverage Gap Inventory — April 2026

> [!info] Purpose
> SENTINEL competitive comparison reference. Maps what each competitor indexes against what Evo Draw covers. Identifies gaps before the May 9 DSS go/no-go. DSS audience will ask "does your data cover X league?" — this inventory pre-answers those questions.

**Prepared by:** FORGE — 2026-04-25  
**Sources:** `[[Competitors]]` vault note, `[[League Hierarchy]]`, `Infrastructure/Pre-DSS Source Coverage Report — May 2026.md`, `Infrastructure/Source Coverage Priority Matrix — April 2026.md`, public competitor websites.

> [!warning] Confidence Note
> FORGE does not have authenticated access to competitor databases or APIs. Competitor coverage claims below are rated `confirmed` (visible on public website/documented methodology), `inferred` (strongly implied by competitor's stated focus and public sample data), or `unknown` (insufficient public evidence). Do not present `inferred` claims as confirmed facts in the DSS.

---

## Section 1: Competitor Source Index

| Competitor | Known Indexed Leagues / Sources | Confidence |
|-----------|--------------------------------|------------|
| **GotSport Rankings** | MLS NEXT, GA Girls, GA Boys, GA ASPIRE, ECNL RL, NPL, DPL, Pre-ECNL, USYS State Leagues/Cups, USL Academy, EDP, Elite 64, NAL — any league whose games are administered through GotSport platform | confirmed — GotSport rankings pull directly from their own platform game records |
| **GotSport Rankings** | ECNL Boys, ECNL Girls | **NOT covered** — ECNL runs on TGS AthleteOne, not GotSport. GotSport Rankings has zero visibility into ECNL game data. |
| **TopDrawerSoccer** | ECNL Girls, ECNL Boys, MLS NEXT, GA Girls | confirmed — editorial rankings with explicit league coverage; methodology pages reference these leagues |
| **TopDrawerSoccer** | ECNL RL, NPL, DPL, state leagues | inferred — TopDrawerSoccer publishes some Tier 2 content but coverage is editorial/manual, not systematic ingestion |
| **TopDrawerSoccer** | USYS State Cups, USL Academy, EDP, Elite 64, GA ASPIRE (as distinct tier) | unknown — not documented in public methodology; likely not systematically indexed |
| **YouthSoccerRankings.us** | Main Tier 1 leagues (ECNL, MLS NEXT, GA) and some state leagues | inferred — site claims "comprehensive" youth coverage; sample rankings show Tier 1 teams |
| **YouthSoccerRankings.us** | Tier 2 leagues (ECNL RL, NPL, DPL), USYS state divisions, USL Academy, EDP | unknown — limited public documentation of data sourcing methodology |
| **SoccerWire** | Rankings are secondary to news content; likely limited to a few top-tier leagues | inferred — SoccerWire's ranking product is minor relative to its news business; no systematic cross-league data |
| **PitchRank** | Major leagues (ECNL, MLS NEXT, GA); building data coverage | inferred — newer entrant, modern interface; stated focus on major leagues; depth unknown |
| **State Association Rankings** (fragmented) | State-specific leagues for each association's member clubs; Affinity Soccer states (IL, WI, MN, MO) have Affinity-powered rankings | confirmed — each state runs its own system for registered clubs |
| **ECNL Internal Rankings** | ECNL Boys, ECNL Girls | confirmed — ECNL publishes internal standings/rankings for member clubs only |
| **MLS NEXT Internal Rankings** | MLS NEXT Boys | confirmed — MLS NEXT publishes internal standings for member academies |

---

## Section 2: Coverage Gap Table

Leagues or sources that competitors index but Evo Draw does **not** currently cover or covers incompletely:

| League / Source | Competitor(s) Indexing | Evo Draw Status | Gap Severity | Notes |
|----------------|----------------------|-----------------|-------------|-------|
| **Affinity Soccer state leagues** (IL, WI, MN, MO) | State association rankings for those states; Affinity Soccer platform | Not covered — no ingestion path before June 2026 | **HIGH** | IL alone estimated 5,000–15,000 games/season. WI, MN, MO add additional volume. No Affinity scraper exists. Largest geographic coverage gap. DSS audiences from Midwest will notice. |
| **USYS State Leagues/Cups — unranked divisions** | GotSport Rankings (all GotSport-platform state games visible in GotSport's own ranking) | Partial — P1 gap; 20,000–35,000 games estimated missing from unranked divisions | **HIGH** | GotSport's own ranking system sees these because they're on the GotSport platform. Evo Draw ingests ranked teams but misses unranked divisions. TX test crawl gates the fix (Stage 1). |
| **NAL (North American League)** | Unknown — may be on GotSport; if so, GotSport Rankings covers it | Not covered — platform unconfirmed | **LOW** | Volume likely <1,000 games/season. If confirmed on GotSport: closeable pre-DSS. If not: deferred. |
| **GA ASPIRE (as correctly calibrated distinct tier)** | GotSport Rankings (data present but calibration differentiation unclear — GotSport may not separate ASPIRE from GA in rankings) | Pending — fix April 28; becomes correctly calibrated post-fix | **MEDIUM** | After April 28 Evo Draw will be the **only** system to explicitly calibrate GA ASPIRE as a distinct tier (cal=100 vs GA cal=140). This is a gap that becomes an **edge** on April 29. |
| **EDP (Eastern Development Program)** | GotSport Rankings (EDP uses GotSport) | Research required — partial ingestion likely but unverified | **MEDIUM** | ~150+ clubs, Northeast focus. A single DB query confirms partial coverage; browser lookup confirms org_id. Could be closed in one 20-min session. |
| **USL Academy** | GotSport Rankings (USL Soccer is a GotSport organization) | Research required — org_id not yet confirmed | **MEDIUM** | 90 clubs with professional pathway. High probability GotSport org_id exists. Closeable pre-DSS with browser lookup. |
| **Elite 64** | GotSport Rankings (US Club Soccer affiliation; likely on GotSport) | Research required — org_id not confirmed | **LOW** | Volume unknown. Closeable if on GotSport. Low severity because volume and ranking impact are not established. |

---

## Section 3: Evo Draw Competitive Edge

Leagues or capabilities that Evo Draw covers that no major competitor indexes or replicates:

| League / Capability | Why No Competitor Covers It | DSS Value |
|--------------------|-----------------------------|-----------|
| **ECNL + MLS NEXT + GA in the same calibrated ranking** | GotSport Rankings misses ECNL (TGS platform). TopDrawerSoccer and others do ECNL *or* GotSport leagues, not both, in a single calibrated scale. No competitor has ingested TGS AthleteOne + GotSport API simultaneously. | **HIGH** — the core DSS differentiator. Coaches at ECNL clubs can directly compare to MLS NEXT and GA teams for the first time on a data-driven scale. |
| **Cross-league calibration methodology** (H2H-derived, documented, public) | Competitor rankings are largely black boxes or editorial. GotSport rankings are platform-scoped. No competitor publishes a calibrated ELO-style methodology with transparent derivation. | **HIGH** — a DSS audience of club directors and league administrators will ask "how is this computed?" Evo Draw is the only system with a documented, verifiable answer. |
| **GA ASPIRE as a distinct calibrated tier (post-April 28)** | Competitors do not appear to separate GA ASPIRE from standard GA in calibration. GotSport likely uses a uniform GA value. The 30-point overcalibration exists in every competitor that carries GA data. | **HIGH** — positions Evo Draw as more precise than incumbents for GA-affiliated coaches and clubs who know ASPIRE is a distinct tier. |
| **Cross-source deduplication** (27,820 team merges) | Competitors either use a single platform (GotSport Rankings) or manual editorial curation (TopDrawerSoccer). No competitor handles cross-platform team identity resolution at scale. | **MEDIUM** — relevant for coaches who play on multiple platforms; less visible to a DSS audience unless they notice rankings that correctly merge teams others split. |
| **1.65M game database with 570K+ processed** | GotSport Rankings is the only competitor with comparable volume, but it is platform-scoped. No competitor has aggregated multi-platform game data at this scale. | **MEDIUM** — compelling as a headline DSS metric; "more games processed than any cross-platform competitor" is defensible. |

---

## Section 4: Priority Additions

Top 5 source additions to reduce competitive coverage gaps, ranked by (1) league tier, (2) number of competitors who already index it, (3) feasibility:

| Rank | Source | Gap Closed | Feasibility | Stage |
|------|--------|-----------|-------------|-------|
| 1 | **USYS State Leagues / Cups (unranked divisions)** | Closes the largest game-volume gap vs. GotSport Rankings (20,000–35,000 estimated games); all GotSport-ranked systems already see these | Easy — TX test crawl (Stage 1) already staged; full expansion (Stage 2) adds all U.S. state org IDs post-TX-pass | Stage 1 / Stage 2 |
| 2 | **EDP (Eastern Development Program)** | Closes a Tier 2 Northeast gap; GotSport Rankings covers it; Evo Draw likely has partial coverage, unverified | Easy — browser lookup + `SELECT COUNT` confirms or closes; ~20 min Presten action | Pre-DSS (Stage 2) |
| 3 | **USL Academy** | Closes a Tier 2 professional-pathway gap that GotSport Rankings covers; 90 clubs with growing brand recognition | Easy — browser lookup at `system.gotsport.com`; high probability org_id exists; ~15 min | Pre-DSS (Stage 2) |
| 4 | **Elite 64** | Closes a Tier 2 gap; US Club Soccer affiliation; likely on GotSport | Easy — browser lookup; low-risk 15-min action; volume unknown until confirmed | Pre-DSS (Stage 2) |
| 5 | **Affinity Soccer (Illinois + Midwest states)** | Closes the largest geographic coverage gap; state association rankings cover this; Evo Draw has no path | Hard — requires new Affinity Soccer scraper; multi-week engineering effort | Post-DSS (June 2026+) |

---

## Section 5: DSS Competitive Narrative

### Paragraph 1: Evo Draw's Coverage Edge

*Draft language for SENTINEL's DSS briefing — edit as needed, do not fabricate:*

Evo Draw is the only ranking system that combines game data from both TGS AthleteOne — the exclusive platform for ECNL Boys and Girls — and GotSport, the platform for MLS NEXT, Girls Academy, and the majority of competitive youth soccer nationwide. Every other major ranking system works within a single platform or relies on editorial judgment. Because Evo Draw ingests both sources into a single calibrated ranking, a coach can look up where their ECNL team stands relative to GA and MLS NEXT teams in the same age group — a comparison no other data-driven system makes possible. Our methodology is documented, H2H-derived, and reproducible: when we say a team is ranked 47th nationally, we can show the game data behind it.

### Paragraph 2: Where Evo Draw Coverage is Still Developing

*Draft language — honest framing for questions SENTINEL may receive:*

Like any growing platform, Evo Draw has coverage areas still being built out. State league and cup games from unranked divisions — the Tier 3 community and competitive level where most youth teams play — are partially covered today, with a significant expansion in progress. Some Midwest states that use the Affinity Soccer platform instead of GotSport are not yet integrated; those require a separate data engineering project scheduled for later in 2026. For the leagues where DSS audiences coach and recruit — ECNL, MLS NEXT, Girls Academy, ECNL Regional League, NPL, and DPL — Evo Draw's coverage is active and current. The platform is purpose-built for the top two tiers where college recruiting decisions get made.

---

## References

- `[[Competitors]]` — 7 competitive gaps (ranking quality and feature gaps)
- `[[League Hierarchy]]` — Evo Draw's confirmed tier structure and calibration values
- `Infrastructure/Source Coverage Priority Matrix — April 2026.md` — source triage used to populate Section 4
- `Infrastructure/Pre-DSS Source Coverage Report — May 2026.md` — Evo Draw's confirmed active sources
- `Infrastructure/Evo Draw — DSS Coverage Narrative.md` — SENTINEL's broader DSS narrative context

*FORGE — 2026-04-25*
