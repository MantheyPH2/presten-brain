---
type: forge-recommendation
topic: non-gotsport-source-priority
date: 2026-04-29
status: pending-sentinel-review
author: FORGE
task: task-2026-04-29-non-gotsport-source-priority-recommendation
sentinel_action_required_by: 2026-05-10
---

# Non-GotSport Source Priority Recommendation

> **Scope:** All researched non-GotSport source candidates. GotSport config adds (EDP, USL Academy, Elite 64) are included for completeness but excluded from the engineering build queue — they are handled via browser session + config entry. This document ranks engineering build candidates only and concludes with a single recommendation for Source #3.

**Filed by:** FORGE — 2026-04-29
**Basis:** `Infrastructure/Non-GotSport Sources #3–5 — Candidate Synthesis and Recommendation.md`, all source research tasks filed April 23–27, and synthesis filed 2026-04-28.

---

## Ranked Priority Table

| Rank | Source | Classification | Est. Games/Year | Build Complexity | FORGE Recommendation | Reason |
|------|--------|----------------|-----------------|-----------------|---------------------|--------|
| **#3** | **SnapSoccer** | Non-GotSport engineering build | 5,000–7,000 ongoing + 10,000–20,000 historical | **Low-Medium** — 5.5–9.5 hrs test / 8–14 hrs prod; drops to 4–6 hrs if SincSports parser reusable | **BUILD — first non-GotSport sprint (May 17+)** | Highest score (18), completed design docs, short build window, Southeast gap not addressable via GotSport org-ID expansion |
| **#4** | **Affinity Soccer (IL)** | Non-GotSport engineering build | 5,000–15,000 (IL) + WI/MN/MO additive | **High** — 30–55 hrs; auth wall behavior unconfirmed | **HOLD — schedule browser audit first (June 2026)** | Highest absolute volume but access path unconfirmed; auth wall may add 15 hrs and require IYSA partnership |
| **#5** | **NAL** | Platform unconfirmed | < 1,000/yr est. (low confidence) | **Unknown** — 0 hrs if GotSport config add / 20–50 hrs if non-GotSport | **BROWSER CHECK FIRST** — if GotSport: exits this list as config add; if non-GotSport: deprioritize indefinitely | Volume too low to justify engineering sprint if non-GotSport; could be a 5-min config add |
| EXCLUDED | **Elite 64** | GotSport config add | Unknown — GotSport platform | 5-min config add | **Add in browser session** | GotSport-platform source; not an engineering sprint |
| EXCLUDED | **USL Academy** | GotSport config add (Source #2) | 2,000–5,000/yr | 5-min config add | **Stage 2 activation (May 17+)** — browser session pending | Already in Stage 2 pipeline; exits this engineering queue |
| EXCLUDED | **SincSports (generic)** | NO-GO as standalone | N/A | N/A | **NO-GO** — SnapSoccer IS the SincSports build | No standalone SincSports scraper pre-DSS; SnapSoccer uses SincSports infrastructure — building SnapSoccer = building the SincSports parser |
| EXCLUDED | **Tournament Director** | Export pipeline (operator import) | Scope undefined | Unknown — depends on export format | **DEFERRED — define scope before scheduling** | Not a public scraper; requires TD operator partnership or CSV export agreement; game volume unknown; re-evaluate post-DSS |

---

## FORGE Recommendation

**SnapSoccer should be built first.** It has the highest priority score (18 vs. 16 vs. 10), completed design documentation (`Infrastructure/SnapSoccer — Test Extraction Plan.md`, `Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md`), and a short single-session build window (5.5–9.5 hours) that the SincSports parser reuse assessment may compress further. The only remaining gate before scheduling the build session is a 20-minute browser check confirming `soccer.sincsports.com` page accessibility — this check can be folded into the existing TYSA/EDP/USL Academy browser session.

**Timing:** SENTINEL authorization needed by May 10 to hit the May 17+ post-DSS build window. Affinity Soccer (Source #4) requires a dedicated 60-minute browser access audit before any build commitment — schedule this for June 2026, not before.

---

## Build Sequence

| Phase | Source | Classification | Dependency | Target Start |
|-------|--------|----------------|------------|-------------|
| Config add (Stage 2) | EDP Soccer | GotSport config add | Browser session org-ID confirmation | May 17–20 (Stage 2 go-live) |
| Config add (Stage 2) | USL Academy | GotSport config add | Browser session org-ID confirmation | May 17–20 (Stage 2 go-live) |
| Config add (optional) | Elite 64 | GotSport config add | Browser session org-ID confirmation | Combine with Stage 2 config session |
| Config add (conditional) | NAL | GotSport config add if confirmed | 15-min browser platform check | As soon as browser check runs |
| **Source #3 (engineering)** | **SnapSoccer** | Non-GotSport build | Browser accessibility check + SENTINEL authorization | **May 17+ (post-DSS)** |
| Source #4 (engineering) | Affinity Soccer (IL) | Non-GotSport build | 60-min browser access audit | June 2026 |
| Source #5 (conditional) | NAL (non-GotSport path) | Non-GotSport build | Platform confirmed non-GotSport + volume reassessment | Post-DSS; only if platform confirmed non-GotSport and volume > 2,000/yr |
| Deferred | Tournament Director | Export pipeline | Scope definition + operator agreement | TBD post-DSS |

---

## Pre-Authorization Items (Presten Action Required)

| Item | Needed From | By Date | Est. Time |
|------|-------------|---------|-----------|
| SnapSoccer browser check (`soccer.sincsports.com` accessibility) | Presten browser | April 29 or May 1 | 20 min — fold into existing browser session |
| NAL platform identification | Presten browser | April 29 or May 1 | 15 min — fold into existing browser session |
| SENTINEL Source #3 build authorization | SENTINEL | May 10 | After browser check confirms access |
| Affinity Soccer access audit (dedicated browser session) | Presten browser | June 2026 | 60 min — dedicated session |

---

## SENTINEL Authorization Request

> FORGE recommends SnapSoccer as the first non-GotSport engineering source to build, with an estimated 5,000–7,000 ongoing games/year added to coverage plus 10,000–20,000 historical catch-up. FORGE requests SENTINEL review and Presten authorization before initiating the build session, targeted for May 17+ post-DSS. Authorization gate: (1) SnapSoccer browser check confirms `soccer.sincsports.com` accessibility, and (2) SENTINEL issues written authorization by May 10.

---

## References

- `Infrastructure/Non-GotSport Sources #3–5 — Candidate Synthesis and Recommendation.md` — full evaluation matrix and per-source analysis
- `Infrastructure/SnapSoccer — SincSports Parser Reuse Assessment.md` — build effort and reuse verdict
- `Infrastructure/SnapSoccer — Test Extraction Plan.md` — extraction design
- `Infrastructure/SnapSoccer Browser Check — Findings Report.md` — browser check template (fill before May 14)
- `Infrastructure/Affinity Soccer Source Research.md` — access path and volume estimates
- `Data Sources/NAL (North American League) — Platform Identification Research.md` — platform identification status
- `Infrastructure/Non-GotSport Source #2 — USL Academy Implementation Build Plan.md` — USL Academy Stage 2 plan

*FORGE — 2026-04-29*
