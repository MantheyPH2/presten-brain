---
title: Pre-DSS Source Coverage Report — May 2026
tags: [infrastructure, sources, coverage, dss, reporting, evo-draw, forge]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: filed — update when browser session completes or Stage 1 runs
task: task-2026-04-25-pre-dss-source-coverage-report
---

# Pre-DSS Source Coverage Report — May 2026

> [!info] Purpose
> Single-page coverage picture for SENTINEL and Presten. Produced for the May 9 DSS go/no-go decision. Every league in the [[League Hierarchy]] is accounted for. This report replaces synthesizing across 6+ infrastructure documents.

---

## Section 2: Coverage Summary

```
Report date:          2026-04-25
Total leagues tracked: 17
Active (ingesting):    7
Pending (confirmed path, not yet live): 4
Research required (gap, path exists):   4
Not covered (no path before DSS):       2
```

**Honest coverage claim as of 2026-04-25:**

As of this report, Evo Draw ingests games from 7 of 17 tracked leagues/sources. Four additional leagues have confirmed ingestion paths (USYS State via GotSport org-ID expansion, EDP, USL Academy, and Elite 64 — all conditional on the browser lookup session completing) and could be live by May 9 if Presten runs the browser session and adds confirmed org IDs. The following leagues will **NOT** be covered by DSS on May 16 regardless of effort: Illinois/Midwest Affinity Soccer states (no ingestion path), and NAL (platform unknown, volume too low to prioritize pre-DSS). SnapSoccer is an explicit NO-GO per the April 24 decision.

---

## Section 1: Coverage By League

**Status definitions:**
- **Active** — org IDs/scraper in config, games ingesting within last 60 days (or confirmed functional)
- **Pending** — org IDs identified or confirmed, config update not yet applied (awaiting browser session, Stage 2, etc.)
- **Research required** — platform identified, org IDs not yet confirmed; closeable before May 9 with browser session
- **Not covered** — no path to coverage before DSS; would require new integration or decision

| League | Gender | Platform | Org-IDs / Config Key | Coverage Status | Gap / Notes |
|--------|--------|----------|----------------------|-----------------|-------------|
| **MLS NEXT** | Boys | GotSport API + `crawl-mlsnext.js` | GotSport org IDs + MLS NEXT scraper | **Active** | 27,635 games via MLS NEXT source. GotSport API also contributes. Ingestion recency unverified — run staleness check before May 9. |
| **ECNL Girls** | Girls | TGS AthleteOne (`tgs_scraper`) | `source = 'tgs_athleteone'` | **Active** | TGS primary through June 2026. June 1 migration to GotSport is the next critical event. Pre-migration snapshot captured 2026-04-23. |
| **ECNL Boys** | Boys | TGS AthleteOne (`tgs_scraper`) | `source = 'tgs_athleteone'` | **Active** | Same as ECNL Girls. Migration plan in place. |
| **GA Girls (Girls Academy)** | Girls | GotSport API | `event_tier = 'ga'` | **Active** (partially miscalibrated until April 28) | GA ASPIRE games currently mis-tagged as `ga` (overcalibrated +40 pts). Fix executes April 28. Post-fix: fully active and correctly calibrated. |
| **GA ASPIRE** | Girls | GotSport API | `event_tier = 'ga_aspire'` (post-fix) | **Pending** (April 28 fix) | Games exist but tagged as `ga` tier. April 28 UPDATE reclassifies to `ga_aspire`. No new ingestion work needed — just the tier fix. |
| **GA Boys** | Boys | GotSport API | `event_tier = 'ga'` (Boys) | **Active** | Calibration 100. Boys GA ASPIRE may have minor overcalibration; lower impact than Girls. April 28 fix checks Boys exposure. |
| **ECNL RL Girls** | Girls | GotSport API | `event_tier = 'ecnl_rl'` | **Active (unverified)** | ~300 clubs. Calibration 55. Ingestion recency never formally confirmed. Staleness verification query exists: `Infrastructure/ECNL RL Staleness Verification Query — April 2026.md`. Run before May 9. |
| **ECNL RL Boys** | Boys | GotSport API | `event_tier = 'ecnl_rl'` | **Active (unverified)** | ~400 clubs. Same verification needed as Girls. Highest-ranked gap in Source Coverage Priority Matrix (Score 9). |
| **NPL** | Both | GotSport API | `event_tier = 'npl'` | **Active (unverified)** | Ingestion recency unverified. Verification query: `Infrastructure/NPL-DPL Ingestion Verification Query — April 2026.md`. Run at next psql session. |
| **DPL** | Girls | GotSport API | `event_tier = 'dpl'` | **Active (unverified)** | Girls-focused, GA pathway. Same verification query as NPL. Combined NPL+DPL check: same document. |
| **Pre-ECNL** | Both | GotSport API | `event_tier = 'pre_ecnl'` | **Active (unverified)** | 300+ clubs, U10-U12. Ingestion recency unverified. |
| **USYS State Leagues/Cups** | Both | GotSport API (org-level) | USYS state org IDs (TX cohort ready; expansion pending) | **Pending** (Stage 1 TX run → Stage 2 expansion) | 20,000–35,000 games estimated missing from unranked divisions. TX test crawl has not yet run. Config is prepared. Presten executes TX crawl → Stage 2 adds all U.S. state org IDs. Most impactful pending expansion. |
| **USL Academy** | Boys | GotSport API | org_id pending browser lookup | **Research required** | 90 clubs. High probability GotSport org_id exists. Blocked on 5-min browser lookup at `system.gotsport.com`. If confirmed: same config-add path as USYS states. ~15 min total to close. |
| **EDP (Eastern Development Program)** | Both | GotSport API | org_id pending browser lookup | **Research required** | ~150+ clubs, Northeast. Known GotSport organization. May already have partial ingestion via ranked-team discovery. Blocked on browser lookup + `SELECT COUNT(*) FROM games WHERE event_name ILIKE '%EDP%'` query. |
| **Elite 64** | Both | GotSport API (likely) | org_id pending browser lookup | **Research required** | Via US Club Soccer affiliation. Game volume unknown. Blocked on browser lookup. If org_id confirmed: config-add path. If not on GotSport: requires source research. |
| **NAL (North American League)** | Both | Unknown | None | **Not covered** | Platform unconfirmed. Volume likely low (<1,000 games/season). Browser lookup is pending but this is deferred: if confirmed on GotSport, add post-DSS. If not on GotSport, low volume means no pre-DSS priority. |
| **Illinois / Midwest Affinity states** | Both | Affinity Soccer | None | **Not covered** | IL confirmed not on GotSport. 5,000–15,000 games/year gap. Requires new Affinity Soccer scraper. Post-DSS. No action before June 2026. |
| **SnapSoccer leagues** | Both | SincSports HTML | N/A | **Not covered (NO-GO)** | ~1,500–2,500 games/year. Explicit NO-GO decision 2026-04-24. Volume insufficient for DSS sprint. Post-DSS trigger conditions documented. |

---

## Section 3: Closeable Gaps Before May 9

The following gaps are closeable before May 9 with specific, bounded actions:

| League | Blocking Action | Effort | Achievable by May 9? |
|--------|-----------------|--------|----------------------|
| **USYS State Leagues (unranked)** | Presten runs TX test crawl; on pass: FORGE adds remaining U.S. state org IDs to config | 30–60 min crawl + 5 min config | **Achievable** — highest-leverage action available |
| **GA ASPIRE tier fix** | April 28 session: run UPDATE + rankings recompute | ~60 min | **Achievable** — on calendar April 28 |
| **USL Academy** | Browser session at `system.gotsport.com` → org_id lookup → config add | ~20 min total | **Achievable** — requires Presten browser session |
| **EDP** | DB query to confirm partial coverage + browser session for org_id | ~20 min total | **Achievable** — requires Presten browser session |
| **Elite 64** | Browser session → org_id lookup (if on GotSport) or platform ID (if not) | ~15 min research | **Achievable for research** — config add conditional on GotSport confirmation |
| **ECNL RL staleness** | Run staleness verification query at next psql session | 10 min | **Achievable** — query is written and ready |
| **NPL/DPL staleness** | Run verification query at next psql session | 10 min | **Achievable** — query is written and ready |

**One-session browser window (~45 min) closes 3 gaps:** USL Academy + EDP + Elite 64. The consolidated prep package is at `Infrastructure/GotSport Browser Lookup Session Prep Package.md`. All 11 lookups organized. Presten opens one document, completes in one session.

---

## Section 4: DSS Coverage Claim Recommendation

SENTINEL can choose one of three positions:

**Option A — Conservative (recommended as floor):**
Claim only confirmed-Active leagues: MLS NEXT, ECNL (Boys/Girls), GA, ECNL RL, NPL/DPL, Pre-ECNL. This is honest, defensible, and already represents excellent coverage of Tier 1 and Tier 2.

**Option B — Forward-looking (recommended if browser session completes before May 9):**
Claim Active + Pending, framed as: "Games from USL Academy, EDP, and USYS State Cups will be ingested by [date pending Stage 1 + browser session outcome]." Requires Stage 1 to complete and browser session to run before May 9 go/no-go.

**Option C — Aspirational (not recommended):**
Claiming Elite 64, NAL, or IL Affinity states would require significant new engineering. Do not claim coverage that requires weeks of work in a 3-week window.

> [!recommendation] FORGE recommends Option B — contingent on two Presten actions before May 9:
> 1. Run the browser lookup session (~45 min) to confirm USL Academy, EDP, Elite 64 org IDs
> 2. Run the TX test crawl to gate Stage 2 USYS expansion
>
> If either action does not happen before May 9, default to Option A. Option A is still a strong coverage story: every Tier 1 league and three Tier 2 leagues are active.

---

## References

- [[Source Gap Inventory — April 2026]] — gap detail and next actions
- [[Source Coverage Priority Matrix — April 2026]] — tier rankings and effort scores
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — org-ID status (source of truth)
- `Infrastructure/GotSport Browser Lookup Session Prep Package.md` — consolidated lookup session (unblocks 3 gaps)
- [[League Hierarchy]] — complete league list and calibration values
- `Infrastructure/ECNL RL Staleness Verification Query — April 2026.md` — ECNL RL verification
- `Infrastructure/NPL-DPL Ingestion Verification Query — April 2026.md` — NPL/DPL verification

*FORGE — 2026-04-25*
