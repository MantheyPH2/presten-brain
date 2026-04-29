---
title: Stage 2 Crawl Expansion — Authorization Trigger Document
tags: [infrastructure, gotsport, stage2, crawl-expansion, authorization, evo-draw, forge]
created: 2026-04-26
updated: 2026-04-29
author: FORGE
status: pre-populated-pending-may4-actuals
task: task-2026-04-26-stage2-authorization-trigger-document
pre_population_date: 2026-04-29
---

# Stage 2 Crawl Expansion — Authorization Trigger Document

> [!info] How to use
> FORGE fills Sections 1–3 after the May 1 first run and at least 3 days of pipeline monitoring (target: file by May 5). When complete, SENTINEL reads Sections 1–3 and makes a single authorization decision in Section 4. No multi-document cross-reference required.

---

## Status Summary

```
Last updated: 2026-04-29 (pre-population complete — Stage 2 Expansion Scope, Risk Assessment, FORGE Certification, Section 2 pre-staged)
Stage 1 first run: NOT YET EXECUTED (May 1 target)
Section 1 metrics filled: 0 of 6 [TO FILL: May 4 — after May 1–3 pipeline runs]
Section 2 blocking issues: 2 known pre-run issues logged
Section 3 browser entities confirmed: 0 of 11 (all pending-browser-lookup)
Authorization status: PRE-POPULATED — fill Section 1 actuals on May 4; SENTINEL review targeted May 4–5
```

---

## Stage 2 Expansion Scope

> Pre-populated from `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md`. All org-IDs require browser session confirmation before Stage 2 activation.

### Stage 2 org-IDs Pre-Staged (Pending Browser Lookup)

**Group A — USYS State Associations (7 entities):**

| # | Entity | State | Config Entry Status | Org-ID |
|---|--------|-------|--------------------|----|
| 1 | Cal North Soccer (CA NorCal) | California | Pre-staged in config template | [TBD-ORG-ID] |
| 2 | Cal South Soccer (CA SoCal) | California | Pre-staged in config template | [TBD-ORG-ID] |
| 3 | Florida Youth Soccer Assoc (FYSA) | Florida | Pre-staged in config template | [TBD-ORG-ID] |
| 4 | Eastern NY Youth Soccer (ENYYSA) | New York | Pre-staged in config template | [TBD-ORG-ID] |
| 5 | NJ Youth Soccer Assoc (NJYSA) | New Jersey | Pre-staged in config template | [TBD-ORG-ID] |
| 6 | Washington Youth Soccer (WSYSA) | Washington | Pre-staged in config template | [TBD-ORG-ID] |
| 7 | Eastern PA Youth Soccer (EPYSA) | Pennsylvania | Pre-staged in config template | [TBD-ORG-ID] |

**Group B — League Entities (4 entities):**

| # | Entity | Notes | Config Entry Status | Org-ID |
|---|--------|-------|--------------------|----|
| 8 | EDP Soccer | Source #1 — GotSport config add | Pre-staged in config template | [TBD-ORG-ID] |
| 9 | USL Academy | Source #2 — GotSport config add | Pre-staged in config template | [TBD-ORG-ID] |
| 10 | Elite 64 | GotSport config add (5-min add) | Pre-staged in config template | [TBD-ORG-ID] |
| 11 | NAL | Platform unconfirmed — may exit this list if non-GotSport | Pre-staged conditionally | [TBD-ORG-ID or EXCLUDED] |

### Entities Confirmed NOT on GotSport (excluded from Stage 2 config)

Per `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` Section 3: SnapSoccer and Affinity Soccer are not GotSport-platform sources. They require separate engineering builds and are NOT part of Stage 2 config expansion.

### Open Item: TYSA Org-ID

TYSA (Texas Youth Soccer Association) is Stage 1 TX, not Stage 2. However, it is the single open item for **Stage 1 completion** and has downstream effects on Stage 2 timing. Status as of 2026-04-29: org-ID `pending-browser-lookup`. FORGE fills TYSA config within 1 hour of org-ID receipt from Presten.

### Sources Pending Stage 2 Activation

- GotSport API (existing parser) — reused for all 11 Stage 2 org-IDs. Zero code changes required.
- No new scraper engineering required for Stage 2. All Stage 2 entities use existing `gotsport_api` pipeline.

---

---

## Section 1: Stage 1 Performance Results

> FORGE fills this after the May 1 first run and at least 3 days of pipeline monitoring. All "Required Threshold" values are pre-populated below — do not leave them blank at fill time.

| Metric | Required Threshold | Actual Result | Pass? |
|--------|--------------------|---------------|-------|
| First-run game count (new games in first 24h) | > 0 (basic pass); ≥ 500 for confidence in org-ID coverage | [TO FILL: May 4] | [TO FILL: May 4] |
| Run completion (no crash) | Last log line = `DAILY PIPELINE COMPLETED SUCCESSFULLY` on May 1 run | [TO FILL: May 4] | [TO FILL: May 4] |
| Active team ID count (STEP 1 log line) | 15,000–40,000 (if < 1,000 or > 100,000: FAIL — filter not working) | [TO FILL: May 4] | [TO FILL: May 4] |
| Source-level error rate | DLQ ≤ 2% of total teams attempted (from SYNC_RUN_SUMMARY); zero TX org-ID 403/404 errors | [TO FILL: May 4] | [TO FILL: May 4] |
| ECNL game ingestion | ≥ 1 confirmed new ECNL game in first 3 days (confirms TGS scraper active) | [TO FILL: May 4] | [TO FILL: May 4] |
| Duplicate rate | 0 duplicate rows in 24h window (query from Stage 2 Pre-Auth Criteria, G2) | [TO FILL: May 4] | [TO FILL: May 4] |

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

> Pre-populated with known pre-run issues as of 2026-04-29. FORGE updates as issues are resolved or new issues arise during May 1–7 monitoring.

| Issue | Date Observed | Resolution | Resolved? |
|-------|---------------|------------|-----------|
| TYSA org-ID pending browser lookup — Stage 1 TX crawl cannot run without it | 2026-04-26 | Presten browser session lookup (`system.gotsport.com` → search "Texas Youth Soccer" → extract `?org_id=XXXX`). FORGE adds to config within 1 hour of receipt. | ☐ No — pending |
| April 28 pipeline session not yet run — GA ASPIRE event_tier fix, ecnl_verified ALTER TABLE, and Step 2b backfill are unconfirmed. ELO gate checks G1–G4 running against pre-fix data. | 2026-04-28 | Presten runs the April 28 session; FORGE executes Step 2b backfill SQL and confirms via Post-Session Schema Confirmation. | ☐ No — pending as of 2026-04-29 |

> If no issues remain by May 4: replace table entries with "No blocking issues observed in Stage 1 monitoring window (May 1–[date]). Pre-run issues from 2026-04-29 resolved."

**Gate:** Any unresolved blocking issue in this table = Stage 2 NOT authorized, regardless of Section 1 metric results.

**Note on April 28 session impact on Stage 2:** The April 28 session issues do NOT block May 1 Stage 1 pipeline launch. May 1 Stage 1 launch is UNAFFECTED by G0 status. The April 28 items affect ELO gate checks (G1–G4, CP1) but not the GotSport org-ID expansion pipeline. Remove these from this blocking issues log once the April 28 session runs and FORGE confirms fixes applied.

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

## Risk Assessment

> Pre-populated 2026-04-29. Does not require post-May-1 data.

### Known Risks of Stage 2 Expansion

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Org-ID crawl scope increase causes GotSport rate-limiting (429 errors) | Medium — adding 7–11 new org-IDs increases team-ID discovery surface | Medium — partial crawl only; no data loss | Backoff implementation (`INTER_REQUEST_DELAY_MS = 600ms`) already in place. FORGE monitors DLQ rate for 48 hrs post-Stage-2 config add. If DLQ > 2%: pause Stage 2 additions, investigate rate-limit pattern. |
| Browser session org-IDs not confirmed before Stage 2 go-live (May 17–20) | Medium — browser session window closes May 17 | Low — Stage 2 simply proceeds with confirmed org-IDs only; unconfirmed entities deferred to Stage 3 | Stage 2 can proceed with any confirmed subset. No minimum number of org-IDs required. |
| New org-ID produces unexpected game volume spike (data quality issue) | Low — all Stage 2 sources use existing `gotsport_api` parser | Medium — large volume spike could distort ELO ratings if not caught | ELO monitors R3 (large mover count) after each Stage 2 org-ID addition. FORGE files per-org-ID game count in first-run report. |
| NAL exits GotSport list (non-GotSport platform) | Medium — platform unconfirmed | Low — NAL exits Stage 2 scope; no engineering work needed | Browser check resolves before May 17. If non-GotSport: NAL moves to non-GotSport engineering queue as Source #5 conditional. |
| TYSA Stage 1 first-run game volume unexpectedly high or low | Medium — no prior TX crawl baseline | Low-Medium — high volume = ELO rating churn; low volume = coverage concern | TYSA first-run output compared against `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` GREEN floor (≥ 5,000 games). FORGE flags anomaly within 24 hrs of first run. |

### Stage 2 Monitoring Commitments

FORGE runs the following during Stage 2 expansion week (post-authorization, ~May 17–20):
1. Per-org-ID game count check within 48 hours of each new org-ID addition
2. DLQ rate check after each config change (GREEN: ≤ 2%)
3. Duplicate rate check (GREEN: 0 duplicates)
4. Notify ELO immediately if any new org-ID produces > 15 large movers (R3 threshold)

### Rollback Path

If Stage 2 expansion causes pipeline instability:
1. Remove new org-IDs from config (revert Stage 2 config edit)
2. Re-run pipeline to confirm Stage 1 org-IDs remain stable
3. File Stage 2 Rollback Report for SENTINEL within 2 hours of rollback decision
4. Do not re-add Stage 2 org-IDs until SENTINEL authorizes re-attempt

Config rollback is a 5-minute operation. No database changes required — GotSport org-ID additions do not write to the database until the pipeline runs. Rollback returns pipeline to Stage 1 scope with zero data loss.

---

## FORGE Certification (Pre-Stage)

> Pre-populated 2026-04-29. Fields FORGE can certify now — before Stage 1 launch.

| Item | Status | Notes |
|------|--------|-------|
| Stage 2 config entries pre-written for all 11 entities | ✅ COMPLETE | See `Infrastructure/Stage 2 Config Pre-Population — Pending Org-IDs.md` |
| All 11 entities reviewed for GotSport platform status | ✅ COMPLETE | NAL conditional; SnapSoccer and Affinity excluded |
| Game volume forecast filed for Stage 1 baseline | ✅ COMPLETE | `Infrastructure/May 1 — Per-Source Game Volume Forecast.md` |
| Week 1 monitoring log template pre-populated | ✅ COMPLETE | `Infrastructure/Pipeline Week 1 Monitoring Log.md` |
| Stage 2 Authorization Monitoring Rubric filed | ✅ COMPLETE | `Infrastructure/Stage 2 Authorization — Week 1 Monitoring Rubric.md` |
| Rollback path defined | ✅ COMPLETE | See Risk Assessment section above |
| SENTINEL notification plan established | ✅ COMPLETE | FORGE notifies SENTINEL via next briefing and direct queue item on: (a) browser session receipt, (b) Stage 1 first-run results, (c) Stage 2 authorization signal |
| **Week 1 actuals confirmed** | ☐ NOT YET | [TO FILL: May 4 — after May 1–3 pipeline runs] |
| **SENTINEL authorization received** | ☐ NOT YET | [TO FILL: May 4–5 — SENTINEL authorization date] |

**FORGE pre-stage certification:** All pre-population-eligible fields above are complete as of 2026-04-29. Document is ready for Sections 1 and 4 fill-in on May 4.

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
