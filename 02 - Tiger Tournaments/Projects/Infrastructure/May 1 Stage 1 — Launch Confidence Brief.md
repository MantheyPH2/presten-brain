---
title: May 1 Stage 1 — Launch Confidence Brief
tags: [infrastructure, stage1, may1, pipeline, deployment, tysa, forge, confidence-brief]
created: 2026-04-29
updated: 2026-04-29
author: FORGE
status: filed — MEDIUM confidence; one unresolved item (TYSA org-ID)
task: task-2026-04-28-stage1-may1-launch-confidence-brief
due: 2026-04-30 EOD
---

# May 1 Stage 1 — Launch Confidence Brief

> **Filed by FORGE — 2026-04-29 @ 04:29**
> **Audience:** Presten (2-minute read before May 1 session)
> **Gate document:** `Infrastructure/May 1 Stage 1 — Final Launch Authorization Checklist.md` (SENTINEL signs before launch)

---

## Section 1: Stage 1 Scope

Stage 1 is a single-entity pilot crawl of **Texas Youth Soccer Association (TYSA)** via the existing GotSport API pipeline. It is not an ECNL change, not a new scraper, and not Stage 2 expansion. One org-ID, one state association, same code path as all existing GotSport sources.

---

## Section 2: Readiness Status as of April 29

| Item | Status | Notes |
|------|--------|-------|
| TYSA org-ID confirmed | **PENDING** | This is the only actual launch risk. Presten provides org-ID → FORGE executes config add within 1 hour → Stage 1 is ready. |
| Stage 1 scope = TYSA only | **CONFIRMED** | By design. ECNL changes not part of Stage 1. |
| No ECNL changes in Stage 1 | **CONFIRMED** | G0-Contingent Action Card (`Infrastructure/April 29 — FORGE G0-Contingent Action Card.md`) confirms Stage 1 is G0-independent. |
| GROUP BY archive bug fix | **UNCONFIRMED** | Presten confirms with a grep at April 28/29 session. Does **NOT block** Stage 1. |
| ELO R5 pre-launch baseline | **NOT YET CAPTURED** | ELO must capture R5 stddev baseline before first May 1 run per ELO monitoring plan. ELO to confirm. |
| G0 gate | **NO-GO** | Confirmed NO-GO (ELO filed 2026-04-29 00:09). Stage 1 is **NOT BLOCKED** by G0 result — TYSA crawl is independent. |
| Pipeline code path (FM1/FM2) | **PENDING** | Code review not yet filed by Presten (day 6). Does **NOT block** Stage 1 TYSA crawl directly. |

---

## Section 3: May 1 Launch Decision

**FORGE confidence level: MEDIUM — HIGH once TYSA org-ID is confirmed.**

The one item that, if unresolved by May 1 morning, creates the only actual launch risk is the **TYSA org-ID**. Without it, Stage 1 cannot activate. Every other item on the checklist is either confirmed or non-blocking. The org-ID lookup is a 5-minute browser task at `system.gotsport.com` — search "Texas Youth Soccer" and extract the `?org_id=XXXX` parameter.

At May 1 session open, FORGE's first action (30 seconds) is to verify the TYSA org-ID is in the config and run `node -c crawl-gotsport-api-parallel.js` to confirm syntax before any pipeline execution.

---

## Section 4: What Presten Does on May 1

1. **Before running anything:** Run pre-activation baseline query (from Config Final Review Section 4, Step 1) — capture current game count and timestamp.
2. **Open** `crawl-gotsport-api-parallel.js` — confirm TYSA org-ID is present (should have been added by FORGE after browser session). If not yet added: share org-ID with FORGE, FORGE adds it within 5 minutes.
3. **Syntax check:** `node -c crawl-gotsport-api-parallel.js && echo "Syntax OK"` — confirm no errors.
4. **Run Stage 1 crawl:** `node crawl-gotsport-api-parallel.js` (or trigger via daily pipeline mechanism).
5. **Within 15 minutes:** Run post-activation check query (Config Final Review Section 4, Step 5) — confirm ≥ 5,000 new games ingested (GREEN) or ≥ 500 (YELLOW — investigate before proceeding).

Full step-by-step guide: `Infrastructure/May 1 Deployment — Day-Of Runbook.md`

---

## Section 5: What FORGE Does After First Run

Within 30 minutes of the first TYSA crawl completing, FORGE runs the Stage 1 Actual vs. Expected Reconciliation queries (game count by org-ID, dedup check, no 4xx errors) and confirms volume is within the GREEN band (≥ 5,000 games first run). Findings go into `Infrastructure/Stage 1 TX Crawl — Actual vs. Expected Reconciliation.md`, which is pre-staged and ready.

---

## Known Open Items (Non-Blocking for May 1)

| Open Item | Why Not Blocking | Target Resolution |
|-----------|-----------------|-------------------|
| TYSA org-ID unconfirmed | Pending Presten browser session — executes same-day if needed | Before May 1 run |
| G0 = NO-GO (April 28 session) | Stage 1 is ECNL-independent; TYSA crawl proceeds regardless | Session pending Presten |
| FM1/FM2 code review blank | ECNL classifier accuracy is not a Stage 1 TYSA crawl dependency | Presten fills when available |
| Archive dry-run overdue | Completely separate workflow; does not affect Stage 1 | Presten runs 5-min command |
| 112-team stale backfill | Optional pre-May-3 improvement | Non-blocking |

---

## References

- `Infrastructure/May 1 Stage 1 — Final Launch Authorization Checklist.md` — SENTINEL authorization gate (SENTINEL fills before May 1)
- `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md` — org-ID manifest and activation commands
- `Infrastructure/TYSA Config Add — Execution Checklist.md` — TYSA-specific config add steps
- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — full session guide
- `Infrastructure/Stage 1 TX Crawl — Actual vs. Expected Reconciliation.md` — post-run validation template
- `Infrastructure/May 1 Pipeline Launch — ELO Rankings Monitoring Plan.md` — ELO R5 baseline and monitoring checks

*FORGE — 2026-04-29 @ 04:29*
