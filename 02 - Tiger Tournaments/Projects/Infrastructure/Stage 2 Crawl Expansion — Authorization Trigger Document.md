---
title: Stage 2 Crawl Expansion — Authorization Trigger Document
tags: [infrastructure, gotsport, stage2, crawl-expansion, authorization, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-26
author: FORGE
status: template — fill after May 1 first run and 3-day monitoring window (target: May 5)
task: task-2026-04-26-stage2-authorization-trigger-document
---

# Stage 2 Crawl Expansion — Authorization Trigger Document

> [!info] How to use
> FORGE fills Sections 1–3 after the May 1 first run and at least 3 days of pipeline monitoring (target: file by May 5). When complete, SENTINEL reads Sections 1–3 and makes a single authorization decision in Section 4. No multi-document cross-reference required.

---

## Status Summary

```
Last updated: 2026-04-26 (template — not yet filled)
Stage 1 first run: NOT YET EXECUTED
Section 1 metrics filled: 0 of 6
Section 2 blocking issues: —
Section 3 browser entities confirmed: 0 of 11
Authorization status: PENDING — fill after May 1 + 3-day window
```

---

## Section 1: Stage 1 Performance Results

> FORGE fills this after the May 1 first run and at least 3 days of pipeline monitoring. All "Required Threshold" values are pre-populated below — do not leave them blank at fill time.

| Metric | Required Threshold | Actual Result | Pass? |
|--------|--------------------|---------------|-------|
| First-run game count (new games in first 24h) | > 0 (basic pass); ≥ 500 for confidence in org-ID coverage | | ☐ |
| Run completion (no crash) | Last log line = `DAILY PIPELINE COMPLETED SUCCESSFULLY` on May 1 run | | ☐ |
| Active team ID count (STEP 1 log line) | 15,000–40,000 (if < 1,000 or > 100,000: FAIL — filter not working) | | ☐ |
| Source-level error rate | DLQ ≤ 2% of total teams attempted (from SYNC_RUN_SUMMARY); zero TX org-ID 403/404 errors | | ☐ |
| ECNL game ingestion | ≥ 1 confirmed new ECNL game in first 3 days (confirms TGS scraper active) | | ☐ |
| Duplicate rate | 0 duplicate rows in 24h window (query from Stage 2 Pre-Auth Criteria, G2) | | ☐ |

**Threshold sources:**
- First-run game count: `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` Section 2
- Run completion log line: `Infrastructure/Daily Pipeline Log Format Spec.md`
- Active team ID count: `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` Step 2
- DLQ rate: `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` G4
- Duplicate check: Stage 2 Pre-Auth Criteria G2 query
- ECNL ingestion: inferred from `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` Step 4

**FORGE: fill "Actual Result" and "Pass?" columns from Morning-After Runbook data and First-Week Monitoring Log.**

---

## Section 2: Stage 1 Blocking Issues Log

| Issue | Date Observed | Resolution | Resolved? |
|-------|---------------|------------|-----------|
| | | | |

> If no issues: replace table with "No blocking issues observed in Stage 1 monitoring window (May 1–[date])."

**Gate:** Any unresolved blocking issue in this table = Stage 2 NOT authorized, regardless of Section 1 metric results.

---

## Section 3: Browser Session Org-ID Status

> Pull current status from `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` Section 2 at fill time.

**Stage 2 gate:** All entities intended for Stage 2 must have confirmed org-IDs before expansion. Entities still `pending-browser-lookup` are excluded from Stage 2 scope until confirmed.

| # | Entity | Type | Org-ID Confirmed? | Added to Config? | Stage 2 Eligible? |
|---|--------|------|-------------------|------------------|-------------------|
| 1 | USL Academy | League | ☐ pending-browser-lookup | ☐ No | ☐ Not yet |
| 2 | EDP Soccer | League | ☐ pending-browser-lookup | ☐ No | ☐ Not yet |
| 3 | Elite 64 | League | ☐ pending-browser-lookup | ☐ No | ☐ Not yet |
| 4 | NAL | League | ☐ pending-browser-lookup — platform unconfirmed | ☐ No | ☐ Deferred until platform confirmed |
| 5 | Cal North Soccer (CA NorCal) | USYS State Assoc | ☐ pending-browser-lookup | ☐ No | ☐ Not yet |
| 6 | Cal South Soccer (CA SoCal) | USYS State Assoc | ☐ pending-browser-lookup | ☐ No | ☐ Not yet |
| 7 | Florida Youth Soccer Assoc (FYSA) | USYS State Assoc | ☐ pending-browser-lookup | ☐ No | ☐ Not yet |
| 8 | Eastern NY Youth Soccer (ENYYSA) | USYS State Assoc | ☐ pending-browser-lookup | ☐ No | ☐ Not yet |
| 9 | NJ Youth Soccer Assoc (NJYSA) | USYS State Assoc | ☐ pending-browser-lookup | ☐ No | ☐ Not yet |
| 10 | Washington Youth Soccer (WSYSA) | USYS State Assoc | ☐ pending-browser-lookup | ☐ No | ☐ Not yet |
| 11 | Eastern PA Youth Soccer (EPYSA) | USYS State Assoc | ☐ pending-browser-lookup | ☐ No | ☐ Not yet |

**FORGE: when Presten returns browser session results, update the Status column from `pending-browser-lookup` to `confirmed-[org_id]` or `not-on-gotsport` and update the Org-ID Master Reference simultaneously.**

---

## Section 4: Authorization Request to SENTINEL

> FORGE completes when Sections 1–3 are fully filled. Template is pre-written — fill in bracketed fields only.

```
Stage 2 Authorization Request
Date: [date — target May 5]
From: FORGE
To: SENTINEL

Stage 1 Performance: [PASS / FAIL]
- [N] of 6 metrics passed required thresholds
- [N] blocking issues observed; [all resolved / N unresolved] (see Section 2)
- First-run game count: [N] new games
- Duplicate rate: [0 / N] duplicates in first 24h

Browser Session Status: [N] of 11 entities confirmed with org-IDs. [N] still pending-browser-lookup.

Stage 2 Proposed Scope:
- Entities ready to add: [list entities where "Stage 2 Eligible = YES"]
- Estimated new game volume: [pull from Stage 2 Pre-Auth Criteria Section 1 — 60,000–120,000 games for P1+P2 states]
- Estimated crawl time: [pull from Stage 2 Pre-Auth Criteria Section 1 — ~4–20 hours depending on org count]
- Config change: Add [N] new org-IDs to crawl config per Infrastructure/GotSport Org-ID Config Edit Reference.md

FORGE Recommendation: [AUTHORIZE Stage 2 / HOLD Stage 2]
Reason: [one paragraph — cite specific metrics from Section 1 and gate conditions met/not met]

SENTINEL decision requested by: [date — to hit the Stage 3 window; suggest May 5–6]
```

---

## Section 5: Post-Authorization Steps (FORGE reference)

If SENTINEL authorizes Stage 2, FORGE executes in order:

1. Edit crawl config to add Stage 2 org-IDs — use `Infrastructure/GotSport Org-ID Config Edit Reference.md`
2. Update `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — move authorized entities from Section 2 (pending) to Section 1 (active), add `date_added` field
3. Update this document status to `authorized` and add `authorized_date`
4. Monitor next pipeline run for Stage 2 org-ID game ingestion
5. File Stage 2 first-run counts in next FORGE briefing
6. Run post-edit verification query (from Org-ID Config Edit Reference, Section 4) 24 hours after adding each new org-ID

---

## Gate Conditions Summary (for FORGE reference)

From `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md`:

| Gate | Condition | Status |
|------|-----------|--------|
| G1 | Second TX crawl delta = 0 (Stage 1 stabilized) | ☐ pending Stage 1 execution |
| G2 | Zero duplicate games from Stage 1 | ☐ pending Stage 1 execution |
| G3 | Zero 403/404 errors for Stage 1 org-IDs | ☐ pending Stage 1 execution |
| G4 | DLQ ≤ 2% in Stage 1 run | ☐ pending Stage 1 execution |
| G5 | FORGE issued explicit Stage 2 authorization in Stage 1 Results Report | ☐ pending — see `Reports/gotsport-org-expansion-stage1-results-[date].md` |

All 5 must be GREEN before Section 4 can read AUTHORIZE.

---

## References

- `Infrastructure/GotSport Org-ID Expansion Stage 2 — Pre-Authorization Criteria.md` — G1–G5 gate conditions
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — threshold queries
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — GREEN/YELLOW/RED criteria
- `Infrastructure/GotSport Org-ID Master Reference — April 2026.md` — browser session entity status
- `Infrastructure/GotSport Org-ID Config Edit Reference.md` — how to add Stage 2 org-IDs
- `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md` — primary data source for Section 1

*FORGE — 2026-04-26*
