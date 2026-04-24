---
title: Affinity Soccer Source Research
tags: [data-sources, research, affinity-soccer, usys, evo-draw]
created: 2026-04-24
updated: 2026-04-24
status: complete
author: FORGE
recommendation: deprioritize-until-post-DSS
---

# Affinity Soccer Source Research

**Researched by:** FORGE  
**Date:** 2026-04-24  
**Task:** `task-2026-04-24-affinity-soccer-source-research.md`  
**Format follows:** `SnapSoccer Source Research.md`

---

## 1. Platform Overview

**Affinity Soccer** (affinitysoccer.com) is a youth sports management platform owned by **Active Network / Blue Star Sports**. It is a direct competitor to GotSport in the state association management space — clubs and state associations use it to manage registrations, publish schedules, and record game results.

Unlike GotSport, which is dominant in tournament and league play nationally, Affinity Soccer is stronger in specific regional markets, particularly Midwest and Great Plains state associations that adopted it before GotSport's national expansion. It is sometimes called "Affinity Sports" in older documentation.

**Key distinction from SnapSoccer:** SnapSoccer is an event organizer that uses third-party platforms (SincSports) to host its results. Affinity Soccer IS the platform — it both manages and publishes results for the organizations on it. Building an Affinity ingestion pipeline means scraping affinitysoccer.com directly.

---

## 2. States and Leagues Using Affinity Soccer

### Confirmed: Illinois

**Illinois Youth Soccer Association (IYSA)** uses Affinity Soccer as its primary results platform. This was confirmed in the USYS State Cup Source Research (2026-04-23), and Illinois was explicitly excluded from the GotSport org-ID expansion task for this reason.

Illinois has approximately **50,000+ registered youth soccer players** — one of the top 5 states by player population.

### Likely Affinity States (Based on Historical Adoption Patterns)

| State/League | Platform Assessment | Notes |
|---|---|---|
| Illinois (IYSA) | **Confirmed: Affinity Soccer** | Largest confirmed non-GotSport state |
| Michigan (MSYSA) | **Likely: Affinity Soccer** | Midwest; Active Network historically strong in MI |
| Minnesota (MSYSA) | **Likely: Affinity Soccer** | Midwest; requires browser confirmation |
| Ohio (OSA) | **Possibly split** | OSA may use GotSport for some events, Affinity for state league — check |
| Indiana (ISYA) | **Likely: Affinity Soccer** | Midwest; relatively small state population |
| Wisconsin (WYSA) | **Possibly: Affinity Soccer** | Midwest; requires confirmation |
| Missouri (MSYSA) | **Unknown — possibly GotSport** | Adjacent states suggest mixed |

**States confirmed NOT on Affinity (on GotSport):** TX, FL, NY, NJ, WA, CO, CA, PA (documented in USYS State Cup Research).

### National Leagues on Affinity

- **ECNL (previously):** ECNL was historically on TGS AthleteOne / Affinity ecosystem before its June 2026 migration to GotSport. Post-June 2026, ECNL will be on GotSport.
- **US Club Soccer / NPL:** Some NPL regional divisions may use Affinity for specific states. The national NPL is primarily SincSports; state-level NPL divisions may vary.
- **US Youth Soccer (USYS) state leagues:** The states listed above run their state leagues on Affinity.

---

## 3. Public Data Exposure Assessment

### Public Results Pages
**Assessment: Limited and inconsistent.**

Affinity Soccer does have a public-facing results interface at `affinitysoccer.com`, but it is significantly less consistent than GotSport's public tournament pages. The platform is designed primarily as a club/state management tool, and public result visibility depends heavily on each individual organization's settings.

| Data Point | Assessment |
|---|---|
| Public results pages | Exist but inconsistently configured per org |
| URL pattern | `affinitysoccer.com/home/programs/{org_id}` or `affinitysoccer.com/tournament/{event_id}` — not confirmed without browser check |
| Authentication required | Sometimes — some orgs gate results behind a login |
| Data format | Server-rendered HTML (no public JSON API confirmed) |
| Stable IDs | Unknown — Affinity may use session-based pagination |

### API Access
**Assessment: No confirmed public API.**

Affinity Soccer has an internal API used by their mobile apps, but no publicly documented developer API. Unlike GotSport (which has a documented API endpoint at `/api/v1/team_ranking_data`), Affinity's API is not intended for external consumption. Any scraping would rely on HTML parsing, not structured API calls.

### Data Quality
**Assessment: Complete for events that are configured correctly; gaps elsewhere.**

Where Affinity does expose results publicly, the data quality is comparable to GotSport HTML pages: team names, scores, dates, and division names are present. However:
- Team name normalization is inconsistent (clubs may enter names differently than GotSport)
- No stable cross-platform team IDs (Affinity team IDs are internal to Affinity)
- Cross-system dedup with GotSport games would require fuzzy matching on team name + date + event name

---

## 4. Coverage Gap Estimate

### Illinois Alone

Illinois is the anchor state. With ~50,000 registered players:
- Estimated active competitive teams: 3,000–5,000
- State league games per season: ~30,000–50,000 (including all age groups, both genders, spring and fall)
- State cup games per season: ~5,000–8,000

**However:** Many Illinois ECNL and MLS NEXT teams play in GotSport-based events (national leagues) in addition to state league. These games are likely already in our DB via GotSport. The uniquely Affinity data is the **IYSA state league and state cup results** for teams that only compete within Illinois.

**Illinois-specific gap estimate: 8,000–20,000 games per season not currently in our DB.**

### Midwest States (Michigan, Minnesota, Indiana, Wisconsin)

If confirmed on Affinity, these four states collectively represent approximately:
- Michigan: ~35,000 registered players → ~5,000–12,000 games/season on Affinity
- Minnesota: ~30,000 players → ~4,000–10,000 games/season
- Indiana + Wisconsin: ~15,000 players each → ~2,000–5,000 each

**Combined Midwest gap estimate: 15,000–35,000 additional games/season** (if all confirmed on Affinity).

### Total Coverage Gap Assessment

| Scope | Estimated Games/Season | Confidence |
|---|---|---|
| Illinois alone | 8,000–20,000 | Medium-High (IL confirmed on Affinity) |
| + Midwest states (MI, MN, IN, WI) | 15,000–35,000 | Low-Medium (platform not confirmed for all) |
| **Total potential gap** | **20,000–55,000/season** | Low (many unknowns) |

**Classification: Major gap (>10,000 games/year, confirmed for IL, likely for several Midwest states).** This is larger than SnapSoccer's estimated 1,500–2,500 games/year.

---

## 5. USA Rank Coverage Comparison

**Does USA Rank have Illinois state league teams?**

Based on the competitive intel file, USA Rank claims 400+ sources and does not specifically call out Affinity Soccer by name. However:
- USA Rank's source family table does not list Affinity Soccer explicitly
- Their confirmed sources are GotSport-adjacent (SincSports, SnapSoccer, USYS)
- **If USA Rank lacks Illinois state league data as well, the gap is industry-wide** — not a competitive differentiator

**Assessment: Unknown.** USA Rank may have partial Affinity coverage via user-submitted events, or they may also lack Illinois state league data. Checking whether top Illinois clubs (e.g., Chicago Fire Juniors, Sockers FC) appear in USA Rank's rankings with state league game data would resolve this. If they appear only with GotSport/national-league games, USA Rank is also missing Affinity data.

---

## 6. Effort Estimate and Comparison

| Platform | Relative Difficulty | Why |
|---|---|---|
| GotSport (existing) | Baseline | Known API, established scraper, documented endpoint |
| SnapSoccer (via SincSports) | Low (+1 day) | Uses existing SincSports HTML parsing; just more tournament IDs |
| **Affinity Soccer** | **High (+5–7 days)** | New scraper from scratch; inconsistent public pages; no public API; org-by-org authentication behavior unknown |

Affinity is harder than SnapSoccer for two reasons:
1. No confirmed public API → must rely on HTML scraping of org-specific pages
2. Per-org configuration variance → what works for IYSA may not work for MSYSA without separate testing

---

## 7. Recommendation

**DEPRIORITIZE UNTIL POST-DSS.**

Reasoning:
1. **HTML-only scraping with per-org variance** is not a 1-session build. Estimating 5–7 days minimum, which exceeds the available pre-DSS window (DSS: May 16, 22 days out as of this writing).
2. **The gap may be smaller in practice** than the raw player count suggests, because Illinois ECNL and MLS NEXT teams (the elite teams most likely to matter for DSS) play in GotSport-based national events anyway. The state league teams we'd gain are lower-division regional teams, not the showcase demo tier.
3. **Unknown auth behavior** means risk of a scraper that works for one org and fails silently for others — exactly the kind of reliability problem to avoid pre-DSS.
4. **GotSport org-ID expansion** (already specced and ready to execute) gets us 20,000–42,000 games with a config change + existing code. That wins on the coverage-per-effort ratio.

**Post-DSS priority:**
- After May 16, Affinity Soccer should be Sprint 2 or Sprint 3 — after SnapSoccer and after GotSport org expansion is confirmed stable.
- Pre-work: manual browser investigation of IYSA's affinitysoccer.com page to confirm auth requirements and URL patterns. This can be done in 1 hour and should happen before any build commitment.
- If USA Rank also lacks Affinity data (to be verified), the urgency drops further — it's not a competitive differentiator gap, it's an industry-wide gap.

---

## 8. Deliverable Summary for Briefing

- **Confirmed on Affinity:** Illinois (IYSA)
- **Likely on Affinity:** Michigan, Minnesota, Indiana, Wisconsin (requires browser confirmation)
- **Gap estimate:** 8,000–20,000 games/season (IL alone); up to 55,000/season if Midwest states confirmed
- **Recommendation:** Deprioritize until post-DSS. Too much build uncertainty for 22-day window. GotSport org-ID expansion is the better pre-DSS move.
- **Pre-condition for future sprint:** 1-hour browser audit of IYSA's Affinity page to confirm auth/URL patterns before any code commitment.

---

## Related Notes

- [[Data Sources Overview]] — current 7 sources
- `Data Sources/USYS State Cup Source Research.md` — Affinity mentioned in platform map; IL excluded from GotSport expansion
- `Data Sources/SnapSoccer Source Research.md` — comparison reference; SincSports vs Affinity scraping complexity
- `Knowledge/USA Rank Competitive Intel.md` — does USA Rank show IL state league teams?
- `task-2026-04-24-affinity-soccer-source-research.md` — assigned task

---

*Researched by FORGE — 2026-04-24*
