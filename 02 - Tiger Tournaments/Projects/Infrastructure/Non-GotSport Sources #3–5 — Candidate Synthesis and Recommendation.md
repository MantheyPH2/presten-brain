---
title: Non-GotSport Sources #3–5 — Candidate Synthesis and Recommendation
type: decision-document
agent: FORGE
filed: 2026-04-28
status: complete
due: 2026-04-30
sentinel_action_required_by: 2026-05-10
---

# Non-GotSport Sources #3–5 — Candidate Synthesis and Recommendation

> **Scope:** This document covers Sources #3 through #5 in the non-GotSport ingestion roadmap. Sources #1 (EDP) and #2 (USL Academy) are GotSport-platform sources with build plans already filed. This document ranks the remaining engineering candidates for post-DSS implementation. **Synthesized from existing research — no new investigation.**

**Filed by:** FORGE — 2026-04-28  
**Basis:** `Data Sources/Non-GotSport Source Priority Matrix — April 2026.md`, `Data Sources/Non-GotSport-Source-Priority-Rerank-2026-04-27.md`, and all source research tasks filed April 23–27.

---

## Section 1: Candidate Sources Evaluated

| Source | Research Document(s) | Last Updated | Classification |
|--------|------------------|--------------|----------------|
| SnapSoccer | `task-2026-04-23-snapsoccer-source-research.md`, `task-2026-04-27-snapsoccer-test-extraction-plan.md`, `task-2026-04-27-sincsports-parser-reuse-check.md` | 2026-04-27 | Non-GotSport engineering sprint |
| Affinity Soccer (IL) | `task-2026-04-24-affinity-soccer-source-research.md` | 2026-04-27 | Non-GotSport engineering sprint |
| NAL (North American League) | `task-2026-04-25-elite64-nal-source-research.md` | 2026-04-25 | Platform unconfirmed — may be GotSport config add |
| Elite 64 | `task-2026-04-25-elite64-nal-source-research.md` | 2026-04-25 | **GotSport-platform source — excluded from this list** (see note below) |
| SincSports (generic) | `task-2026-04-27-sincsports-go-no-go-decision-package.md` | 2026-04-27 | NO-GO as standalone source — covered via SnapSoccer build |

**Note on Elite 64:** Elite 64 operates on GotSport. It is a 5-minute config add, not an engineering sprint. It should be prioritized in the same browser session as EDP, USL Academy, and TYSA. It does not belong in this engineering queue.

**Note on SincSports (generic):** SincSports as a platform is the basis for the SnapSoccer ingestion build. The "SincSports NO-GO" decision (filed 2026-04-27) means: no general SincSports scraper for non-SnapSoccer events pre-DSS. SnapSoccer (running on SincSports infrastructure) is Source #3.

---

## Section 2: Evaluation Matrix

**Scoring Formula:** Priority Score = (Volume × 3) + (Build Efficiency × 2) + (Access Confidence × 2) + (Strategic Value × 1)

| Dimension | Score 1 | Score 2 | Score 3 |
|-----------|---------|---------|---------|
| Volume (net new games/yr) | Low: < 2,000 | Medium: 2,000–8,000 | High: > 8,000 |
| Build Efficiency (lower effort = higher score) | Low: > 30 hrs | Medium: 10–30 hrs | High: < 10 hrs |
| Access Confidence | Low: unknown or auth-gated | Medium: likely accessible, browser check pending | High: confirmed public access |
| Strategic Value | Low: niche coverage | Medium: fills regional gap | High: largest coverage gap, DSS-relevant |

| Source | Net New Games/Yr | Build Effort | Access Confidence | Strategic Value | Vol | Eff | Access | Strat | **Score** |
|--------|-----------------|--------------|-------------------|-----------------|-----|-----|--------|-------|-----------|
| **SnapSoccer** | 5,000–7,000 ongoing + 10,000–20,000 historical | 5.5–9.5 hrs (test) / 8–14 hrs (prod) | Medium — browser check pending; public pages likely accessible | Medium — fills SE gap not addressable via GotSport | 2 | 3 | 2 | 2 | **18** |
| **Affinity Soccer (IL)** | 5,000–15,000/yr (IL); WI/MN/MO additive | 30–50 hrs (no auth) / 35–55 hrs (session auth) | Low-Medium — access path unconfirmed; auth wall behavior unknown | High — IL = 50,000+ players; largest single non-GotSport coverage gap | 3 | 1 | 1 | 3 | **16** |
| **NAL** | < 1,000/yr (estimated; low confidence) | 0 hrs if GotSport (config add) / 20–50 hrs if non-GotSport | Unknown — platform not confirmed | Low — small national league; limited calibration uplift | 1 | 2* | 1 | 1 | **10** |

*NAL efficiency score: averaged between GotSport scenario (Score 3) and non-GotSport scenario (Score 1) pending platform confirmation.*

---

## Section 3: Ranked Recommendation

### Source #3: SnapSoccer (Post-DSS May 17+)

- **Coverage impact:** Gulf Coast and Southeast competitive tournaments — fills SE gap that GotSport org-ID expansion cannot address. Historical catch-up estimated 10,000–20,000 games; ongoing ~5,000–7,000/yr. Leagues confirmed visible in prior research: GSSL, SE regional playdates, Gulf Coast club tournaments.
- **Access path:** Public HTML pages on `soccer.sincsports.com` via SnapSoccer event links. Single browser check (20 min) resolves final accessibility confirmation. No auth wall confirmed in prior research; Cloudflare risk is on `sincsports.com` main domain (not the schedule subdomain).
- **Build effort:** 5.5–9.5 hrs test run / 8–14 hrs production. Drops to 4–6 hrs (test) if existing SincSports parser is reusable (REUSE verdict pending — `task-2026-04-27-sincsports-parser-reuse-check.md`).
- **Parser reuse:** HIGH potential. Existing SincSports bulk import code in Evo Draw codebase. Reuse assessment filed — confirms same HTML table structure. Adaptation estimated < 2 hrs if structure matches.
- **Key risk:** SnapSoccer accessibility gate (5-minute browser check) must be confirmed before build session. If `soccer.sincsports.com` requires auth → build effort increases to REBUILD range (8–12 hrs) and timeline extends.
- **SENTINEL authorization needed before:** Build session scheduled (target May 17+). Authorization after: (1) browser check confirms access, and (2) SincSports parser reuse verdict filed.

---

### Source #4: Affinity Soccer — Illinois (June 2026+)

- **Coverage impact:** Highest absolute game volume of any non-GotSport source. Illinois Youth Soccer Association (IYSA) and affiliated leagues — estimated 5,000–15,000 games/yr for IL alone. WI, MN, MO may be additive. IL has no GotSport presence for state league play, making this a full-coverage gap (not incremental).
- **Access path:** `affinitysoccer.com` platform. Access path research complete (April 27) — CONDITIONAL verdict. Auth wall behavior on IYSA result pages unconfirmed; a 60-minute browser audit is required before build can be scoped accurately.
- **Build effort:** 30–50 hrs (no auth wall) / 35–55 hrs (session auth required). Team identity resolution adds architecture complexity not present in SnapSoccer — IL teams have no GotSport `org_id` mapping, requiring a new team-matching layer.
- **Parser reuse:** NONE — Affinity Soccer uses a proprietary platform (Active Network / Blue Star Sports). No existing Evo Draw parser reusable. Full new build required.
- **Key risk:** Auth wall behavior is the primary unknown. If IYSA results require login: build effort increases by 5–15 hrs and may require a partnership agreement with IYSA or Affinity. This determination requires the browser audit before any build commitment.
- **SENTINEL authorization needed before:** Browser audit session (June 2026). FORGE recommends a 1-hour dedicated access audit as the first Affinity Soccer session — not a build session.

---

### Source #5: NAL — North American League (Conditional / Post-DSS)

- **Coverage impact:** National development league. Volume estimated < 1,000 games/yr at low confidence. If NAL is on GotSport: becomes a config add (exits this list entirely); if non-GotSport: game volume is too low to justify a dedicated engineering sprint vs. other priorities.
- **Access path:** Unknown — platform not confirmed. Medium-confidence hypothesis: GotSport. Browser check (15 min, combinable with existing browser session for EDP/USL Academy/TYSA/Elite 64) resolves this definitively.
- **Build effort:** 0 hrs if GotSport (config add) / 20–50 hrs if non-GotSport platform identified.
- **Parser reuse:** Cannot assess until platform confirmed.
- **Key risk:** Entirely contingent on platform identification. If GotSport → Source #5 exits this engineering list and becomes a config add in the browser session. If non-GotSport with volume < 1,000/yr → deprioritize indefinitely and do not schedule a build session.
- **SENTINEL authorization needed before:** Browser check (combine with existing session). No engineering authorization needed until platform confirmed and volume reassessed.

---

## Section 4: Sources Not Recommended (with Reason)

| Source | Reason Not Recommended for Sources #3–5 |
|--------|------------------------------------------|
| Elite 64 | GotSport-platform source — 5-minute config add, not engineering sprint. Exits this list; handle in browser session. |
| SincSports (generic) | NO-GO as standalone source pre-DSS. SnapSoccer is the correct use case for SincSports platform knowledge. Post-DSS, SnapSoccer implementation IS the SincSports parser build. |
| USYS State Cup (non-TX states) | Covered incrementally by GotSport org-ID expansion per state. Not a distinct non-GotSport engineering effort. Re-evaluate only if a state runs its State Cup on a non-GotSport platform. |
| TopDrawerSoccer / recruiting platforms | Editorial content, not game result data. No ingestible score data in structured format. No viable ingestion path. |
| Demosphere-hosted leagues | No Demosphere sources identified with > 2,000 games/yr and confirmed public access. |

---

## Section 5: Decision Required from SENTINEL/Presten

**SENTINEL must confirm Source #3 (SnapSoccer) authorization by May 10** to allow FORGE to begin detailed build planning for the May 17 session window.

**Presten pre-authorization items needed before May 17:**

| Item | Needed From | By Date | Note |
|------|-------------|---------|------|
| SnapSoccer accessibility browser check (~20 min) | Presten browser | April 29 or May 1 (combine with TYSA/EDP session) | 5-min check; can be folded into existing browser session |
| SincSports parser reuse verdict confirmation | FORGE internal code read | Before May 10 | Assessment filed; FORGE confirms verdict stands before build session |
| SENTINEL Source #3 build authorization | SENTINEL | May 10 | After browser check confirms access |
| NAL platform identification check (~15 min) | Presten browser | April 29 or May 1 (combine with existing browser session) | May move NAL from this list to config-add list |
| Affinity Soccer access audit (60 min) | Presten browser (dedicated session) | June 2026 | Not urgent; schedule after SnapSoccer build completes |

---

## FORGE Recommendation Summary

| Source | Rank | Action | Window |
|--------|------|--------|--------|
| SnapSoccer | #3 | Build now post-DSS — single session (5.5–9.5 hrs). Gate: browser check + SENTINEL auth. | May 17+ |
| Affinity Soccer (IL) | #4 | Schedule browser access audit first (1 hr), then build. Do not start engineering without access confirmation. | June 2026 |
| NAL | #5 | Run 15-min browser check. If GotSport → config add (exits this list). If non-GotSport + < 1,000 games/yr → deprioritize. | Browser check April 29 / May 1 |

**SnapSoccer is the correct #3 source.** The matrix score (18 vs. 16 vs. 10), the completed design documentation, and the short accessibility gate all confirm this. Affinity Soccer has the highest absolute volume potential but is not schedulable before June 2026 due to the access audit requirement.

---

## References

- `Data Sources/Non-GotSport Source Priority Matrix — April 2026.md` — scoring matrix and sequencing (this document supersedes the April 27 rerank)
- `Data Sources/Non-GotSport-Source-Priority-Rerank-2026-04-27.md` — prior rerank
- `Data Sources/SincSports-GoNoGo-Decision-Package-2026-04-27.md` — SincSports build effort and coverage
- `Infrastructure/Non-GotSport Source Implementation Priority List.md` — Sources #1 and #2 (EDP and USL Academy)
- `task-2026-04-27-sincsports-go-no-go-decision-package.md` — most complete single-source analysis
- `task-2026-04-27-non-gotsport-source-priority-matrix.md` — matrix basis
- `task-2026-04-27-non-gotsport-source-rerank-post-sincsports.md` — updated ranking post-SincSports decision

*FORGE — 2026-04-28 @ 17:25*
