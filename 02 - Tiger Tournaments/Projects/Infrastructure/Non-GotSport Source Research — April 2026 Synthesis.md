---
title: Non-GotSport Source Research — April 2026 Synthesis
tags: [infrastructure, sources, gap-analysis, coverage, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: complete
task: task-2026-04-26-non-gotsport-source-research-synthesis
---

# Non-GotSport Source Research — April 2026 Synthesis

> [!info] Purpose
> Consolidates all non-primary-pipeline source research from April 2026 into a single go/no-go reference. "Non-GotSport" here means sources that are NOT the USYS state org-ID expansion track — including GotSport sources that require separate org_id confirmation and non-GotSport platforms entirely. SENTINEL uses this document to determine Stage 2 and Stage 3 source queue candidates.

**Sources covered:** EDP, USL Academy, Elite 64, NAL, SnapSoccer, Affinity Soccer  
**Prepared by:** FORGE — 2026-04-26  
**Research completed:** 2026-04-24 (vault research phase). Org_id confirmation pending Presten browser session.

---

## Section 1: Research Summary Table

| Source | Platform | Score (Priority Matrix) | Data Available | Org-ID or API Exists | FORGE Recommendation | Due Diligence Status |
|--------|----------|:---:|---|---|---|---|
| **EDP (Eastern Development Program)** | GotSport | 7 | Yes — partial ingestion via ranked-team discovery likely already active | Yes — EDP Soccer is a confirmed GotSport organization | **PROCEED** — pending org_id confirmation from browser session | Vault research complete. Org_id not confirmed. Presten browser lookup required at `system.gotsport.com`. |
| **USL Academy** | GotSport | 6 | Unknown — no recent ingestion confirmed | High probability — USL Soccer is a GotSport organization | **PROCEED** — pending org_id confirmation from browser session | Vault research complete. Org_id not confirmed. Presten browser lookup required. |
| **Elite 64** | GotSport (likely — via US Club Soccer) | 6 | Unknown — game volume not confirmed | High probability — affiliated with US Club Soccer (primary GotSport org) | **CONDITIONAL PROCEED** — if GotSport org_id confirmed via browser session; defer if not on GotSport | Vault research complete. Org_id not confirmed. Presten browser lookup required. Game volume unknown (national championship/showcase format). |
| **NAL (North American League)** | Unknown — research required | 3 | Unknown | Unknown | **DEFER** — platform not confirmed; volume estimated < 1,000 games/year | Vault research complete. Next step: GotSport search + NAL website identification. No action until platform confirmed. |
| **SnapSoccer** | SincSports (`soccer.sincsports.com`) | 4 | Yes — fully public HTML, no auth required | N/A (HTML scrape) | **NO-GO for DSS sprint** — post-DSS trigger conditions documented | Design complete. Go/no-go decision filed 2026-04-24. |
| **Affinity Soccer (IL + Midwest)** | Affinity Soccer (proprietary platform) | 6 (IL alone) | Unknown — no public API or scrape path confirmed | No — requires new scraper | **DEFER** — post-DSS. Hard blocker: no existing scraper. | Research complete. See `Data Sources/Affinity Soccer Source Research.md`. No action before June 2026. |

---

## Section 2: Source Profiles

---

### EDP (Eastern Development Program)

- **Platform:** GotSport API
- **Games in scope:** ~150+ clubs, multi-state Northeast (NJ, NY, PA, CT, MA, MD, RI, DE, VA, VT), U8–U19. Estimated 3,000–8,000 games/season (regional full-season league with structured divisions).
- **Data format:** Structured — GotSport API (same as current primary pipeline)
- **Access method:** GotSport org_id → add to crawl config (5-minute Presten config change once org_id confirmed)
- **Known blockers:** Org_id not yet confirmed. Requires browser lookup at `system.gotsport.com` → search "Eastern Development Program" or "EDP Soccer."
- **Implementation complexity:** LOW — same config-add path as USYS state expansion. EDP is a known GotSport organization. Partial coverage may already exist via ranked-team discovery; org-level config would fill in unranked divisions.
- **Priority Matrix rank:** 4 (Score 7 — tied with USYS/NPL/DPL)
- **FORGE notes:** EDP's northeastern footprint covers teams that are likely represented in the top-500 rankings. Adding the org_id after browser confirmation requires no engineering work — it is a copy-paste config addition. DB verification query: `SELECT COUNT(*) FROM games WHERE event_name ILIKE '%EDP%' OR event_name ILIKE '%Eastern Development%'` will show whether ranked-team discovery is already pulling some EDP games. If count > 500, status upgrades to Partial; if < 100, org_id addition has significant coverage impact.

---

### USL Academy

- **Platform:** GotSport API
- **Games in scope:** 90 clubs with professional pathway (MLS/USL affiliate academies), U13–U19. Professional pathway teams at calibration 70 (between NPL and GA tiers). Estimated 2,000–5,000 games/season.
- **Data format:** Structured — GotSport API
- **Access method:** GotSport org_id → add to crawl config (5-minute config change once confirmed)
- **Known blockers:** Org_id not confirmed. USL Soccer is a major GotSport organization — high probability the org_id exists. Requires browser session at `system.gotsport.com` → search "USL Academy" or "United Soccer League Academy."
- **Implementation complexity:** LOW — identical path to EDP and USYS state expansion
- **Priority Matrix rank:** 6 (Score 6)
- **FORGE notes:** USL Academy teams have a professional pathway identity (USL affiliates, similar to MLS NEXT in status). At DSS, being able to say "USL Academy coverage confirmed" strengthens the competitive claim vs USA Rank. The 90-club footprint is focused on Boys — Girls equivalent pipeline (USL Super Y?) would be a separate research item.

---

### Elite 64

- **Platform:** GotSport (likely — via US Club Soccer affiliation)
- **Games in scope:** Unknown. Elite 64 is a national championship/showcase format, not a full-season league. Estimated 500–2,000 games/year (national-scale event with 64-team brackets). Event count is low; game volume per event is high.
- **Data format:** Structured — GotSport API (if on GotSport)
- **Access method:** GotSport org_id via US Club Soccer or a dedicated Elite 64 org → add to config. If not on GotSport: Elite 64 official website → determine result publication platform.
- **Known blockers:** (1) Org_id not confirmed. (2) Game volume unknown — national championship format means events are infrequent (annual or seasonal). Volume may not justify a separate org_id config entry if games are sparse.
- **Implementation complexity:** LOW if GotSport org_id confirmed; MEDIUM if on a separate platform (requires platform assessment before engineering scoping)
- **Priority Matrix rank:** 7 (Score 6)
- **FORGE notes:** The GO case (org_id confirmed) is trivially easy — same config add as USL Academy/EDP. The NO case (not on GotSport) requires a separate platform assessment before any engineering decision. National championship format means Elite 64 events may appear under US Club Soccer's org_id rather than a dedicated Elite 64 org — browser search should check both.

---

### NAL (North American League)

- **Platform:** Unknown
- **Games in scope:** Unknown. Tier 2 in the League Hierarchy alongside USL Academy and EDP, but actual game volume not confirmed. Estimated < 1,000 games/year based on league size heuristics (low national footprint vs. established regional leagues).
- **Data format:** Unknown
- **Access method:** Unknown. Required first step: (1) search GotSport for "North American League" or "NAL Soccer"; (2) if not found, identify NAL official website and check result publication format.
- **Known blockers:** Platform entirely unknown. Cannot estimate implementation complexity or game volume without first confirming the platform.
- **Implementation complexity:** UNKNOWN — depends entirely on platform. If GotSport: LOW. If independent platform: HIGH (new scraper required).
- **Priority Matrix rank:** 10 (Score 3)
- **FORGE notes:** NAL is the lowest-confidence source in this synthesis. The combination of unknown platform + unknown volume means FORGE cannot recommend any engineering investment until at minimum a 30-minute GotSport search + website review is complete. If volume is confirmed < 1,000 games/year and platform is non-GotSport, deprioritize indefinitely.

---

### SnapSoccer

- **Platform:** SincSports (`soccer.sincsports.com`) — same underlying platform as the existing bulk Sinc import (34,961 games already ingested)
- **Games in scope:** 1,500–2,500 games/year (ongoing); 10,000–20,000 historical (one-time catch-up). Southeast focus: Alabama, Florida, Georgia, Louisiana.
- **Data format:** Semi-structured — server-rendered HTML (ASP.NET WebForms). All scores in initial HTTP response; no JavaScript execution required. Fully public, no authentication.
- **Access method:** HTML scrape of `soccer.sincsports.com/schedule.aspx?tid={TID}` — 15–30 event pages, entire crawl < 1 minute per run. TID discovery from `snapsoccer.com/events/`.
- **Known blockers:** MVP build estimate 5–9 hours (test run) / 8–12 hours (full production). Ongoing annual volume (1,500–2,500) doesn't justify DSS sprint vs Org-ID expansion (20,000+ games, config-only).
- **Implementation complexity:** MEDIUM — new ingestion script required, but can adapt existing Sinc Sports parser. Dedup is clean (SincSports `(source, source_id)` key; no cross-source overlap with GotSport).
- **Priority Matrix rank:** 9 (Score 4)
- **Decision filed:** NO-GO for DSS sprint (May 16 deadline). Design complete in `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md`.
- **Post-DSS trigger conditions:**
  1. USYS org-ID expansion confirmed in production
  2. Presten has a 6–8 hour session available (May 17+ earliest)
  3. Browser test confirms `schedule.aspx` HTML table structure matches parsing spec
  4. Existing Sinc Sports parser confirmed reusable (if yes: effort drops to 4–6 hours test / 6–8 hours production)

---

### Affinity Soccer (Illinois + Midwest States)

- **Platform:** Affinity Soccer (proprietary — independent of GotSport and SincSports)
- **Games in scope:** Illinois alone: estimated 5,000–15,000 games/year. Wisconsin, Missouri, Minnesota may also use Affinity Soccer (research incomplete). Combined Midwest gap could be 15,000–40,000 games/year.
- **Data format:** Unknown — no public API confirmed. May require direct outreach or scraping.
- **Access method:** New scraper required — no existing integration path. See `Data Sources/Affinity Soccer Source Research.md` for platform details.
- **Known blockers:** (1) No existing Affinity Soccer scraper. (2) No confirmed public access path for game results. (3) Illinois represents the largest single non-GotSport gap in the pipeline, but the engineering investment is weeks, not hours.
- **Implementation complexity:** HIGH — new platform, new scraper, no existing code to reuse
- **Priority Matrix rank:** 8 (Score 6 for Illinois alone; Score 3 for other Affinity states with incomplete research)
- **FORGE notes:** Despite high volume, Affinity Soccer cannot be acted on before DSS. The Priority Matrix score (6) reflects volume and impact, not feasibility — the Effort score is 1 (Hard). Recommended: post-DSS, assign as a standalone research + engineering sprint. If Illinois volume is confirmed at 5,000+ games, this becomes the highest-priority post-DSS source expansion.

---

## Section 3: Stage 2 Queue Recommendation

The following sources are recommended for Stage 2 or Stage 3 queue consideration. Sources are ranked by implementation readiness and coverage impact.

### Tier A — Authorize Immediately After Browser Session (Post-Stage-1 Config Adds)

These sources require only org_id browser confirmation + config addition. Zero engineering work. Can be executed in the same session as the browser lookup.

| Priority | Source | Why Now | Prerequisites | Estimated Time |
|:---:|---|---|---|---|
| **1** | **EDP** | Score 7. Known GotSport org. ~150+ clubs in NE. Partial coverage likely already active — org_id addition fills unranked divisions. | Org_id confirmed in browser session | 5 min (config add) |
| **2** | **USL Academy** | Score 6. Professional pathway teams. 90 clubs. Browser session will likely confirm GotSport presence. | Org_id confirmed in browser session | 5 min (config add) |
| **3** | **Elite 64** | Score 6. If org_id found, same config-add path. National championship format adds showcase events not otherwise covered. | Org_id confirmed in browser session; game volume > 500/year | 5 min (config add) |

**Combined Stage 2 impact (if all three confirmed):** 5,000–13,000 additional games/year. 15–20 minutes of Presten config time after browser session completes.

---

### Tier B — Defer Pending Prerequisites

These sources have blockers that prevent Stage 2 queue entry until conditions are met.

| Source | Blocker | Revisit Condition |
|---|---|---|
| **NAL** | Platform unknown. Volume unknown. | After 30-min GotSport search + website review confirms platform. If GotSport: same Tier A path. If not: re-score after volume estimate. |
| **SnapSoccer** | NO-GO for DSS. Volume insufficient vs Org-ID expansion ROI. | Post-DSS (May 17+). After USYS expansion stable + Presten has 6–8 hour session. Post-DSS trigger checklist in design doc. |
| **Affinity Soccer** | New scraper required. Weeks of engineering. No confirmed access path. | Post-DSS (June+). After Illinois research confirms accessible data format and outreach/access path is defined. |

---

## Section 4: Open Research Items

The following sources from the [[Source Coverage Priority Matrix — April 2026]] have not yet been fully researched and are not covered in this synthesis:

| Source | Score | Status | Why Deferred | Recommend Research Now? |
|---|:---:|---|---|---|
| **NAL platform identification** | 3 | Incomplete | FORGE could not confirm platform from vault research alone. Requires browser/website check. | **Yes** — 15-minute check. Include in browser lookup session alongside EDP/USL Academy/Elite 64 org_id confirmation. No engineering commitment required until platform confirmed. |
| **GA Boys ASPIRE classification** | (not scored separately) | Potential gap | GA ASPIRE fix targets Girls. Boys GA ASPIRE event classification correctness not verified. | **Yes** — check as part of FM1 classifier audit (already scoped in `task-2026-04-26-ga-aspire-pipeline-classifier-audit.md`). |
| **Other Affinity states (WI, MN, MO)** | 3 | Incomplete | Research incomplete. Platform not confirmed — likely same as Illinois (Affinity Soccer) but not verified. | **No** — defer until Illinois/Affinity Soccer post-DSS sprint. Unlikely to change the decision given Illinois takes priority. |
| **MLS NEXT dedup overlap with GotSport API** | (not scored as gap) | Known limitation | 27,635 MLS NEXT source games may partially duplicate GotSport API games. Dedup pipeline handles post-ingest. | **No** — not a source gap; dedup pipeline is the correct handling mechanism. No research needed. |

---

## FORGE Assessment

As of April 26, the non-primary-pipeline source landscape breaks down cleanly:

- **3 sources are 5-minute config adds** after browser session: EDP, USL Academy, Elite 64 (if GotSport confirmed)
- **1 source is NO-GO** for DSS with clear post-DSS trigger conditions: SnapSoccer
- **1 source is deferred** with high long-term volume but hard engineering blocker: Affinity Soccer (IL + Midwest)
- **1 source needs a 15-minute platform check** before any decision: NAL

The browser lookup session (already on the schedule) is the critical unlock. Running it before May 1 could add 3 sources to the crawl config with zero engineering time — equivalent to 5,000–13,000 additional games/year for ~15 minutes of Presten config work.

*FORGE — 2026-04-26*
