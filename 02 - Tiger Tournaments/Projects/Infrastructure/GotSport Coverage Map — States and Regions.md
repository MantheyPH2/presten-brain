---
title: GotSport Coverage Map — States and Regions
tags: [infrastructure, gotsport, org-ids, coverage, geographic, states, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: active — update when browser session completes and org-IDs are confirmed
---

# GotSport Coverage Map — States and Regions

> Geographic view of current GotSport crawl coverage by US state. Complements the Org-ID Master Reference (entity-centric) with a regional picture: which states are covered, which are gaps, and which would be newly covered if all 11 browser-session entities are confirmed.

---

## Section 5: Coverage Summary Block

```
Last updated: 2026-04-26
States with confirmed GotSport org-ID coverage: 1 (TX — Stage 1 config, pending first run)
States with coverage if all 11 browser entities confirmed: +8 states (CA, FL, NY, NJ, WA, PA, + national leagues)
States with zero GotSport coverage: 43 (see Section 2)
Browser session entities awaiting confirmation: 11
States that would be newly covered if all 11 browser entities confirmed: 7 additional states (CA NorCal, CA SoCal → 1 state, FL, NY, NJ, WA, PA)
National league entities pending (USL Academy, EDP, Elite 64, NAL): coverage is national, not state-specific
```

---

## Section 1: Current Coverage — State-Level Table

Org-IDs currently in or confirmed for the crawl config. **As of 2026-04-26: TX is the only active configuration; first run has not yet executed.**

| State | Org Name | Type | Stage | Org-ID Status | Game Volume Estimate |
|-------|----------|------|-------|---------------|---------------------|
| TX | Texas Youth Soccer Association (TYSA) | USYS State Association | Stage 1 pilot | In config — pending browser confirmation of exact org_id before first run | High — TX is the largest USYS state by game volume |

> [!warning] No confirmed active coverage yet
> The TX crawl configuration is in place but the org_id must be confirmed via browser lookup before the Stage 1 run executes. All other states have zero org-ID-level coverage as of this writing. The pipeline does ingest GotSport games via ranked-team discovery (teams discovered through rankings), but org-level config provides comprehensive coverage — not just ranked teams.

---

## Section 2: Coverage Gaps — States with Zero Org-ID Presence

States are grouped by region. Note: MLS NEXT (national) and ECNL (national) are ingested via separate pipelines and provide partial coverage of elite teams in all states; the gaps below refer specifically to GotSport state association coverage, which captures recreational-to-competitive teams not appearing in elite league schedules.

### Northeast

| State | Notes |
|-------|-------|
| ME | No GotSport org in config. Maine Soccer (MYSA) is a smaller-volume state. |
| NH | No GotSport org in config. |
| VT | No GotSport org in config. |
| MA | No GotSport org in config. MA is a high-volume state (500+ clubs). Priority post-browser-session. |
| RI | No GotSport org in config. Low volume. |
| CT | No GotSport org in config. CT is a moderate-volume Northeast state. |
| NY (upstate) | ENYYSA (Eastern NY) covers NYC metro. Upstate NY (WNYSA) is a gap. ENYYSA pending browser confirmation. |
| PA (western) | EPYSA (Eastern PA) covers Philadelphia metro — pending browser session. Western PA (WPYSA / Pittsburgh metro) is a separate org and remains a gap. |
| DE | No GotSport org in config. Very low volume. |
| MD | No GotSport org in config. Moderate volume — proximity to DC/DMV market. |
| DC | No GotSport org in config. Low volume (small geography). |

### Southeast

| State | Notes |
|-------|-------|
| VA | No GotSport org in config. Virginia is a high-volume state (Blue Ridge, ODSL). Priority gap. |
| NC | No GotSport org in config. Large youth soccer market. |
| SC | No GotSport org in config. Moderate volume. |
| GA | No GotSport org in config (GA ASPIRE is classified via event_tier, not org-ID). GA Soccer is a major GotSport org — research pending. |
| FL | FYSA pending browser confirmation. **High priority — FL is one of the top 3 states by game volume.** |
| AL | No GotSport org in config. Likely GotSport platform. Moderate volume. |
| MS | No GotSport org in config. Low volume. |
| TN | No GotSport org in config. Moderate volume. |
| KY | No GotSport org in config. Moderate volume. |
| WV | No GotSport org in config. Low volume. |

### Midwest

| State | Notes |
|-------|-------|
| OH | No GotSport org in config. Large state, high game volume. |
| IN | **Affinity Soccer platform** — confirmed not on GotSport. Deferred post-DSS. |
| IL | **Affinity Soccer platform** — confirmed not on GotSport. Largest non-GotSport gap. 5,000–15,000 games/year estimated. |
| MI | **Likely Affinity Soccer** — browser confirmation pending; deprioritized. |
| WI | **Likely Affinity Soccer** — deprioritized. Low volume. |
| MN | **Likely Affinity Soccer** — deprioritized. |
| IA | No GotSport org in config. Low–moderate volume. |
| MO | No GotSport org in config. Likely Affinity Soccer or GotSport — research needed. |
| KS | No GotSport org in config. Low volume. |
| NE | No GotSport org in config. Low volume. |
| SD | No GotSport org in config. Very low volume. |
| ND | No GotSport org in config. Very low volume. |

### South Central

| State | Notes |
|-------|-------|
| TX | **TYSA in config — Stage 1.** Highest-priority expansion state. |
| OK | No GotSport org in config. Moderate volume — teams travel to TX leagues. |
| AR | No GotSport org in config. Low–moderate volume. |
| LA | No GotSport org in config. Moderate volume. |

### Mountain / Plains

| State | Notes |
|-------|-------|
| CO | No GotSport org in config. Large youth soccer market — Colorado Soccer Assoc. High priority post-Stage-2. |
| WY | No GotSport org in config. Very low volume. |
| MT | No GotSport org in config. Very low volume. |
| ID | No GotSport org in config. Low volume. |
| UT | No GotSport org in config. Moderate volume — large youth programs. |
| NV | No GotSport org in config. Moderate volume (Las Vegas metro). |
| NM | No GotSport org in config. Low–moderate volume. |
| AZ | No GotSport org in config. Large youth soccer market. High priority. |

### West

| State | Notes |
|-------|-------|
| CA (NorCal) | Cal North Soccer pending browser confirmation. **High priority — CA NorCal is one of the largest USYS states.** |
| CA (SoCal) | Cal South Soccer pending browser confirmation. **High priority — CA SoCal is one of the largest USYS states.** |
| OR | No GotSport org in config. Moderate volume. |
| WA | Washington Youth Soccer (WSYSA) pending browser confirmation. High volume Pacific Northwest state. |
| AK | No GotSport org in config. Very low volume. |
| HI | No GotSport org in config. Low volume. |

---

## Section 3: Browser Session Coverage Impact

Pre-populated from the Browser Lookup Session Prep Package. Update after browser session completes.

| # | Entity (from Prep Package) | Type | State Represented | Fills a State Gap? | Status | Priority if Confirmed |
|---|---------------------------|------|-------------------|--------------------|--------|-----------------------|
| 1 | USL Academy | National League | National (all states) | No — league teams already partially covered via ranked-team discovery | pending-browser-lookup | High — adds 90 professional-pathway clubs |
| 2 | EDP Soccer | Regional League | NJ, NY, PA, CT, MA, MD, RI, DE, VA, VT (Northeast) | Partial — adds EDP-specific club games in states not covered by state-assoc org | pending-browser-lookup | High — fills NE league games gap |
| 3 | Elite 64 | National League | National (showcase events) | No — national showcase format; adds high-level showcase results | pending-browser-lookup | Medium — high-visibility games but low frequency |
| 4 | NAL | National League (platform unconfirmed) | Unknown | Unknown until platform confirmed | pending-browser-lookup | Low — DEFER until platform confirmed |
| 5 | Cal North Soccer (CA-NorCal) | USYS State Assoc | CA (NorCal) | **Yes — CA NorCal is a complete state gap** | pending-browser-lookup | **Very High** |
| 6 | Cal South Soccer (CA-SoCal) | USYS State Assoc | CA (SoCal) | **Yes — CA SoCal is a complete state gap** | pending-browser-lookup | **Very High** |
| 7 | Florida Youth Soccer (FYSA) | USYS State Assoc | FL | **Yes — FL is a complete state gap** | pending-browser-lookup | **Very High** |
| 8 | Eastern NY Youth Soccer (ENYYSA) | USYS State Assoc | NY (metro/Eastern) | Partial — covers NYC metro; upstate NY (WNYSA) remains a gap | pending-browser-lookup | High |
| 9 | NJ Youth Soccer (NJYSA) | USYS State Assoc | NJ | **Yes — NJ is a complete state gap** | pending-browser-lookup | High |
| 10 | Washington Youth Soccer (WSYSA) | USYS State Assoc | WA | **Yes — WA is a complete state gap** | pending-browser-lookup | High |
| 11 | Eastern PA Youth Soccer (EPYSA) | USYS State Assoc | PA (Eastern) | Partial — covers Philadelphia metro; Western PA remains a gap | pending-browser-lookup | High |

> [!note] Update instructions
> After browser session runs: change `pending-browser-lookup` to `confirmed-[org_id]` for found entities. Change to `not-found` if 2–3 search attempts failed. Move confirmed entities to Section 1 of the Org-ID Master Reference.

---

## Section 4: Recommended Stage 2 Geographic Priorities

States representing the highest-value Stage 2 targets based on estimated game volume and current zero-coverage status.

| Rank | State | Estimated Game Volume | Reason for Priority | Likely GotSport Entity to Target |
|------|-------|----------------------|--------------------|---------------------------------|
| 1 | CA (NorCal + SoCal combined) | Very High — California is the largest USYS state | Cal North + Cal South cover ~400+ clubs combined. Confirmed GotSport platform. Browser session will yield org_ids. | Cal North Soccer + Cal South Soccer (2 separate orgs) |
| 2 | FL | Very High — 3rd or 4th largest USYS state | FYSA covers statewide. Confirmed GotSport platform. Confirmed in browser session. | Florida Youth Soccer Association (FYSA) |
| 3 | VA | High — large blue-chip suburban market | Virginia Soccer covers major northern VA market (DC suburbs, soccer-dense population). Not yet in browser session queue — needs research. | Virginia Youth Soccer Association (VYSA) — research needed |
| 4 | CO | High — growing major market | Colorado Soccer Association. Not yet in browser session queue. | Colorado Youth Soccer Association (CYSA) — research needed |
| 5 | AZ | High — large youth market | Arizona Youth Soccer Association (AYSA). Hot weather → year-round play → high annual game count. | Arizona Youth Soccer Association (AYSA) — research needed |

**Geographic gap summary:**

| Region | Assessment |
|--------|------------|
| Northeast | Partially covered via browser-session entities (NY, NJ, PA-East, EDP). MA, CT, VA, MD remain gaps. |
| Southeast | FL pending browser session. GA, NC, VA are high-priority uncovered states. |
| Midwest | IL/WI/MN/IN confirmed Affinity Soccer — not GotSport. OH is a major uncovered GotSport state. |
| South Central | TX in Stage 1. OK, LA, AR are uncovered adjacents with cross-border travel implications. |
| West | CA and WA pending browser session — highest priority in region. AZ, CO are major uncovered markets. |
| Mountain/Plains | Low priority — low-volume states. CO is the exception (high priority). |

---

## References

- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — entity-level status for all org-IDs
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — 11 entities this map pre-populates in Section 3
- `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` — Affinity Soccer states context
- `Infrastructure/Source Gap Inventory — April 2026.md` — league-level gap inventory

*FORGE — 2026-04-26*
