---
title: SnapSoccer — Authorization Response Package
tags: [infrastructure, snapsoccer, sincsports, ingestion, stage3, authorization, evo-draw, forge]
created: 2026-04-29
updated: 2026-04-29
author: FORGE
status: ready-to-activate — execute on SENTINEL GO (by May 10)
linked_task: task-2026-04-29-snapsoccer-authorization-response-package
authorization_deadline: 2026-05-10
build_target: 2026-05-17
---

# SnapSoccer — Authorization Response Package

> **Activate the moment SENTINEL issues GO on SnapSoccer.**
> Authorization gate: May 10, 2026. Build target: May 17, 2026. 7-day window.
> This package is pre-built so FORGE starts immediately — no planning lag after authorization.

---

## 1. Authorization Trigger

When SENTINEL authorizes SnapSoccer Source #3 (by May 10), **FORGE takes these actions within 1 hour:**

1. Open this document — update `status` to `active`
2. Create `Infrastructure/SnapSoccer Build Log.md` — running log of daily build progress (entries from May 10 until first production run confirmed)
3. Confirm SnapSoccer browser check is in hand: `soccer.sincsports.com/schedule.aspx` must have returned HTTP 200 with visible schedule table (per `Infrastructure/SnapSoccer Browser Check — Findings Report.md`). If not yet confirmed: FORGE requests Presten run the 5-minute check before May 11 build start.
4. Confirm `cheerio` npm package is present in project: `ls node_modules/cheerio` or `cat package.json | grep cheerio`. If absent: `npm install cheerio`.
5. No additional SENTINEL or Presten notification needed beyond the acknowledgment in FORGE's next briefing.

**Authorization acknowledgment in FORGE's next briefing after May 10:**

> **SnapSoccer Source #3 — Build Initiated [Date]:** SENTINEL authorization received. Authorization Response Package activated. Build schedule: May 10–17. First production run target: May 17. Build Log created at `Infrastructure/SnapSoccer Build Log.md`.

---

## 2. Build Schedule (May 10–17 Window)

**Total estimated build time: 5.5–9.5 hours (REBUILD verdict — no parser reuse exists)**

| Day | Phase | Deliverable | Est. Hours | Dependencies |
|-----|-------|------------|-----------|--------------|
| **May 10 (Mon)** | Authorization receipt | Package activated; Build Log created; browser check + cheerio confirmed | 0.5 hr | SENTINEL GO |
| **May 11 (Tue)** | Test extraction session | `test-snapsoccer-extraction.js` — minimal fetch + parse for one known TID; stdout validation against Section 3 pass/fail criteria in `Infrastructure/SnapSoccer — Test Extraction Plan.md` | 1.5–2.5 hrs | One completed-event TID from snapsoccer.com/events/ (Presten provides) |
| **May 12 (Wed)** | TID discovery module | `discoverTournamentIds()` — crawls snapsoccer.com/events/, extracts `tid=` params, filters Playdate TIDs (SNAPLEAG, SNAPLEAF, SSSPF) | 1–2 hrs | Test extraction PASS |
| **May 13 (Thu)** | HTML parser core | `parseSchedulePage(html, tid)` — full ASP.NET table parse, all 10 fields, division headers for age/gender extraction using existing Evo Draw parsing rules | 2–4 hrs | TID discovery complete |
| **May 14 (Fri)** | DB insertion + pipeline integration | `insertGame()` ON CONFLICT upsert into `games` table; add SnapSoccer step to `run-daily.sh` | 1 hr | HTML parser complete |
| **May 15 (Sat)** | Multi-event test run | Test against 3–5 known completed events; verify score parsing, age/gender extraction, dedup against existing 34,961 Sinc records in DB | 1–2 hrs | Full parser + DB insertion complete |
| **May 16 (Sun)** | Buffer / fix day | Address any May 15 failures; final syntax check. Note: DSS is May 16 — no coding during DSS event window. Buffer applies to pre/post-DSS hours. | Buffer | May 15 test results |
| **May 17 (Mon)** | First production run | SnapSoccer Source #3 live: trigger `crawl-snapsoccer.js` via daily pipeline; confirm game count ingested; file first-run results in Build Log | 0.5 hr | All May 11–15 phases PASS |

**If build cannot complete by May 17:** Critical path is the HTML parser (May 13). If May 13 runs past 4 hours, May 15 multi-event testing may need to shift into May 16 buffer. If that extends: earliest realistic first production run is **May 19** (Monday following week). FORGE files a delay notice in the May 16 briefing if May 15 does not produce PASS results.

**Full implementation spec:** `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md`
**Scraper skeleton foundation:** `Data Sources/SnapSoccer Source Research.md`

---

## 3. Engineering Dependencies

| Dependency | Status | Notes |
|------------|--------|-------|
| `soccer.sincsports.com` accessibility | Pre-confirm before May 11 | Must confirm HTTP 200 + schedule table visible, no Cloudflare block or CAPTCHA. See `Infrastructure/SnapSoccer Browser Check — Findings Report.md`. If blocked: entire build stops; FORGE escalates to SENTINEL immediately. |
| Parser reuse from SincSports | **NO REUSE — REBUILD** | No live SincSports HTML parser exists. Original 34,961 SincSports games were bulk-imported (CSV/data export), not scraped live. Full build from scratch. See `Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md`. |
| GotSport HTML scraper (optional credit) | Conditional — confirm before May 13 | If `crawl-gotsport-html.js` has a modular parse function, Presten can extract it to reduce HTML parser phase from 2–4 hrs to 1–2 hrs. Not counted in base estimate. Presten confirms before May 13 session. |
| Schema / database changes | **NONE REQUIRED** | All 10 SnapSoccer fields map cleanly to existing `games` table. `source = 'snapsoccer'` is a new source value requiring no DDL. No ALTER TABLE. |
| Node.js dependencies | `cheerio` may need install | `axios` or `node-fetch` already in project. Confirm `cheerio` present on May 10 (Step 4 of authorization trigger). |
| ELO coordination | None required at build time | SnapSoccer games go into `games` table with `source = 'snapsoccer'` and feed ELO's calibration cycle normally. FORGE notifies ELO after first-run confirmation (Section 4 milestone). |

---

## 4. SENTINEL Milestone Reporting

FORGE's briefings include a **"SnapSoccer Build Status"** section from May 10 until first production run is confirmed. SENTINEL should not need to ask for status — FORGE reports at each phase completion.

| Milestone | FORGE Briefing Signal | Expected Date |
|-----------|----------------------|---------------|
| Build start confirmed | "SnapSoccer Source #3 — Build Initiated. Package activated. Test extraction session begins May 11." | May 10 briefing |
| Test extraction PASS | "SnapSoccer test extraction PASS: [N] games extracted from TID [TID]. All pass/fail criteria met. Proceeding to full build." | May 11 briefing |
| Parser complete | "SnapSoccer HTML parser built and unit-tested: all 10 fields extracting correctly across [N] test divisions." | May 13–14 briefing |
| Multi-event test PASS | "SnapSoccer 3-event test PASS: [N] games extracted, 0 duplicates vs. existing Sinc records, score parsing clean." | May 15 briefing |
| First production run | "SnapSoccer Source #3 LIVE: [N] games ingested from first production run. Source active in daily pipeline." | May 17 briefing |

If any milestone is not met on the expected date, FORGE files a **delay notice that same day**: what blocked it, revised target date, and whether May 17 is still achievable.

---

## 5. Risk Log

| Risk | Probability | Timeline Impact | Mitigation |
|------|------------|-----------------|------------|
| **Cloudflare or anti-bot block on soccer.sincsports.com** | Low (browser check expected to confirm HTTP 200) | **HIGH** — blocks entire build if confirmed blocked | 5-min browser check before May 11 build start. If blocked: FORGE escalates to SENTINEL immediately; May 17 target suspended pending resolution or source re-evaluation. |
| **HTML table structure differs from documented format** | Medium — Reuse Assessment could not verify live `schedule.aspx` HTML directly | **MEDIUM** — adds 1–2 hours to May 13 parser phase | Test extraction on May 11 surfaces structural surprises before full parser build. If structure differs: document new format, adjust approach, absorb extra time in May 13 estimate. Low probability of total failure; medium probability of 1–2 hour delay. |
| **DSS event on May 16 reduces buffer capacity** | Certain (DSS is May 16) | **LOW** if May 15 multi-event test passes cleanly | Buffer day exists. If May 15 produces failures requiring code changes, first production run shifts to May 19. FORGE files delay notice in May 16 briefing. |
| **Build time exceeds high estimate (9.5 hrs)** | Low | **MEDIUM** — shifts first production run to May 19 | GotSport HTML scraper architecture credit (Section 3) can reduce high estimate by 1–2 hrs if applicable. TID discovery automation can be deferred post-v1 if it blocks schedule — use manual TID list as fallback. |
| **TID discovery returns 0 tournament IDs** | Very Low | **LOW** — manual fallback available | If snapsoccer.com/events/ page structure changes, automated discovery fails. Fallback: Presten provides 5–10 known completed event TIDs manually. Build proceeds without automated discovery; add discovery module as post-v1 enhancement. |

---

## References

- `Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md` — REBUILD verdict and build scope
- `Infrastructure/SnapSoccer — Test Extraction Plan.md` — test extraction steps and pass/fail criteria
- `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md` — full implementation spec (primary build guide)
- `Data Sources/SnapSoccer Source Research.md` — scraper architecture and TID URL structure
- `Infrastructure/Non-GotSport Source Priority Recommendation.md` — SnapSoccer as Source #3, authorization request and timing
- `Infrastructure/SnapSoccer Browser Check — Findings Report.md` — accessibility pre-check (fill before May 11)

*FORGE — 2026-04-29*
