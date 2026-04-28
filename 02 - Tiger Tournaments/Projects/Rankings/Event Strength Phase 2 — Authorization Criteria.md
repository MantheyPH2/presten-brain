---
title: Event Strength Phase 2 — Authorization Criteria
type: authorization-criteria
author: ELO
date: 2026-04-27
status: pre-authorization — all gates pending completion
related: "[[Event Strength Phase 2 — Prerequisites and Scope]]", "[[Event Strength Phase 1 — SENTINEL Authorization Criteria]]", "[[Post-DSS Calibration Sprint Plan]]"
tags: [event-strength, phase-2, authorization, criteria, evo-draw]
---

# Event Strength Phase 2 — Authorization Criteria

> **Purpose:** Pre-define the gates SENTINEL checks when ELO presents a Phase 2 authorization request. Phase 1 authorization was deliberative because criteria were not pre-defined. Phase 2 authorization will be a criteria-check using this document.
>
> **Target authorization window:** June 5–9, 2026. Phase 2 compute targeted June 9–15.
>
> **Section 6 is intentionally blank** — reserved for SENTINEL pre-approval disposition. That is correct and expected.

---

## Section 1 — Phase 1 Completion Gate

Phase 2 **cannot begin** until Phase 1 is confirmed complete. "Phase 1 complete" means:

- [ ] Event strength scores computed for all Phase 1 leagues (per `Rankings/Event Strength Phase 1 — SENTINEL Authorization Criteria.md` scope)
- [ ] Phase 1 post-compute validation cleared: all 4 ELO validation queries (Q1–Q4 from `Rankings/Event Strength Phase 1 — ELO Execution Package.md`) returned PASS
- [ ] ELO Phase 1 Clearance Filing confirmed in ELO briefing (format: "Event Strength Phase 1 — ELO Validation Clearance / Date: [date] / Q1–Q3: PASS")
- [ ] SENTINEL issued Phase 1 clearance in a briefing (cite date when filled): ___________

**Hard stop:** If Phase 1 is not fully cleared, Phase 2 authorization request is rejected regardless of other criteria. ELO should not submit a Phase 2 request until Phase 1 clearance is filed.

---

## Section 2 — Data Coverage Gate

Phase 2 applies `event_strength.strength_percentile` as a K-factor multiplier. Meaningful K-factor adjustment requires that `strength_percentile` values are statistically reliable — games-thin events produce volatile percentile estimates.

Minimum data requirements for Phase 2 leagues (drawn from `Rankings/Event Strength Phase 2 — Prerequisites and Scope.md` §3):

| League / Tier Group | Minimum Games in DB (Phase 1 computed) | Confidence Threshold | Source Status Required |
|--------------------|----------------------------------------|---------------------|------------------------|
| ECNL (Girls + Boys) | 2,000 games | High — at confidence | Daily pipeline active (post-May-1) |
| MLS NEXT (Boys all tiers) | 1,500 games | High — at confidence | Daily pipeline active |
| GA / GA ASPIRE (Girls) | 500 games | Medium — Phase 2 K-adjustment modest for thin sources | Historical backfill sufficient + daily pipeline |
| NPL / DPL | 800 games | Medium | Historical backfill sufficient |
| ECNL RL | 600 games | Medium | Historical backfill sufficient |
| State Premier / Regional circuits | 300 games per league group | Low — capped K-factor adjustment for thin tiers | Historical backfill acceptable |
| USL Academy / EDP / Elite 64 / NAL | 200 games (post-Tier-2 calibration fix) | Low | Historical + daily; apply Phase 2 only if Tier 2 cal values have been applied first |

**If any Phase 2 league falls below its minimum at authorization time:** That league is deferred to Phase 3 (future quarterly Phase 2 expansion). Phase 2 proceeds for leagues meeting threshold. ELO documents any deferred leagues in the authorization request.

**Threshold for "is this meaningful?":** ELO's position is that `strength_percentile` is unreliable for any event with fewer than 12 participating teams. Any event with < 12 teams is excluded from the Phase 2 K-factor multiplier regardless of game count — treated as `strength_percentile = 50` (no K-factor adjustment). This is a safety floor, not a gate condition.

---

## Section 3 — Pipeline Health Gate

Phase 2 event strength scores update with each daily pipeline run. Phase 2 should not launch until the pipeline is confirmed stable enough to sustain those updates.

Required before Phase 2 authorization:

- [ ] May 1 first pipeline run: GREEN signal — confirmed in the Post-May-1 Rankings Impact baseline (due May 4)
- [ ] 4-day pipeline aggregate (May 1–4): ≥ 700 games ingested — per FORGE May 2–4 monitoring checklist
- [ ] No active RED pipeline signals as of the authorization request date — check most recent FORGE briefing
- [ ] `ecnl_verified = TRUE` being written for new ECNL games (confirmed by FORGE by May 4–6 per the ECNL staleness monitoring spec)

**Conditional:** If pipeline is YELLOW (500–699 games in the May 1–4 window) but otherwise stable, Phase 2 may proceed for leagues that do not depend on GotSport API for recent game ingestion.

**GotSport-API-dependent leagues in Phase 2 scope** (require GREEN pipeline for these specifically):
- ECNL Boys (GotSport-sourced for Northeast and Mid-Atlantic)
- NPL / ECNL RL for GotSport-covered regions

If these leagues are YELLOW-threshold, ELO notes them in the authorization request and recommends a 2-week hold until ingestion confirms stable.

---

## Section 4 — Ratings Stability Gate

Phase 2's K-factor multiplier feeds back into team ratings. The base ratings must be stable before Phase 2 adds another layer of variation.

Required:

- [ ] Girls GA ASPIRE calibration stability confirmed by the monitoring spec (≥ 5 consecutive runs within daily drift threshold post-April-28 fix). ELO files stability clearance note in briefing: "GA ASPIRE calibration stability confirmed as of [date]."
- [ ] No active Level 2 or Level 3 calibration instability flags in ELO's briefings as of the authorization request date
- [ ] U13/U14 calibration fix (Sprint 1, May 17) deployed and validated — ELO validation clearance filed
- [ ] MLS NEXT Boys tier split deployed and validated — ELO validation clearance filed
- [ ] 7-day post-calibration stability window after the last Sprint 1/2 calibration deployment: no nightly pipeline run showing average rating shift > 3 pts across any age group during the monitoring window

**Boys calibration state at authorization:**

Phase 2 affects both Boys and Girls ratings (gender-separated pools within Phase 1).

- If Boys Option A = **PASS + execution complete**: Phase 2 runs over the validated Boys baseline. No restriction.
- If Boys Option A = **FAIL + Girls-Only fallback active**: Phase 2 runs over Girls only. The Boys K-factor multiplier is not applied until Boys calibration is resolved. ELO notes this scope restriction in the authorization request.
- If Boys Option A investigation is still **active / INCONCLUSIVE**: Phase 2 is held entirely. K-factor scaling should not run over an unresolved Boys baseline.

**Does Boys instability block Phase 2 entirely?** ELO's position: Only if Boys Option A is INCONCLUSIVE or active investigation. If Boys Option A = FAIL and Girls-Only fallback is active, Phase 2 can proceed for Girls with Boys excluded. The Girls and Boys Phase 2 computations are independent.

---

## Section 5 — Authorization Request Format

When all 4 gates are met, ELO sends SENTINEL the following in a briefing:

```
ELO Phase 2 Authorization Request — [date]

Gate 1 — Phase 1 Completion: PASS.
  Phase 1 cleared on [date]. ELO clearance note filed [date].
  SENTINEL clearance filed [date].

Gate 2 — Data Coverage: PASS.
  All Phase 2 leagues meet minimum game counts.
  Deferred leagues (below minimum): [list, or "none"].
  Source status: Daily pipeline active post-May-1. ecnl_verified confirmed [date].

Gate 3 — Pipeline Health: PASS.
  May 1–4 aggregate: [N] games. Status: GREEN.
  No active RED signals as of [date].
  GotSport-dependent leagues: [GREEN / YELLOW — see note].

Gate 4 — Ratings Stability: PASS.
  GA ASPIRE stability clearance: [date].
  U13/U14 fix validation clearance: [date].
  MLS NEXT split validation clearance: [date].
  Post-calibration 7-day window: clean from [date] to [date].
  Boys status: [Option A PASS — full Phase 2 / Option A FAIL — Girls-only Phase 2].
  No active instability flags.

Requested authorization: Event Strength Phase 2 compute for [league list].
Boys scope: [full / Girls-only].
Expected compute time: 6–8 hours (FORGE session).
Deliverable: Phase 2 event_strength.strength_percentile applied to K-factor in compute-rankings.js.
  ELO validation clearance filed within 48 hours of Phase 2 recompute completion.
```

SENTINEL responds with authorization or hold in the next briefing.

---

## Section 6 — SENTINEL Pre-Approval

_Left for SENTINEL to fill. Pre-approval here means that when ELO submits the authorization request in Section 5 format with all 4 gates confirmed, SENTINEL's authorization is a criteria-check (not a new deliberation)._

```
SENTINEL Pre-Approval — Event Strength Phase 2

Date reviewed: ___________
Pre-approval disposition: APPROVE (criteria-check) / REQUIRES ADDITIONAL REVIEW
If additional review: condition: ___________

Notes: ___________
```

---

## References

- `Rankings/Event Strength Phase 1 — SENTINEL Authorization Criteria.md` — structural template for this document
- `Rankings/Event Strength Phase 2 — Prerequisites and Scope.md` — prerequisite conditions this document operationalizes
- `Rankings/Event Strength Phase 2 — Authorization Evidence Package.md` — evidence package for Section 1
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — stability monitoring providing Section 4 data
- `Rankings/Post-DSS Calibration Sprint Plan.md` — Phase 2 position in the post-DSS sprint sequence
- `Infrastructure/May 2–4 Pipeline Monitoring Checklist.md` — pipeline health data for Section 3
- `Rankings/Event Strength Phase 1 — ELO Execution Package.md` — Phase 1 validation queries

*ELO — 2026-04-27*
