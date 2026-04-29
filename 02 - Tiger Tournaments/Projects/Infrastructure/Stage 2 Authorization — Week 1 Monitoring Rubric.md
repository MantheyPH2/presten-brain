---
title: "Stage 2 Authorization — Week 1 Monitoring Rubric"
tags: [infrastructure, pipeline, stage2, authorization, monitoring, sentinel, forge, elo]
created: 2026-04-28
updated: 2026-04-28
author: FORGE
status: filed — SENTINEL fills Section 6 on authorization decision (~May 4–5)
task: task-2026-04-28-stage2-monitoring-rubric-definition
due: 2026-05-05
---

# Stage 2 Authorization — Week 1 Monitoring Rubric

> [!info] Purpose
> Single rubric document for SENTINEL's Stage 2 GO/NO-GO decision (~May 4–5). SENTINEL reads Sections 2–5, checks results against thresholds, and issues the authorization verdict in Section 6. All thresholds are drawn from existing filed documents — no new decisions required from FORGE.

**Filed by:** FORGE — 2026-04-28  
**Authorization decision date:** ~May 4–5, 2026 (after 3 Stage 1 runs: May 1, 2, 3)  
**Authorization issued by:** SENTINEL  

---

## Section 1: Stage 2 Authorization Decision Context

| Item | Value |
|------|-------|
| Authorization decision date | ~May 4–5 (after 3 Stage 1 runs: May 1, May 2, May 3) |
| Authorization issued by | SENTINEL |
| Stage 2 go-live target | ~May 17–20 |
| Sources in Stage 2 | EDP Soccer (Source #1), USL Academy (Source #2), TYSA expansion org-IDs |
| Sources NOT in Stage 2 | SnapSoccer (#3), Affinity Soccer (#4), NAL (#5 conditional) — these require separate authorization after DSS |
| Stage 2 config type | GotSport API only (EDP and USL Academy both reuse existing parser — config add, no code changes) |

**Stage 2 is NOT authorized until all of Sections 2, 3, 4, and 5 show GREEN or acceptable status in Section 6.**

---

## Section 2: Game Volume Rubric

**Source:** `Infrastructure/Pipeline Week 1 — Game Volume Actuals Report.md` — Sections 1 and 5

FORGE fills the Actuals column after 3 Stage 1 runs. SENTINEL reads the verdict column.

| Source | Expected Games (Days 1–3) | GREEN Floor | YELLOW Range | RED Trigger |
|--------|--------------------------|-------------|--------------|-------------|
| TYSA (GotSport API) | 3,500–7,000 (est. 60% annual volume by May) | ≥5,000 games first run | 500–4,999 — investigate before Stage 2 | <500 — diagnose before proceeding |
| gotsport_api (cumulative Stage 1) | Per Week 1 Actuals baseline | Per `Pipeline Week 1 — Game Volume Actuals Report.md` Section 1 | Same doc | Same doc |
| tgs_athleteone (existing ECNL source) | No expected change — existing ingestion | Stable (within 10% of pre-May-1 daily average) | >10% drop from baseline | 0 games any run |
| tgs (existing ECNL source) | No expected change | Stable | >10% drop | 0 games any run |
| **Total Stage 1 Volume (aggregate)** | Dominated by TYSA first-run volume | **≥6,400 total games over Days 1–3** | 1,500–6,399 — review before authorizing | <1,500 — RED; Stage 2 held |

**Source:** Aggregate GREEN floor (≥6,400) from `Infrastructure/Pipeline Week 1 — Game Volume Actuals Report.md` Section 1.  
**TYSA-specific GREEN floor** (≥5,000 first run) from `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` Section 3.

**SENTINEL threshold for Stage 2 authorization:**
- All sources GREEN overall → Stage 2 authorized as scheduled
- Any source YELLOW → SENTINEL reviews specific source before authorizing; may authorize conditionally
- Any source RED → Stage 2 held pending FORGE investigation and remediation report

---

## Section 3: Pipeline Health Rubric

**Source:** `Infrastructure/Source Health Monitoring Queries.md` (filed from `task-2026-04-24-source-health-monitoring-queries.md`)

FORGE monitors and fills Actuals column over Days 1–3. SENTINEL reads verdict.

| Health Check | GREEN Condition | YELLOW Condition | RED Condition |
|-------------|----------------|-----------------|--------------|
| Failed crawl runs (Days 1–3) | 0 failed runs | 1 failed run — recovered without data loss | 2+ failed runs OR 1 unrecovered failure |
| Ingestion recency (all sources) | All sources ingested within last 7 days | Any source last ingested 7–30 days ago | Any source last ingested >30 days ago |
| Duplicate game rate | 0 duplicate source_id rows post-run | Any duplicates — investigate before Stage 2 | Any duplicates unresolved within 24 hrs |
| Stale game rate (not updated >48 hrs) | <5% of Stage 1 games older than 48 hrs | 5–15% stale | >15% stale |
| Pipeline error log volume | 0 parse errors OR <5% error rate per run | 5–15% error rate — review before Stage 2 | >15% error rate OR any critical error |
| New source game confirmation (TYSA) | Games from TYSA org-ID confirmed within 48 hrs of config add | Games appear >48 hrs after config add | No games from TYSA after 72 hrs |

**SENTINEL threshold for Stage 2 authorization:**
- All GREEN → Stage 2 authorized
- Any YELLOW → SENTINEL reviews; may authorize conditionally with FORGE remediation plan
- Any RED → Stage 2 held; FORGE must file a failure investigation report before SENTINEL re-evaluates

---

## Section 4: Ranking Signal Health Rubric (ELO)

**Source:** `Infrastructure/May 1 Pipeline Launch — ELO Rankings Monitoring Plan.md` — R1–R5 checks

ELO fills the Actuals column for each check over Days 1–3. SENTINEL reads verdict.

| ELO Check | Check ID | GREEN Threshold | YELLOW Condition | RED Condition | SENTINEL Action if YELLOW/RED |
|-----------|----------|-----------------|-----------------|--------------|-------------------------------|
| New Game Volume per run | R1 | ≥100 new games per post-run check | 50–99 games | <50 games | Confirm TYSA crawl is running; check source_org_id counts |
| Average Rating Shift | R2 | <10 pts average shift across all teams | 10–25 pts average | >25 pts average | ELO files anomaly report before SENTINEL issues authorization |
| Large Mover Count | R3 | 0–5 teams moved >50 pts in a single run | 6–15 large movers | >15 large movers OR any team >200 pts shift | Stage 2 held; ELO investigates specific movers |
| Source Distribution | R4 | ≥3 active org-ID sources producing games | 2 sources active | 1 or 0 sources active | FORGE investigates inactive source; Stage 2 held |
| Rating Variance (stddev vs. baseline) | R5 | Within 5% of pre-launch stddev baseline | 5–15% deviation | >15% deviation from baseline | ELO files variance investigation; Stage 2 held |

**ELO baseline required:** R5 requires a pre-launch stddev baseline captured before May 1 first run. See `Infrastructure/May 1 Stage 1 — Final Launch Authorization Checklist.md` Section 4.

**SENTINEL threshold for Stage 2 authorization:**
- All R1–R5 GREEN → authorized
- Any R-check YELLOW → SENTINEL may authorize conditionally; ELO must confirm YELLOW is within acceptable range
- Any R-check RED → Stage 2 held; ELO must file resolution document before SENTINEL re-evaluates

---

## Section 5: Stage 2 Source Readiness Pre-Checks

Before Stage 2 can launch, these source-level gates must pass regardless of Week 1 volume data.

**Source:** `Infrastructure/Non-GotSport Source #1 — Implementation Build Plan.md` and `Infrastructure/Non-GotSport Source #2 — USL Academy Implementation Build Plan.md`

| Source | Gate | Status at Authorization | Notes |
|--------|------|------------------------|-------|
| EDP (Source #1) | Day 0 config activated — browser session required | [ ] YES / [ ] NO / [ ] PENDING | EDP org-ID is pending browser confirmation. Stage 2 cannot include EDP without confirmed org-ID. |
| EDP (Source #1) | Gate 1 — Parse quality: 0 critical parse errors, game count 3,000–8,000/year range | [ ] PASS / [ ] FAIL / [ ] NOT YET RUN | Gate 1 runs after Day 1 EDP first pipeline run. |
| EDP (Source #1) | Gate 2 — ELO ranking impact: no team >200 pts shift from EDP games alone | [ ] PASS / [ ] FAIL / [ ] NOT YET RUN | Gate 2 is ELO's check. Tighter threshold than general R3. |
| USL Academy (Source #2) | Day 0 config activated — browser session required | [ ] YES / [ ] NO / [ ] PENDING | USL Academy org-ID pending browser confirmation. Stage 2 cannot include USL Academy without confirmed org-ID. |
| USL Academy (Source #2) | ELO boys threshold: no boys team >50 pts shift from USL Academy games | [ ] PASS / [ ] FAIL / [ ] NOT YET RUN | Boys threshold is tighter than EDP (>50 pts vs. >200 pts) due to active calibration window. |
| USL Academy (Source #2) | ELO written confirmation required before Gate 2 | [ ] RECEIVED / [ ] PENDING | FORGE does not proceed to USL Academy Gate 2 without ELO written confirmation. |

**If browser session not completed before May 4–5:** EDP and USL Academy rows remain PENDING. SENTINEL may authorize Stage 2 as CONDITIONAL — browser session must complete before FORGE adds org-IDs to config. This conditional window closes May 17 (Stage 2 go-live target).

---

## Section 6: SENTINEL Authorization Block

```
SENTINEL STAGE 2 REVIEW — ~May 4–5, 2026

Section 2 Game Volume:
[ ] GREEN — all sources at or above GREEN floor
[ ] YELLOW — one or more sources in YELLOW band (notes below)
[ ] RED — one or more sources below RED threshold (Stage 2 HOLD)

Section 3 Pipeline Health:
[ ] GREEN — all health checks at GREEN
[ ] YELLOW — minor issues, within acceptable range (notes below)
[ ] RED — critical pipeline issue (Stage 2 HOLD)

Section 4 ELO Rankings (R1–R5):
[ ] GREEN — all five checks GREEN
[ ] YELLOW — one or more checks in YELLOW, ELO has confirmed acceptable (notes below)
[ ] RED — one or more checks RED (Stage 2 HOLD)

Section 5 Source Readiness:
  EDP (Source #1): [ ] READY  [ ] NOT READY (org-ID pending)  [ ] CONDITIONAL
  USL Academy (Source #2): [ ] READY  [ ] NOT READY (org-ID pending)  [ ] CONDITIONAL

Notes (SENTINEL fills if any YELLOW/CONDITIONAL):
___________________________________________________________________________

SENTINEL VERDICT:
[ ] STAGE 2 AUTHORIZED — target go-live May 17–20
[ ] STAGE 2 HOLD — reason: ___________
[ ] STAGE 2 CONDITIONAL — conditions that must be met before go-live:
    ___________________________________________________________________________

Signed: SENTINEL
Date: ___________
```

---

## References

- `Infrastructure/Pipeline Week 1 — Game Volume Actuals Report.md` — Section 1 (GREEN/YELLOW/RED floors), Section 5 (Stage 2 auth criteria)
- `Infrastructure/May 1 Pipeline Launch — ELO Rankings Monitoring Plan.md` — R1–R5 checks and thresholds
- `Infrastructure/Source Health Monitoring Queries.md` — health check query patterns and thresholds
- `Infrastructure/Non-GotSport Source #1 — Implementation Build Plan.md` — EDP gate criteria
- `Infrastructure/Non-GotSport Source #2 — USL Academy Implementation Build Plan.md` — USL Academy gate criteria and ELO threshold
- `Infrastructure/May 1 Stage 1 — Final Launch Authorization Checklist.md` — Stage 1 authorization gate (prerequisite for this rubric)
- `Infrastructure/Stage 2 Expansion — SENTINEL Authorization Gate.md` — broader Stage 2 authorization framework

*FORGE — 2026-04-28*
