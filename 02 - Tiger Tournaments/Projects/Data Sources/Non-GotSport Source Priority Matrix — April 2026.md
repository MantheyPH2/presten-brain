---
title: Non-GotSport Source Priority Matrix — April 2026
tags: [data-sources, priority-matrix, post-dss, ingestion, evo-draw, forge]
created: 2026-04-27
author: FORGE
status: complete — living document, update when research updates
---

# Non-GotSport Source Priority Matrix — April 2026

> **Purpose:** Consolidated prioritization of all non-GotSport sources FORGE has identified. Answers the question: "Which non-GotSport source do we build next, and why?" Use this as the single reference for post-DSS source sequencing.

**Note on GotSport-platform sources:** EDP, USL Academy, Elite 64, and NAL (if confirmed on GotSport) are NOT included here. GotSport-platform sources are 5-minute config adds — they should always be prioritized before any non-GotSport engineering sprint. See `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` for their status.

---

## Section 1: Source Inventory

| Source | Platform | Coverage | Research Status |
|--------|----------|----------|----------------|
| **SnapSoccer** | SincSports HTML (`soccer.sincsports.com`) | Gulf Coast / Southeast competitive tournaments; ~10–15 events/year | Research complete. Parser spec filed. Browser accessibility check pending. |
| **Affinity Soccer (IL)** | Affinity Soccer (Active Network / Blue Star Sports) | Illinois, possibly WI/MN/MO; IYSA and affiliated state leagues | Access path research complete (2026-04-27). Auth wall behavior unconfirmed. Browser audit required before build. |
| **NAL (North American League)** | Unconfirmed — GotSport primary hypothesis (medium confidence) | National development league; USYS-affiliated; estimated < 1,000 games/yr | Platform identification research complete (2026-04-27). Browser check required to confirm platform. |
| **USYS State Cup (non-TX states)** | Varies by state (GotSport for most, Affinity for some) | USYS State Cup bracket results in states not covered | Research in vault (`Data Sources/USYS State Cup Source Research.md`). No dedicated access path research. |

---

## Section 2: Priority Matrix

### Scoring Formula

**Priority Score = (Volume × 3) + (Build Efficiency × 2) + (Access Confidence × 2) + (Strategic Value × 1)**

| Dimension | Score 1 | Score 2 | Score 3 |
|-----------|---------|---------|---------|
| Volume (net new games/yr) | Low: < 2,000 | Medium: 2,000–8,000 | High: > 8,000 |
| Build Efficiency (lower effort = higher score) | Low: > 30 hrs | Medium: 10–30 hrs | High: < 10 hrs |
| Access Confidence | Low: unknown or auth-gated | Medium: likely accessible, browser check pending | High: confirmed public access |
| Strategic Value | Low: niche coverage, marginal narrative value | Medium: fills regional gap | High: largest single coverage gap, DSS-relevant |

### Matrix

| Source | Est. Net New Games/Yr | Build Effort (hrs) | Access Confidence | Strategic Value | Volume Score | Efficiency Score | Access Score | Strategy Score | **Priority Score** |
|--------|----------------------|--------------------|-------------------|-----------------|-------------|-----------------|-------------|---------------|-------------------|
| **SnapSoccer** | ~5,000–7,000 (net new; 10,000–20,000 historical catch-up) | 5.5–9.5 (test) / 8–14 (production) | Medium — browser check pending; no confirmed auth wall | Medium — fills SE gap, DSS breadth story | 2 | 3 | 2 | 2 | **(2×3)+(3×2)+(2×2)+(2×1) = 18** |
| **Affinity Soccer (IL)** | 5,000–15,000 (IL alone); WI/MN/MO additive | 20–35 (no auth) / 35–55 (session auth) | Low-Medium — CONDITIONAL verdict; auth wall behavior unconfirmed | High — largest single non-GotSport coverage gap; IL = 50,000+ players | 3 | 1 | 1 | 3 | **(3×3)+(1×2)+(1×2)+(3×1) = 16** |
| **NAL** | < 1,000 estimated (low confidence); could be 0 if GotSport → exits this list | Unknown — trivial if GotSport (0 hrs); 20–50 hrs if non-GotSport | Unknown — platform unconfirmed | Low — small national league; limited narrative uplift | 1 | 2* | 1 | 1 | **(1×3)+(2×2)+(1×2)+(1×1) = 10** |

_*NAL efficiency score uses conditional average: if GotSport (Score 3) vs. non-GotSport (Score 1) → scored at 2 pending confirmation._

---

## Section 3: Sequencing Recommendation

1. **First build (May 17 post-DSS):** **SnapSoccer** — because design is complete, build fits a single session (5.5–9.5 hrs), the only open gate is a 5-minute browser check, and historical catch-up (~10,000–20,000 games) meaningfully improves Southeast regional coverage with no additional engineering.

2. **Second build (June–July sprint):** **Affinity Soccer (IL)** — because it represents the highest absolute game volume of any non-GotSport source (5,000–15,000/yr for IL alone), but requires a dedicated 1-hour access path browser audit before engineering begins. If IYSA pages are publicly accessible: schedule a June 2026 build sprint. If auth-gated: re-evaluate.

3. **Conditional build:** **NAL** — if the browser check confirms GotSport platform, NAL exits this list and becomes a config add (5 minutes, no engineering). If non-GotSport with volume < 1,000 games/yr: indefinitely deprioritized. Run the NAL browser check as part of the existing browser session (already in prep package).

**SnapSoccer first is confirmed.** This recommendation is consistent with the working plan. The matrix scores validate the sequence: SnapSoccer (18) > Affinity Soccer (16) > NAL (10 conditional). Affinity Soccer's high strategic value (score 3) is outweighed by its low access confidence (score 1) and high build effort (score 1), making it the right second target rather than first.

---

## Section 4: Outstanding Access Unknowns

| Source | Browser Check Required | Est. Time | What Confirms ACCESSIBLE |
|--------|----------------------|-----------|-------------------------|
| SnapSoccer | Navigate to `soccer.sincsports.com`, open any SnapSoccer tournament schedule page via `snapsoccer.com/events/` link | ~20 min | `schedule.aspx` loads with game rows visible, no login prompt |
| Affinity Soccer | Navigate to `affinitysoccer.com` → search "Illinois Youth Soccer Association" → open IYSA program page | ~60 min | Result tables load without login prompt for a recent season |
| NAL | Navigate to `system.gotsport.com` → search "North American League" or "NAL Soccer" | ~15 min | Org entry appears with org-ID (GotSport confirmed), or website check if not found |

All three checks can be combined with the existing browser session (already planned for EDP, USL Academy, Elite 64, TYSA, and 7 other entities). Estimated combined additional time: ~1.5 hours.

---

## Section 5: Sources Explicitly Deprioritized

| Source | Reason | Revisit Condition |
|--------|--------|------------------|
| **USYS State Cup (non-TX states)** | Covered piecemeal by existing GotSport org-ID expansion as state associations are confirmed. Not a distinct engineering effort. | Re-evaluate if a state not on GotSport runs its State Cup on a separate platform. |
| **TopDrawerSoccer / recruiting platforms** | Editorial data, not game result data. Not ingestible via scraping at the game level. No score data published in structured format. | No viable ingestion path identified. |
| **Demosphere-hosted leagues** | No Demosphere sources identified in current gap inventory with volume > 2,000 games/yr and confirmed public access. | If NAL is confirmed on Demosphere with volume > 2,000: reassess. |

---

## References

- `Data Sources/Affinity-Soccer-Access-Path-Research-2026-04-27.md` — Affinity access path detail
- `Data Sources/NAL-Platform-Identification-Research-2026-04-27.md` — NAL platform research
- `Data Sources/SnapSoccer-TID-Coverage-Estimate-2026-04-27.md` — SnapSoccer volume and runtime
- `Data Sources/SnapSoccer-Minimum-Viable-Parser-Spec-2026-04-27.md` — SnapSoccer parser design
- `Data Sources/Non-GotSport-Source-Priority-Rerank-2026-04-27.md` — prior rerank (this document supersedes)
- `Infrastructure/Source Gap Inventory — April 2026.md` — full gap inventory context
- `Infrastructure/Competitive Coverage Gap Inventory — April 2026.md` — strategic coverage framing

*FORGE — 2026-04-27*
