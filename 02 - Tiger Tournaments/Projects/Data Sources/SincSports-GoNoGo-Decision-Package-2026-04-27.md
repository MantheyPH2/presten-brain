---
title: SincSports (SnapSoccer) — Go/No-Go Decision Package
tags: [data-sources, sincsports, snapsoccer, go-no-go, stage3, evo-draw, forge]
created: 2026-04-27
author: FORGE
status: complete — RECOMMENDATION: NO-GO pre-DSS; GO post-DSS (May 17+)
task: task-2026-04-27-sincsports-go-no-go-decision-package
---

# SincSports (SnapSoccer) — Go/No-Go Decision Package

> Decision document. SENTINEL can issue a GO/NO-GO call based solely on this document. Design is already complete — see `Infrastructure/Non-GotSport Top Sources — Ingestion Design.md`.

**Prepared by:** FORGE — 2026-04-27  
**Recommendation:** **NO-GO for pre-DSS (before May 16). GO for post-DSS sprint (May 17+).**

---

## 1. Extraction Viability Result

**Test extraction: NOT YET EXECUTED.**

The test extraction plan was filed 2026-04-27 (`Infrastructure/SnapSoccer — Test Extraction Plan.md`). It requires Presten to confirm `soccer.sincsports.com` public schedule pages are accessible without Cloudflare blocking — a 5-minute browser check. That check has not yet been done.

**However, extraction viability can be assessed from prior research:**

The SincSports `schedule.aspx` subdomain (`soccer.sincsports.com`) is distinct from the main `sincsports.com` domain. The main domain is Cloudflare-protected. The `soccer.sincsports.com` subdomain serves public schedule pages linked from `snapsoccer.com/events/` — there is no documented evidence of Cloudflare protection on the schedule subdomain.

**Assessment:** Extraction is likely viable but requires a 5-minute browser confirmation before build commitment. This is a single pending check, not a fundamental blocker.

**If the test is blocked:** FORGE cannot confirm viability autonomously. Presten must run the 5-minute accessibility check. If Presten's check returns "site accessible, no auth required" → extraction is viable. If blocked by Cloudflare or auth → NO-GO indefinitely.

---

## 2. Coverage Estimate

| Metric | Value | Source |
|--------|-------|--------|
| Estimated ongoing game volume | 1,500–2,500 games/year | `Infrastructure/Non-GotSport Top Sources — Ingestion Design.md` Section 1 |
| Historical one-time catch-up | 10,000–20,000 games | Same source |
| Primary geographic coverage | Alabama, Florida, Georgia, Louisiana | Southeast US — independent of GotSport state associations |
| Overlap with GotSport | No overlap — different platform, different ID space | `Non-GotSport Top Sources` Section 2 |
| Overlap with existing SincSports bulk import | Cross-check required — `source_game_id` dedup against existing `sincsports` source rows | `Non-GotSport Top Sources` Section 2 |

**Coverage gap these games fill:** Southeast states with lower GotSport penetration. SnapSoccer covers independent regional tournaments that do not appear in GotSport API returns, even after the full org-ID expansion. This is genuinely additive coverage, not duplication of GotSport data.

**Comparison to GotSport expansion:** The GotSport org-ID expansion (CA, FL, NY, NJ, PA, WA, OH, CO state orgs) is estimated to add 5,000–13,000 games per config update — each update taking under 5 minutes. SnapSoccer's 1,500–2,500 games/year requires a 5.5–9.5 hour build session.

---

## 3. Build Effort Estimate

**Reuse verdict: REBUILD** — No existing SincSports HTML parser exists. Original SincSports games were bulk-imported via file export, not HTML scraping.

Source: `Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md` — Section 3.

| Phase | Effort (hours) |
|-------|---------------|
| TID discovery module | 1–2 |
| Schedule page fetcher | 0.5 |
| HTML parser (core build work) | 2–4 |
| DB insertion (upsert, dedup) | 0.5 |
| Integration into `run-daily.sh` | 0.5 |
| Test against 3–5 known events | 1–2 |
| **Test run total** | **5.5–9.5 hours** |
| Full production (error handling, full TID list, production test) | +3–5 hours |
| **Production total** | **8–14 hours** |

**Conditional reduction:** If the GotSport HTML scraper (`crawl-gotsport-html.js`) has a modular parser, Presten may be able to adapt its HTML parsing logic for SnapSoccer's `schedule.aspx` structure. This could reduce the HTML parser phase to 1–2 hours. Not confirmed — Presten would need to assess the GotSport HTML scraper's modularity.

**Deduplication complexity:** Low. SnapSoccer uses the SincSports platform; dedup key is `(source='snapsoccer', source_game_id)`. Cross-source dedup against existing `sincsports` bulk import rows requires a one-time comparison query but no new dedup architecture.

---

## 4. DSS Relevance (May 16 Demo)

**Assessment: LOW.**

| Factor | Impact |
|--------|--------|
| Game count added before May 16 | 0 — build cannot complete in time (5.5+ hours of new engineering after all pre-DSS work is complete) |
| States covered | AL, FL (partial), GA (partial), LA — Southeast focus |
| Coverage narrative improvement | Marginal — SnapSoccer adds independent SE tournaments. GotSport FL and GA state org-IDs (in Stage 2 scope) cover the same geographic area with higher volume. |
| Alternative before DSS | GotSport FL, NY, NJ, PA, WA org-ID config additions — each 5 minutes, estimated 2,000–5,000 games each |

**Specific DSS delta if activated:** Approximately +1,500–2,500 additional games in the live database. For context, FYSA (Florida Youth Soccer) GotSport config add alone is estimated at ~5,000–10,000 games. **GotSport expansion is strictly higher ROI per hour of Presten's time before DSS.**

---

## 5. Recommendation

**GO / NO-GO:** **NO-GO pre-DSS. GO post-DSS.**

### Pre-DSS NO-GO (Before May 16)

Rationale:
1. Build effort (5.5–9.5 hours minimum) cannot be fit into the pre-DSS calendar alongside Stage 1 TX crawl, May 1 daily pipeline deployment, Stage 2 org-ID browser session, and April 28/29 DB sessions.
2. Coverage delta (1,500–2,500 games) does not materially improve the DSS coverage claim. GotSport org-ID expansion adds more games per hour of effort.
3. Accessibility not yet confirmed (5-minute Presten check is a prerequisite for the build).
4. This recommendation is consistent with the existing design document (`Non-GotSport Top Sources` Section 4, filed 2026-04-26).

### Post-DSS GO (May 17+)

**Proposed build timeline:**
- **May 17–May 20:** Presten accessibility check (5 min) + test extraction execution (1 hour) against a single SnapSoccer event
- **May 21–May 28:** Full scraper build session (6–10 hours over 1–2 Presten sessions)
- **May 28+:** Deploy into daily pipeline, activate, monitor first 3–5 day run

**GO conditions (required before scheduling the build session):**
1. `soccer.sincsports.com` confirmed accessible without auth or Cloudflare blocking (Presten 5-minute check)
2. Stage 1 TX crawl and May 1 pipeline confirmed healthy (daily pipeline is stable before adding new sources)
3. SENTINEL issues explicit GO authorization for SnapSoccer build session post-DSS

---

## 6. Presten Action Required

| Action | Required for GO? | Time estimate |
|--------|-----------------|--------------|
| Browser check: `soccer.sincsports.com` schedule page accessible? | YES — prerequisite to build | 5 minutes |
| Review and accept this GO/NO-GO recommendation | YES — SENTINEL authorization required | < 5 minutes |
| Schedule build session (post-DSS) | YES — after both above complete | N/A |

**If Presten has already checked and `soccer.sincsports.com` is accessible:** The GO conditions for post-DSS are met (pending pipeline health confirmation after May 1). SENTINEL can authorize the post-DSS sprint now.

**If Presten has NOT checked accessibility:** This remains the single blocking action. FORGE cannot initiate the build without this confirmation.

---

## References

- `Infrastructure/Non-GotSport Top Sources — Ingestion Design.md` — Section 4 (original GO/NO-GO recommendation, April 26)
- `Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md` — REBUILD verdict (reuse not possible)
- `Infrastructure/SnapSoccer — Test Extraction Plan.md` — test execution plan for post-DSS sprint readiness
- `Data Sources/SnapSoccer Source Research.md` — scraper skeleton and TID structure
- `Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md` — full implementation design (ready for build)
- `Infrastructure/Source Gap Inventory — April 2026.md` — SnapSoccer listed as Priority 3 (NO-GO for DSS sprint)

*FORGE — 2026-04-27*
