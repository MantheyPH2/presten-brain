---
title: Stage 1 Coverage Map — Gap Analysis
type: forge-analysis
author: FORGE
date: 2026-04-29
status: filed
stage: stage-1
tags: [infrastructure, coverage, stage1, source-gaps, league, geographic, quality, forge]
---

# Stage 1 Coverage Map — Gap Analysis

> **Purpose:** Maps which leagues and states have coverage under Stage 1, which have gaps, and what the highest-priority Stage 2 org-ID and source additions would be. SENTINEL uses this for Stage 2 authorization scope decisions.

**Filed by:** FORGE — 2026-04-29  
**Basis:** GotSport Coverage Map (States and Regions), League Hierarchy, Non-GotSport Source Priority Recommendation, Stage 1 Config Final Review

---

## Section 1 — Methodology

**Coverage definition:** At least one org-ID in the Stage 1 config confirmed to run games for teams in a given league within a given state. Partial = at least one org-ID active for that state, but known gaps (e.g., state has 10 regions, only 2 org-IDs loaded). Not covered = no org-IDs active for that league in that state.

**Stage 1 scope as of May 1, 2026:** GotSport sources (gotsport_api ranked-team discovery + TYSA org-ID if confirmed), TGS sources (tgs_athleteone, tgs, tgs_rl for ECNL/ECNL RL). No state-association org-IDs beyond TYSA are active in Stage 1.

**Important caveat:** gotsport_api operates via ranked-team discovery (scraped from known team IDs in the DB). This provides partial coverage of elite teams in all states — but it does NOT provide comprehensive org-level coverage. Org-ID-level config is required to capture lower-tier and local competitive teams within a state. The gaps below refer to comprehensive coverage.

---

## Section 2 — League × State Coverage Matrix

Top leagues from [[League Hierarchy]] × states with meaningful team density.

| League | Tier | States Covered (Stage 1) | States Partial | States Not Covered | Coverage % (est.) |
|--------|------|--------------------------|---------------|-------------------|-------------------|
| **ECNL Girls** | 1 | National (via tgs_athleteone) | — | — | ~90% of ECNL clubs |
| **ECNL Boys** | 1 | National (via tgs_athleteone) | — | — | ~90% of ECNL clubs |
| **MLS NEXT** | 1 | National (via gotsport_api ranked teams) | — | — | ~85% of MLS NEXT clubs |
| **GA / GA ASPIRE** | 1 | National (via gotsport_api ranked teams) | — | — | ~80% of GA clubs |
| **ECNL RL** | 2 | National (via tgs_rl) | — | — | ~70% — tgs_rl staleness unverified |
| **NPL** | 2 | TX (via TYSA, pending confirmation); partial national via ranked-team discovery | All other states | All non-TX states at org-level | ~20% org-level; ~50% ranked-team discovery |
| **DPL** | 2 | Partial national via gotsport_api ranked teams | — | All states at org-level | ~30% ranked-team discovery |
| **USL Academy** | 2 | NOT YET — pending Stage 2 config add (browser session pending) | — | All states | 0% org-level; some clubs via ranked teams |
| **Elite 64** | 2 | NOT YET — pending browser session | — | All states | ~10% (top teams via ranked-team discovery) |
| **EDP** | 2 | NOT YET — pending Stage 2 config add (browser session pending) | — | NJ, NY, PA, CT, MA, MD, RI, DE, VA, VT | 0% org-level |
| **Pre-ECNL** | 3 | TX (via TYSA pending) | — | All other states | ~10% |
| **State Cups / State Leagues** | 3 | TX (via TYSA, pending confirmation) | — | All other 49 states | ~5% nationally (TX only) |
| **SnapSoccer leagues** | 2–3 | None — SnapSoccer build not yet started | — | Southeast, national | 0% |

**Summary:** FORGE has strong Tier 1 league coverage nationally via ranked-team discovery and TGS sources. Tier 2–3 league coverage is almost entirely absent outside TX at the org-ID level.

---

## Section 3 — Top Coverage Gaps (Priority Ranked)

| Rank | League | State/Region | Estimated Teams Missing | Source Needed | Priority |
|------|--------|-------------|------------------------|---------------|---------|
| 1 | State leagues, NPL, DPL (all tiers) | CA (NorCal + SoCal) | 2,000–5,000+ clubs | Cal North Soccer + Cal South Soccer (GotSport org-IDs — browser session) | **Critical** |
| 2 | State leagues, all competitive tiers | FL | 1,500–3,000+ clubs | FYSA (GotSport org-ID — browser session) | **Critical** |
| 3 | EDP, state leagues | NJ/NY/PA/CT/MA/MD | 800–2,000 clubs | EDP Soccer (GotSport org-ID — browser session) + ENYYSA/NJYSA/EPYSA | **High** |
| 4 | SnapSoccer (local competitive leagues) | Southeast (NC, SC, VA, GA, FL) | 5,000–7,000 games/year | SnapSoccer non-GotSport engineering build (May 17+) | **High** |
| 5 | State leagues, NPL | VA, MD, DC metro | 400–800 clubs | Virginia Youth Soccer Association (VYSA) — not yet in browser queue; research needed | **High** |
| 6 | State leagues | TX (outside TYSA scope) | Additional TX clubs not under TYSA umbrella | TYSA org-ID + potential additional TX org-IDs post-Stage-2 | **Medium** |
| 7 | USL Academy | National (90 clubs) | 90 clubs with pro pathway | USL Academy GotSport org-ID — browser session pending (Stage 2 config add) | **Medium** |
| 8 | Affinity Soccer (IL/WI/MN/IN) | Midwest | 5,000–15,000 games/year (IL alone) | Affinity Soccer non-GotSport build (June 2026 — access audit required first) | **Medium — deferred** |
| 9 | State leagues | CO, AZ | 600–1,200 clubs combined | Colorado Youth Soccer Assoc. + Arizona Youth Soccer Assoc. (GotSport; not yet in browser queue) | **Medium** |
| 10 | State leagues | OH, MI, WI, MN | Potentially 1,000–2,000 clubs (mixed platform) | OH likely GotSport; IL/MI/WI/MN likely Affinity Soccer | **Low-Medium — platform confirmation needed** |

---

## Section 4 — Non-GotSport League Coverage

Leagues or sources not on the GotSport platform.

| League / Source | Source Type | Current Status | Est. Games/Year | Estimated Launch |
|----------------|------------|----------------|----------------|-----------------|
| **SnapSoccer (SincSports)** | Non-GotSport scraper | Authorization response package filed (`Infrastructure/SnapSoccer — Authorization Response Package.md`). Pre-check protocol filed (`Infrastructure/SnapSoccer — SincSports Accessibility Pre-Check.md`). SENTINEL authorization needed by May 10. | 5,000–7,000 ongoing + 10,000–20,000 historical | May 17+ (post-DSS, post-SENTINEL authorization) |
| **Affinity Soccer (IL)** | Non-GotSport scraper | Research filed. Auth wall behavior unconfirmed. Browser access audit required before any build commitment. | 5,000–15,000 (IL) + additive (WI, MN, MO, IN) | June 2026 — after browser access audit |
| **NAL** | Platform TBD | Platform unconfirmed. If GotSport: 5-min config add. If non-GotSport: deferred indefinitely (< 1,000 games/yr estimated). | Unknown (< 1,000 est.) | As soon as browser platform check runs (fold into next browser session) |
| **Tournament Director** | Export pipeline (CSV import) | Scope undefined. Requires operator partnership or CSV export agreement. | Unknown — scope-dependent | TBD post-DSS |
| **USL Academy** | GotSport config add | Browser session pending. Classified as Stage 2 GotSport config add — exits this table on Stage 2 activation. | 2,000–5,000 | May 17–20 (Stage 2 go-live, pending browser session) |
| **EDP Soccer** | GotSport config add | Browser session pending. Stage 2 config add. | Unknown (Northeast league; likely 1,000–4,000) | May 17–20 (Stage 2 go-live, pending browser session) |

Reference: `Infrastructure/Non-GotSport Source Priority Recommendation.md` for full ranked build sequence.

---

## Section 5 — Stage 2 Expansion Recommendations

### Top 3 GotSport org-ID additions for highest coverage impact

**1. Cal North Soccer + Cal South Soccer (California)**
- States: CA (NorCal and SoCal — treated as 2 orgs, 1 state)
- Estimated impact: 2,000–5,000 additional clubs; highest single-state game volume in the US
- Why: California is the largest USYS state. Neither NorCal nor SoCal has any org-ID coverage. A single browser session yields both org-IDs. Adding CA to Stage 2 is the highest-leverage move available.
- Status: In browser session queue. Confirm org-IDs at `system.gotsport.com` → search "Cal North Soccer" and "Cal South Soccer."

**2. Florida Youth Soccer Association (FYSA)**
- States: FL
- Estimated impact: 1,500–3,000 additional clubs; FL is top-3 state by game volume
- Why: FL is zero-coverage today and one of the largest youth soccer states. FYSA is confirmed GotSport. One org-ID covers the entire state.
- Status: In browser session queue. Confirm org-ID at `system.gotsport.com` → search "Florida Youth Soccer."

**3. EDP Soccer (Northeast regional)**
- States: NJ, NY, PA, CT, MA, MD, RI, DE, VA, VT (10-state footprint)
- Estimated impact: 800–2,000 additional clubs across Northeast; fills the NE league games gap
- Why: EDP is a high-volume regional league with a 10-state footprint. A single EDP org-ID captures Northeast competitive league games not covered by state association org-IDs.
- Status: In browser session queue. Note: EDP is a league org-ID, not a state assoc — it supplements (rather than replaces) state assoc org-IDs for NY/NJ/PA.

### Non-GotSport addition recommendation
As noted in the Non-GotSport Source Priority Recommendation: **SnapSoccer is the highest-priority non-GotSport build** (May 17+). It fills the Southeast gap that GotSport org-ID expansion cannot address.

---

## References

- `Infrastructure/GotSport Coverage Map — States and Regions.md` — state coverage data and browser session entity list
- `Infrastructure/Non-GotSport Source Priority Recommendation.md` — non-GotSport build sequence
- [[League Hierarchy]] — tier definitions and calibration weights
- `Infrastructure/Source Gap Inventory — April 2026.md` — league-level gap inventory
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — confirmed and pending org-IDs
- `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` — Stage 2 entities pre-staged for config add

*FORGE — 2026-04-29*
