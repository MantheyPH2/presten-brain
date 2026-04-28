---
title: Non-GotSport Source Priority Ranking — Post-SincSports Decision
tags: [data-sources, stage3, ingestion, priority-ranking, evo-draw, forge]
created: 2026-04-27
author: FORGE
status: complete
basis: SincSports GO/NO-GO decision locked 2026-04-27
---

# Non-GotSport Source Priority Ranking — Post-SincSports Decision

> Synthesis document. Draws from existing research — no new investigation. Use this as the single reference for post-DSS non-GotSport source sequencing.

**Prepared by:** FORGE — 2026-04-27  
**Trigger:** SincSports GO/NO-GO locked (NO-GO pre-DSS; GO post-DSS May 17+). Priority ranking updated accordingly.  
**Basis documents:** `Infrastructure/Non-GotSport Top Sources — Ingestion Design.md`, `Infrastructure/Source Gap Inventory — April 2026.md`, `Data Sources/SincSports-GoNoGo-Decision-Package-2026-04-27.md`, `Data Sources/Affinity Soccer Source Research.md`

---

## Priority Ranking Table

| Rank | Source | Platform | Games/Year Est. | Build Hours Est. | ROI (games/hr) | Accessibility Risk | DSS Narrative Value | Status |
|------|--------|----------|----------------|-----------------|---------------|-------------------|--------------------|----|
| 1 | **SnapSoccer (SincSports)** | SincSports HTML | 1,500–2,500/yr ongoing + 10,000–20,000 historical catch-up | 5.5–9.5 hrs test / 8–14 hrs production | 160–450 games/hr (test run) | **Low** — public schedule pages, no confirmed auth (browser check pending) | Moderate — fills SE gap not covered by GotSport | NO-GO pre-DSS; GO May 17+ (pending browser check + pipeline health confirmation) |
| 2 | **Affinity Soccer (IL)** | Affinity / proprietary | 5,000–15,000/yr (IL alone); WI, MN, MO possibly additive | 30–50 hrs | 100–300 games/hr | **Medium** — access path unconfirmed; may require auth or partnership | High — IL is the single largest GotSport coverage gap (50,000+ registered players) | DEFER to June 2026+ — requires dedicated research sprint |
| 3 | **NAL (North American League)** | Unknown | Unknown | Unknown | Unknown | **Unknown** — platform not identified | Low-to-medium — national Tier 2 league | Research-only. No platform confirmed. Assign platform identification task before build. |

**Note on GotSport-platform sources (EDP, USL Academy, Elite 64):** These are not included in this ranking because they are GotSport-platform sources — ingestion is a 5-minute config add, not an engineering sprint. See `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` and the browser session prep package for their current status. Their ROI per Presten-hour is vastly higher than any non-GotSport source; prioritize them before any non-GotSport build.

---

## Scoring Notes

### SnapSoccer (SincSports) — Rank 1

**Why first:** Design is complete (`Data Sources/SnapSoccer Ingestion MVP — Design 2026-04-24.md`). Build can begin with a single SENTINEL authorization after the browser check. Historical catch-up (10,000–20,000 games) meaningfully improves SE coverage. Build is a single tight session (5.5–9.5 hrs), not a multi-week sprint. Accessibility risk is low — the most likely failure mode (Cloudflare on `soccer.sincsports.com`) is resolved by the pending 5-minute browser check.

**Accessibility risk detail:** `soccer.sincsports.com` (schedule subdomain) is distinct from `sincsports.com` (Cloudflare-protected main domain). Confirmed accessible from SnapSoccer event links in prior research; final confirmation requires Presten browser check (`Data Sources/SincSports-Browser-Check-Protocol-2026-04-27.md`).

**DSS relevance:** LOW pre-May 16. Post-DSS: fills Southeast gap not addressable via GotSport org-ID expansion.

---

### Affinity Soccer (Illinois) — Rank 2

**Why second:** Highest absolute game volume of any non-GotSport source — IL alone represents 5,000–15,000 games/year, the largest single coverage gap in the pipeline. However, the access path is unconfirmed, build effort is 30–50 hours, and the team identity resolution problem (no GotSport org_id → team_id mappings for IL teams) adds architecture complexity not present in SnapSoccer.

**Blocking condition:** A 2-hour access path research session is required before engineering can begin. FORGE recommends scheduling this as the first Affinity Soccer task in June 2026, not a full build session.

**ROI per session:** Highest total game value of any source. Lowest ROI per hour due to build complexity. Justified by volume at the right time (post-DSS sprint calendar).

---

### NAL (North American League) — Rank 3

**Why third:** Platform genuinely unknown. No engineering estimate is possible. FORGE has not confirmed whether NAL hosts results publicly on GotSport, SincSports, or an independent platform. Before any ranking change, FORGE needs a 30-minute platform identification task: search GotSport for "North American League," check the NAL official website for results publication format. If NAL is on GotSport → it becomes a config add (not this list). If on a third-party platform → reassess ROI after volume is estimated.

---

## Recommendation

**#1 post-DSS non-GotSport target: SnapSoccer (SincSports).**

SnapSoccer should be the first post-DSS non-GotSport engineering session for three reasons: (1) design is complete and requires no additional research before the build; (2) the accessibility gate is a single 5-minute Presten check away from resolution; (3) the build fits in one session (5.5–9.5 hours), making it schedulable as a standalone Presten sprint in the May 17–31 window. Affinity Soccer is the higher-volume target but requires a research sprint before engineering — schedule Affinity access path research for June 2026 after the SnapSoccer build is deployed and stable.

NAL should receive a 30-minute platform identification task before any build commitment. If NAL is on GotSport, it exits this list entirely and becomes a config add.

---

## References

- `Infrastructure/Non-GotSport Top Sources — Ingestion Design.md` — source A/B design (SnapSoccer, Affinity)
- `Data Sources/SincSports-GoNoGo-Decision-Package-2026-04-27.md` — SincSports build effort and coverage estimates
- `Data Sources/Affinity Soccer Source Research.md` — IL platform confirmation and volume estimates
- `Infrastructure/Source Gap Inventory — April 2026.md` — NAL, EDP, Elite 64, USL Academy gap documentation
- `Data Sources/SincSports-Browser-Check-Protocol-2026-04-27.md` — SnapSoccer accessibility gate

*FORGE — 2026-04-27*
