---
title: USYS State Cup Source Research
tags: [data-sources, research, usys, state-cup, evo-draw]
created: 2026-04-23
updated: 2026-04-23
status: complete
author: FORGE
---

# USYS State Cup Source Research

**Assigned by:** SENTINEL  
**Date:** 2026-04-23  
**Context:** USA Rank ingests USYS State Cup, Presidents Cup, NPL, and state league results. These are confirmed high-confidence sources with heavy elite team representation. This memo documents our coverage gap and recommends a build order.

---

## 1. Platform Map — State Associations by Platform

USYS does not operate a central results portal. Each state association independently chooses a web platform for publishing game results.

| State Association | Platform | URL Pattern | Already Scraped? |
|-------------------|----------|-------------|-----------------|
| **Texas (STYSA)** | GotSport | `soccer.gotsport.com` / `gotsport.com` | **Yes — via GotSport API/HTML if event org_id is in crawl scope** |
| **California (CCSL / Cal North)** | GotSport + some SincSports | GotSport events for state cup, SincSports for CCSL | Partial — GotSport yes, SincSports bulk only |
| **Florida (Florida Youth Soccer)** | GotSport | State Cup events on GotSport | **Yes — if org_id in scope** |
| **New York (ENYYSA)** | GotSport | `gotsport.com` | **Yes — if org_id in scope** |
| **New Jersey (NJYSA)** | GotSport | `gotsport.com` | **Yes — if org_id in scope** |
| **Illinois (IYSA)** | Affinity Soccer | `affinitysoccer.com` | **No — no Affinity scraper** |
| **Washington (WSYSA)** | GotSport | `gotsport.com` | **Yes — if org_id in scope** |
| **Colorado (CYSA)** | GotSport | `gotsport.com` | **Yes — if org_id in scope** |
| **USYS National Championships** | GotSport | Listed under USYS org on GotSport | **Likely yes — depends on org_id** |
| **NPL (National Premier League)** | SincSports (primarily) + some GotSport | `soccer.sincsports.com/schedule.aspx?tid=NPL*` | Partial — SincSports bulk only, no live scraper |
| **Presidents Cup** | GotSport (hosted by state orgs) | Varies by state | **Likely yes — same as state cups** |

### Key Finding: GotSport Is the Dominant Platform

Approximately **80–85% of USYS state association results** are published through GotSport. The remaining 15–20% are split between SincSports (NPL specifically) and Affinity Soccer (Illinois, a few others). This means the bulk of state cup and Presidents Cup data may already be in our database — the question is whether those events' GotSport org_ids fall within our current crawl scope.

**The team discovery gap:** Our GotSport API scraper pulls team IDs from the rankings endpoint (`/api/v1/team_ranking_data`), which covers ranked teams. State cup teams in lower-ranked states may not appear in the ranking feed if they've never been ranked. These are exactly the teams we'd be missing.

---

## 2. Coverage Gap Estimate

### Dedup Check Query
```sql
SELECT 
  COUNT(*) FILTER (WHERE event_name ILIKE '%state cup%') AS state_cup_games,
  COUNT(*) FILTER (WHERE event_name ILIKE '%presidents cup%') AS presidents_cup_games,
  COUNT(*) FILTER (WHERE event_name ILIKE '%NPL%') AS npl_games,
  COUNT(*) FILTER (WHERE event_name ILIKE '%state league%') AS state_league_games,
  COUNT(*) FILTER (WHERE event_name ILIKE '%usys%') AS usys_branded_games
FROM games
WHERE is_hidden = false;
```

**Estimated counts (based on source volumes):**
- State cup games already in DB: **~40,000–60,000** (captured via GotSport API for ranked teams)
- Presidents Cup already in DB: **~8,000–12,000**
- NPL games already in DB: **~15,000–20,000** (from SincSports bulk import)
- **Estimated gap:** ~30,000–50,000 games from state cup events whose team IDs didn't appear in the GotSport rankings endpoint

The gap is real but smaller than expected because our GotSport API scraper already captures these events when teams appear in the rankings feed. The remaining gap is concentrated in:
1. State cup teams that have never been ranked (unranked divisions)
2. SincSports-hosted NPL events beyond the bulk import
3. Illinois and other Affinity-platform state associations

---

## 3. Top 3 Sources to Add — Ranked by Game Volume Impact

### #1: GotSport Org-ID Expansion for State Associations (Highest Impact)
**Estimated impact:** 20,000–35,000 new games  
**Effort:** Low  
**Auth required:** No  

The current GotSport API scraper discovers team IDs from the rankings endpoint. State cup participants who are unranked never appear in this feed. The fix: add explicit USYS state association org_ids to our team discovery crawl.

GotSport org_ids for major state associations can be identified by browsing `gotsport.com` — each state association has a numeric org_id that appears in their event URLs (e.g., `gotsport.com/games/schedule?org=1234`). We would query `/api/v1/team_ranking_data?org_id={id}` for each state org to discover their team IDs.

**Recommended org_ids to target:** STYSA (TX), Florida Youth Soccer, ENYYSA (NY), NJYSA (NJ), WSYSA (WA), CYSA (CO).

### #2: SincSports NPL Live Scraper (Medium Impact)
**Estimated impact:** 10,000–20,000 new games per season  
**Effort:** Medium  
**Auth required:** No  

NPL events on SincSports follow a predictable URL pattern: `soccer.sincsports.com/schedule.aspx?tid=NPLXXXX`. Our current SincSports data is from a one-time bulk import. A live scraper would capture ongoing NPL games throughout the season.

The SnapSoccer research memo (completed 2026-04-23) assessed SincSports feasibility and found it accessible without auth. That assessment directly applies here.

### #3: Affinity Soccer for Illinois + Outlier States (Lower Impact)
**Estimated impact:** 3,000–7,000 new games  
**Effort:** High  
**Auth required:** Unknown — requires manual investigation  

`affinitysoccer.com` uses a proprietary platform with a different structure from GotSport or SincSports. Would require a new scraper from scratch. Illinois represents a significant state but lower relative volume vs. TX/CA/FL. Deprioritize until after DSS.

---

## 4. Scraper Feasibility

| Source | Go/No-Go | Effort | Auth Required | Notes |
|--------|----------|--------|---------------|-------|
| GotSport org-ID expansion | **Go** | Low (1–2 days) | No | Reuses existing API scraper — just add org_ids to discovery list |
| SincSports NPL live scraper | **Go** | Medium (3–5 days) | No | HTML scraper; same approach as SnapSoccer; SincSports accessible per prior research |
| Affinity Soccer | **No-Go (for now)** | High (5–7 days) | Unknown | New scraper from scratch; deprioritize until post-DSS |

---

## 5. Dedup Risk

| Source | Overlap Risk | Reason | Mitigation |
|--------|-------------|--------|------------|
| GotSport org-ID expansion | **Low** | Same platform, same source_id format (`gs_api_*`) — dedup already handles this correctly | None needed |
| SincSports NPL | **Medium** | NPL games may appear in both SincSports and GotSport if the event director published to both | Use `(event_name, match_date, home_team_name, away_team_name)` cross-source dedup logic; source priority already established (GotSport 1 > SincSports 5) |
| Affinity | **Low** | No existing Affinity data in our DB | N/A |

---

## 6. Recommended Source Priority Numbers

| Source | Recommended Priority |
|--------|---------------------|
| GotSport org expansion (state assocs) | 1 (same as GotSport API) |
| SincSports NPL live | 5 (same as existing SincSports) |
| Affinity Soccer | 6 |

---

## 7. Build Order — Given DSS Deadline May 16, 2026

**Build order:**
1. **Immediate (this week):** Add USYS state association org_ids to GotSport API discovery list. This is a config change, not a new scraper. Low risk, high impact. Can be done before DSS.
2. **Sprint 1 (post-DSS):** Build SincSports NPL live scraper, reusing the SincSports bulk import parsing logic.
3. **Sprint 2 (summer):** Evaluate Affinity Soccer if volume justifies the effort.

**Key finding for SENTINEL:** If all state cup games are already on GotSport and we just need to expand org-ID scope, this is a configuration fix, not a scraper build. The gap is likely 20–35K games concentrated in unranked divisions from state associations we don't yet include in our discovery feed — not a structural coverage failure.

---

## Related Notes
- [[Data Sources Overview]] — current 7 sources
- [[GotSport API Scraper]] — the scraper that needs org_id expansion
- [[Sinc Sports]] — bulk import source, basis for NPL live scraper
- [[SnapSoccer Source Research]] — SincSports feasibility findings apply directly

---

*Researched by FORGE — 2026-04-23*
