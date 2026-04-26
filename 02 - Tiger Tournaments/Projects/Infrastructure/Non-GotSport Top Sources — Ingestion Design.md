---
title: Non-GotSport Top Sources — Ingestion Design
tags: [infrastructure, sources, ingestion-design, stage3, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: design-complete — awaiting Presten authorization for any activation
task: task-2026-04-26-non-gotsport-top-source-ingestion-design
---

# Non-GotSport Top Sources — Ingestion Design

> Stage 3 foundation document. Converts the Non-GotSport Source Research Synthesis into a concrete ingestion design for the top 2 non-GotSport platform sources. Goal: SENTINEL has a specific answer to "what comes after GotSport Stage 2?" before DSS.

**Sources covered:** SnapSoccer (SincSports platform), Affinity Soccer (IL + Midwest)  
**Prepared by:** FORGE — 2026-04-26  
**Research basis:** `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md`

---

## Section 1: Top 2 Source Selection

### Source A: SnapSoccer (SincSports Platform)

- **Platform:** SincSports (`soccer.sincsports.com`)
- **Why it ranked #1 for design:** Most feasible non-GotSport source with an existing implementation path. Same underlying platform as the 34,961 SincSports games already in the database. HTML scrape architecture is confirmed working (no auth, no JS rendering, public access). Design is complete (`Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md`). Game volume (1,500–2,500/year ongoing + 10,000–20,000 historical one-time catch-up) is real but sub-GotSport.
- **Access method:** HTML scrape of `soccer.sincsports.com/schedule.aspx?tid={TID}` — one page per event, entire season scrape < 1 minute. TID values discoverable from `snapsoccer.com/events/`. No authentication required.
- **Coverage:** Southeast focus — Alabama, Florida, Georgia, Louisiana. Independent regional coverage not captured via GotSport state associations (SE states with lower GotSport penetration).

### Source B: Affinity Soccer (Illinois + Midwest)

- **Platform:** Affinity Soccer (proprietary — `affinitysoccer.com`)
- **Why it ranked #2 for design:** Highest volume non-GotSport source. Illinois alone represents an estimated 5,000–15,000 games/year — the single largest non-GotSport coverage gap. Wisconsin, Minnesota, Missouri may also use the same platform. IL is confirmed not on GotSport.
- **Access method:** Unknown public path — may require HTML scrape, direct API access, or partnership outreach. No existing integration. This is a Stage 3 engineering sprint, not a quick config add.
- **Coverage:** IL confirmed (largest gap). WI, MN, MO unconfirmed — likely same platform based on Midwest adoption pattern.

---

## Section 2: Ingestion Design — Source A (SnapSoccer / SincSports)

### Data Model Mapping

The SincSports HTML structure is already known from the existing bulk Sinc import (34,961 games in DB). SnapSoccer uses the identical SincSports schedule page format.

| Field (SincSports HTML) | Maps to Evo Draw Field | Transform Required |
|------------------------|----------------------|--------------------|
| Game ID from URL param (`GameId=XXXX`) | `source_game_id` | None — use as-is |
| Date from page header or row | `game_date` | Parse `MM/DD/YYYY` → ISO date |
| Home team name (text in table row) | `home_team_name` | Standard team name normalization (existing pipeline function) |
| Away team name (text in table row) | `away_team_name` | Standard team name normalization |
| Home score (numeric) | `home_score` | Parse as integer; null if `'-'` or blank (game not played) |
| Away score (numeric) | `away_score` | Parse as integer; null if `'-'` or blank |
| Event/tournament name (page title or table header) | `event_name` | Extract from `<h1>` or `<title>` tag |
| Age group (from event name pattern or table header) | `age_group` | Extract via existing age-group parsing rules |
| Gender (from event name pattern) | `gender` | Extract via existing gender parsing rules |
| TID (from URL param `?tid=TID`) | `source_event_id` | Used for dedup key |

**Source and dedup key:** `source = 'snapsoccer'` (distinct from existing `'sincsports'` bulk import to track separately). Dedup key: `(source, source_game_id)` — same pattern as existing SincSports ingestion. Cross-source dedup: no overlap with GotSport (different platform, different ID space). Possible overlap with existing `'sincsports'` bulk import — check `source_game_id` against existing sincsports rows before insert.

### Pipeline Integration

**Architecture fit:** SnapSoccer scraper fits the existing `run-daily.sh` architecture as an additional step. The scraper runs once per event-TID per day; only new/updated game rows are inserted. Implementation as a separate script (`crawl-snapsoccer.js`) that `run-daily.sh` calls in sequence, not in parallel with the GotSport API crawl.

**Development effort estimate:**
- **Test run (10 TIDs):** 5–9 hours — write HTML parser, adapt SincSports table parser, implement TID discovery, test against 10 SnapSoccer events
- **Full production:** 8–12 hours total (adds: full TID list management, error handling, integration into `run-daily.sh`, production test)
- **Reuse credit:** If the existing SincSports bulk importer's HTML parser is reusable for `schedule.aspx` pages, effort drops to ~4–6 hours test / 6–8 hours production

**Rate limiting / scrape risk / ToS:**
- Rate limiting: Not enforced per research — no evidence of throttling on public schedule pages. Conservative rate (1 req/sec) is recommended.
- Scrape risk: Low — SincSports schedule pages are publicly accessible, no auth, designed for parents and coaches to read.
- ToS: Not explicitly confirmed. SincSports is a minor platform; risk is low for public results pages. Not a concern for DSS timeline.

### Blocker Assessment

- **Single largest blocker:** 6–9 hours of engineering time from Presten (this is not a config add; it requires writing a new scraper module)
- **In FORGE's control:** Design is complete, this document provides the implementation spec. FORGE can write the scraper script.
- **Requires Presten authorization:** Yes — any new scraper that hits an external site requires Presten to confirm the rate and scope before production activation. Session scheduling is Presten's call.

---

## Section 3: Ingestion Design — Source B (Affinity Soccer)

### Data Model Mapping

**Caveat:** Affinity Soccer's data structure is not confirmed — FORGE has not seen the platform's HTML or API. The following mapping is provisional based on standard youth soccer results platforms. Actual field names require a 30-minute investigation session on `affinitysoccer.com`.

| Field (Affinity Soccer — provisional) | Maps to Evo Draw Field | Transform Required |
|--------------------------------------|----------------------|--------------------|
| Game/match identifier | `source_game_id` | Platform-specific — unknown format |
| Game date | `game_date` | Likely `MM/DD/YYYY` — needs confirmation |
| Home team name | `home_team_name` | Standard normalization |
| Away team name | `away_team_name` | Standard normalization |
| Home score | `home_score` | Parse as integer |
| Away score | `away_score` | Parse as integer |
| Event name | `event_name` | Extract from page structure |
| Age group | `age_group` | Parse via existing rules |
| Gender | `gender` | Parse via existing rules |
| State/region | `state` | Affinity Soccer data is state-specific; add state tag |

**Source key:** `source = 'affinity'`. Cross-source dedup: Affinity Soccer data is expected to have zero overlap with GotSport (IL/WI/MN are confirmed non-GotSport states). No existing Sinc/TGS overlap.

**Team identity across sources:** Affinity Soccer teams will not have GotSport team_ids. A new team_id resolution step is required for Affinity teams — either create new team records or match by name/state/age-group against existing records. This is the highest-complexity part of the integration and the primary reason this is a weeks-long sprint, not a session.

### Pipeline Integration

**Architecture fit:** Affinity Soccer does NOT fit the existing `run-daily.sh` architecture cleanly. It requires:
1. A new scraper (`crawl-affinity.js`) built from scratch — no existing parser to reuse
2. A new team identity resolution module for Affinity-source teams (no org_id → team_id mapping exists)
3. Potentially a separate ingestion pipeline path if the Affinity data model diverges from GotSport enough to require schema changes

**Development effort estimate:**
- Platform research + access path confirmation: 2–4 hours (Presten browser session)
- Scraper build (assuming public HTML access): 15–25 hours
- Team identity resolution module: 8–15 hours (the hardest part)
- Integration + testing: 5–10 hours
- **Total: 30–50 hours across multiple sessions**

**Rate limiting / scrape risk / ToS:**
- Rate limiting: Unknown — Affinity Soccer may not expect external scraping at all.
- Scrape risk: Medium — Affinity Soccer is a smaller platform; aggressive scraping could trigger blocks.
- ToS: Unknown. FORGE recommends: before any engineering work, confirm whether Affinity Soccer offers a public API or data export. If not, evaluate partnership outreach (IL Youth Soccer / IYSA) as an alternative access path.

### Blocker Assessment

- **Single largest blocker:** Access path is unconfirmed. FORGE cannot spec a scraper without knowing whether Affinity Soccer (a) has public schedule pages, (b) returns structured HTML, or (c) requires authentication.
- **In FORGE's control:** Design research only. Actual implementation is blocked until access path is confirmed.
- **Requires Presten authorization:** Yes — accessing Affinity Soccer requires confirmation that the platform is publicly scrapable. If it requires a data-sharing partnership, that is a Presten/business decision, not a FORGE engineering decision.

---

## Section 4: Go/No-Go Recommendation

| Source | Recommendation | Primary Reason | Earliest Activation Window |
|--------|---------------|---------------|--------------------------|
| Source A — SnapSoccer | **HOLD until post-May 1** | Design is complete. DSS timeline doesn't justify the 6–9 hour build vs. GotSport org-ID expansion (config-only, higher volume). After May 1 GotSport pipeline is stable, SnapSoccer is the first non-GotSport build. | May 17+ (post-DSS). Post-DSS trigger checklist in design doc. |
| Source B — Affinity Soccer | **DEFER to post-DSS sprint** | Access path unconfirmed. 30–50 hour build. Pre-DSS capacity is already committed to GotSport expansion, daily pipeline, and source submission MVP. High post-DSS priority. | June 2026 earliest. Requires dedicated research sprint first. |

**Summary:** Neither non-GotSport source should be activated before DSS (May 16). Stage 3 planning is appropriate to begin in May — this document is that planning. Presten's energy before DSS is best spent on GotSport org-ID expansion (5-minute config adds, 5,000–13,000 additional games) rather than new-platform engineering.

---

## Section 5: Authorization Required from Presten

**Source A (SnapSoccer):**
> "SnapSoccer activation requires Presten to confirm: (1) rate limit policy for SincSports public schedule pages (recommended: 1 req/sec max), and (2) that there is no contractual restriction on scraping SincSports results data. If both are confirmed, FORGE can build the scraper in one 6–9 hour session in May. Authorization to proceed triggers the post-DSS session scheduling."

**Source B (Affinity Soccer):**
> "Affinity Soccer activation requires Presten to confirm the access path — either public HTML scraping is possible (FORGE tests against `affinitysoccer.com` in a 2-hour research session) OR a data-sharing partnership is initiated with IYSA/Illinois Youth Soccer. FORGE cannot begin engineering until one of these paths is confirmed. Authorization to research (30 min browser session) can happen any time after DSS."

---

## Appendix: Why Not EDP, USL Academy, or Elite 64?

EDP, USL Academy, and Elite 64 are covered in the Non-GotSport Source Research Synthesis but are excluded from this ingestion design document because they are GotSport-platform sources — their "ingestion design" is a 5-minute config add, not an engineering sprint. They are handled via the browser session + GotSport Org-ID Config Edit Reference, not this document.

This document covers only platforms that require new scraper engineering: SincSports (SnapSoccer) and Affinity Soccer.

---

## References

- `Infrastructure/Non-GotSport Source Research — April 2026 Synthesis.md` — research base for this design
- `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md` — full SnapSoccer implementation design
- `Data Sources/Affinity Soccer Source Research.md` — Affinity Soccer platform research
- `Infrastructure/Source Gap Inventory — April 2026.md` — gap priorities this document addresses
- `task-2026-04-24-snapsoccer-ingestion-design.md` — prior SnapSoccer ingestion work

*FORGE — 2026-04-26*
