---
title: Source Gap Inventory — April 2026
tags: [infrastructure, sources, gap-analysis, coverage, evo-draw, forge]
created: 2026-04-24
updated: 2026-04-24 (rows updated at 15:46 — USL Academy, EDP, Elite 64, NAL provisional research added; org ID confirmation pending Presten browser lookup)
author: FORGE
status: complete
task: task-2026-04-24-source-gap-inventory
---

# Source Gap Inventory — April 2026

> [!info] Purpose
> Maps every league in [[League Hierarchy]] to its data source(s), ingestion status, and known gaps. SENTINEL uses this to assign source research tasks. Updated: 2026-04-24.

---

## Section 1: Inventory Table

**Status Definitions:**
- **Active** — source is in the crawl config, ingested within last 60 days
- **Stale** — source is in config but no confirmed ingestion in 60+ days (may need refresh or new events)
- **No source** — league has no corresponding GotSport org_id or scraper configured
- **Partial** — some games ingested (ranked-team discovery only); unranked divisions missing

| League | Gender | Source Type | Source ID / Config Key | Last Known Ingestion | Status | Notes |
|--------|--------|-------------|----------------------|---------------------|--------|-------|
| **MLS NEXT** | Boys | GotSport API + `crawl-mlsnext.js` | GotSport org IDs + MLS NEXT source (priority 4) | Unknown — query needed | Active | Both GotSport and dedicated MLS NEXT scraper. 27,635 games in DB via MLS NEXT source; additional via GotSport API. |
| **ECNL Girls** | Girls | TGS AthleteOne (priority 3) | `tgs_scraper` | Recent (within 60 days) | Active | TGS is primary through June 2026. **Migration to GotSport on June 1.** Post-migration: source shifts to GotSport API. ECNL dedup P0 fix required before migration. |
| **ECNL Boys** | Boys | TGS AthleteOne (priority 3) | `tgs_scraper` | Recent (within 60 days) | Active | Same migration risk as ECNL Girls. |
| **GA (Girls Academy)** | Girls | GotSport API (priority 1) | event_tier = `ga` | Recent (within 60 days) | Active | Currently calibrated at 140. GA ASPIRE events are misclassified as `ga` until April 28 fix. |
| **GA ASPIRE** | Girls | GotSport API (priority 1) | event_tier = `ga_aspire` (post-fix) | N/A — tier not yet applied | Partial | Events with "ASPIRE" in name currently tagged as `ga` (overcalibrated at 140 vs correct 100). Fix pending April 28 execution. Post-fix: active. |
| **GA Boys** | Boys | GotSport API (priority 1) | event_tier = `ga` (Boys) | Recent (within 60 days) | Active | Calibration 100. Boys GA ASPIRE events may be incorrectly tagged — see [[League Hierarchy]] Boys GA ASPIRE flag. |
| **ECNL RL Girls** | Girls | GotSport API (priority 1) | event_tier = `ecnl_rl` | Unknown — query needed | Active | ~300 girls clubs. Calibration 55. `resolveTeamTier()` enforces ECRL teams capped at ecrl even in ECNL events. |
| **ECNL RL Boys** | Boys | GotSport API (priority 1) | event_tier = `ecnl_rl` | Unknown — query needed | Active | ~400 boys clubs. Same override rules as Girls. |
| **NPL** | Both | GotSport API (priority 1) | event_tier = `npl` | Unknown — query needed | Active | Merit-based promotion system. Integrating with USYS National League for 2026-27. |
| **DPL** | Girls | GotSport API (priority 1) | event_tier = `dpl` | Unknown — query needed | Active | Girls-focused, GA pathway. Calibration 55. |
| **USL Academy** | Boys | GotSport API (priority 1) | org_id pending lookup | Unknown — query needed | Stale/Partial | 90 clubs with professional pathway (U13–U19). USL Soccer is a major GotSport-platform organization; **high probability GotSport org_id exists.** Lookup: `system.gotsport.com` → search "USL Academy" or "United Soccer League." If org_id confirmed: add to crawl config (same process as USYS state org expansion). Flag for Presten to add. |
| **Elite 64** | Both | GotSport (likely — via US Club Soccer) | org_id pending lookup | N/A | No source | Elite 64 is affiliated with US Club Soccer, which is a primary GotSport organization. **High probability Elite 64 events appear under a US Club Soccer or dedicated org_id on GotSport.** Lookup: `system.gotsport.com` → search "Elite 64" or "US Club Soccer." Game volume unknown — likely moderate (national championship/showcase format, not full-season). |
| **EDP (Eastern Development Program)** | Both | GotSport API (priority 1) | org_id pending lookup | Unknown — query needed | Partial | Major northeastern regional league (NJ, NY, PA, CT, MA, MD, RI, DE, VA, VT). ~150+ clubs, U8–U19. **EDP Soccer is a known GotSport organization.** Likely ingested via ranked-team discovery already; org-level explicit config not confirmed. Lookup: `system.gotsport.com` → search "Eastern Development Program" or "EDP Soccer." DB query to confirm existing ingestion: `SELECT COUNT(*) FROM games WHERE event_name ILIKE '%EDP%'` |
| **NAL (North American League)** | Both | Unknown — research needed | None confirmed | N/A | No source | Platform genuinely unknown as of 2026-04-24. NAL appears at Tier 2 alongside USL Academy and EDP in the League Hierarchy. Could be on GotSport or an independent platform. **Required next step:** identify NAL website → check result publication platform. Search GotSport for "North American League" or "NAL Soccer." If results found on an independent platform, flag for scraper feasibility assessment. |
| **Pre-ECNL** | Both | GotSport API (priority 1) | event_tier = `pre_ecnl` | Unknown — query needed | Active | U10-U12 developmental. 300+ clubs. Calibration 25. `resolveTeamTier()` enforces correct tier. |
| **State Leagues — GotSport states** | Both | GotSport API (priority 1) | USYS state org IDs (expansion in progress) | Unknown — query needed | Partial | Ranked-division teams discovered via rankings endpoint. Unranked divisions (state cup/state league) **invisible** without org-level discovery. Expansion task adds 9 state org IDs (TX, CA, FL, NY, NJ, PA, WA, OH, CO). |
| **State Leagues — Illinois (Affinity Soccer)** | Both | None | None | N/A | No source | IL uses Affinity Soccer, not GotSport. Deprioritized: post-DSS. Est. 5,000–15,000 games/year gap. See [[Affinity Soccer Source Research]]. |
| **State Leagues — Other Affinity states** | Both | None | None | N/A | No source | WI, MO, MN likely also Affinity Soccer. Research incomplete. |
| **USYS State Cup (unranked divisions)** | Both | GotSport API (org-level) | USYS state org IDs (expansion pending) | None yet | Partial | 20,000–35,000 games estimated missing. Org-ID expansion task addresses this. See [[GotSport USYS Org-ID Master List]]. |
| **SnapSoccer** | Both | SincSports HTML (proposed) | N/A — MVP design complete | N/A | No source | **NO-GO for DSS sprint.** ~1,500–2,500 games/year. Coverage insufficient to justify sprint time vs Org-ID expansion (20,000+ games). Post-DSS trigger: SnapSoccer access confirmed stable + team matching <5% ambiguity. |
| **ODP (Olympic Development Program)** | Both | N/A | N/A | N/A | Not applicable | Talent identification pipeline. No public game results. Rankings not applicable. |

---

## Section 2: Coverage Gap Summary

### Priority 1 — No source (games completely absent from rankings)

1. **USYS State Cup/League unranked divisions** — 20,000–35,000 games estimated missing. GotSport org-ID expansion task addresses this (TX, CA, FL, NY, NJ, PA, WA, OH, CO). Partial solution in progress.
2. **Illinois State Leagues (Affinity Soccer)** — 5,000–15,000 games/year. No ingestion path. Post-DSS.
3. **Elite 64** — game volume unknown. No org_id confirmed. Likely on GotSport via US Club Soccer. Requires browser lookup to confirm GotSport org_id. If confirmed: config update path (same as USYS state expansion). If not on GotSport: research alternative source. **Go/No-Go: conditional — if GotSport org_id confirmed, GO for config addition. If not, assess game volume before committing to new scraper.**
4. **NAL (North American League)** — game volume unknown. Platform unknown. Requires website + GotSport search. **Go/No-Go: defer until platform confirmed. If <5,000 games/year, deprioritize post-DSS. If >5,000 games/year and on GotSport, recommend config addition.**

### Priority 2 — Stale or partial source

1. **USL Academy** — GotSport org_id not confirmed in config. 90 clubs. USL Soccer is a GotSport organization — high probability org_id exists. Lookup: `system.gotsport.com` → "USL Academy." If confirmed: add to config (same path as USYS state expansion). Requires Presten browser lookup; flagged for next session.
2. **EDP** — No explicit org_id in config. EDP Soccer is a known GotSport organization (~150+ clubs, Northeast). May already have game ingestion via ranked-team discovery. Confirmation requires DB query (`SELECT COUNT(*) FROM games WHERE event_name ILIKE '%EDP%'`) + browser lookup for org_id. Flagged for Presten session.
3. **ECNL RL Boys/Girls** — Active source but last ingestion date unverified. Query needed to confirm recent games.
4. **NPL/DPL** — Active source but last ingestion date unverified.
5. **GA ASPIRE** — Misclassified as `ga` tier until April 28 fix executes. Post-fix: active.

### Priority 3 — Source exists, known limitations documented

1. **SnapSoccer** — NO-GO for DSS sprint. Design complete (`Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md`). Post-DSS trigger conditions documented.
2. **Affinity Soccer (IL + Midwest)** — Deprioritized. Research complete (`Data Sources/Affinity Soccer Source Research.md`). Post-DSS per recommendation.
3. **MLS NEXT Source scraper** — Active but may duplicate GotSport API data. 27,635 games ingested; dedup pipeline handles overlap.

---

## Section 3: Next Actions

### Priority 1 Gaps

- **USYS State Cup/League (unranked)** — Run TX org-ID test crawl (Presten execution, per `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` Section 2). Once test passes, add all P1 state org IDs from `Data Sources/GotSport USYS Org-ID Master List.md`.
- **Illinois (Affinity Soccer)** — Deprioritize until post-DSS. Revisit June 2026 after migration stabilizes.
- **Elite 64** — Browser lookup at `system.gotsport.com` → search "Elite 64" and "US Club Soccer." **If org_id found:** note the org_id, add one-line annotation "Org-ID confirmed — pending Presten config addition," promote status to Active. **If not found:** check Elite 64 official website for result publication platform; document platform name for scraper feasibility assessment. Game volume estimate needed before committing build time.
- **NAL** — Two-step: (1) Search GotSport for "North American League" or "NAL Soccer." (2) If not found, identify NAL's official website and check result format. **If on GotSport:** same config-add path as USYS states (low effort). **If on third-party platform:** estimate game volume first — deprioritize if <5,000 games/year. **If no public results available:** flag as inaccessible, remove from Priority 1 list.

### Priority 2 Gaps

- **USL Academy** — FORGE browser lookup: navigate to GotSport org page for USL Academy to retrieve org_id. Add to `GotSport USYS Org-ID Master List.md`.
- **EDP** — FORGE: query DB for `event_name ILIKE '%EDP%' OR event_name ILIKE '%Eastern Development%'` to confirm coverage and volume.
- **ECNL RL / NPL / DPL last ingestion** — Run verification query:

```sql
SELECT e.event_tier, MAX(g.created_at) AS last_game_ingested, COUNT(*) AS game_count
FROM games g
JOIN event_tiers e ON e.event_name = g.event_name
WHERE e.event_tier IN ('ecnl_rl', 'npl', 'dpl', 'pre_ecnl')
GROUP BY e.event_tier
ORDER BY e.event_tier;
```

---

## Verification Query Template

For any league where Last Known Ingestion is "Unknown — query needed":

```sql
SELECT event_tier, MAX(created_at) AS last_game_ingested, COUNT(*) AS total_games
FROM games g
JOIN event_tiers et ON et.event_name = g.event_name
WHERE event_tier = '[tier_identifier]'
GROUP BY event_tier;
```

---

## Related

- [[League Hierarchy]] — master league list and calibration values
- [[Data Sources Overview]] — volume breakdown by source
- [[GotSport USYS Org-ID Master List]] — state association org IDs for expansion
- `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md` — SnapSoccer NO-GO decision
- `Data Sources/Affinity Soccer Source Research.md` — IL/Midwest gap research
- `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — 540-team backfill addressing ranked-discovery staleness

*FORGE — 2026-04-24*
