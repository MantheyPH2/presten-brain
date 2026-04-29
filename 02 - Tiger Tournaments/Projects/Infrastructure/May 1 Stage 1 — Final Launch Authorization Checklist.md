---
title: "May 1 Stage 1 — Final Launch Authorization Checklist"
tags: [infrastructure, pipeline, stage1, deployment, authorization, sentinel, forge]
created: 2026-04-28
updated: 2026-04-28
author: FORGE
status: filed — SENTINEL authorization pending
task: task-2026-04-28-may1-stage1-final-launch-checklist
due: 2026-04-30
---

# May 1 Stage 1 — Final Launch Authorization Checklist

> [!info] Purpose
> Definitive pre-launch gate document for May 1 Stage 1 TYSA pipeline run. SENTINEL reviews all sections and issues authorization in Section 6 before May 1 launch. No launch proceeds without Section 6 signed.

```
Launch Window: May 1, 2026
Launch Type: Stage 1 — TYSA GotSport Crawl (first production run)
Authorization Status: [AUTHORIZED / HOLD / BLOCKED — SENTINEL fills before launch]
Filed by FORGE: 2026-04-28
Reviewed by SENTINEL: [date/time]
```

---

## Section 1: Required Configuration Complete

| Config Item | Status | Notes |
|-------------|--------|-------|
| TYSA org-ID confirmed and added | [ ] YES / [x] PENDING | Browser session not yet executed; Section 5 of Config Final Review blocked on this. FORGE fills within 1 hr of receipt. Source: `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md` |
| EDP Source #1 Day 0 config | [ ] YES / [x] PENDING — depends on browser session | EDP org-ID pending browser confirmation. Stage 1 does NOT require EDP — EDP is Stage 2. |
| USL Academy Source #2 Day 0 config | [ ] YES / [x] PENDING — depends on browser session | USL Academy org-ID pending browser confirmation. Stage 1 does NOT require USL Academy — USL Academy is Stage 2. |
| Stage 1 scope limited to TYSA only | [x] CONFIRMED | Stage 1 is a single-entity pilot by design. Source: Config Final Review, Section 1. |
| No ECNL migration changes in Stage 1 | [x] CONFIRMED | Stage 1 is ECNL-independent. TYSA crawl does not touch ECNL schemas. Source: Config Final Review, Section 3; G0-Contingent Action Card. |

**Section 1 Note:** TYSA PENDING status does not block Section 6 authorization. If TYSA org-ID is received before April 30, FORGE updates this row and Config Final Review Section 5 within 1 hour. If not received by April 30, Stage 1 still launches on May 1 — FORGE will execute the config add same-day upon receipt and proceed with run.

---

## Section 2: Pipeline Infrastructure Ready

| Infrastructure Item | Status | Source | Notes |
|--------------------|--------|--------|-------|
| GROUP BY archive fix deployed | [ ] YES / [ ] NO / [x] UNCONFIRMED | April 28 session (execution log blank) | Does NOT block Stage 1. Presten should run `grep -A 20 "GROUP BY" archive-inactive-teams.js` at April 29 session open to confirm. |
| FM1 activation path confirmed | [ ] YES / [x] PENDING G0 | Pipeline Code Review Results | FM1/FM2 audit docs open; pending G0 = GO trigger. Does NOT block Stage 1 — TYSA crawl is independent. |
| FM2 activation path confirmed | [ ] YES / [x] PENDING G0 | Pipeline Code Review Results | Same as FM1. |
| Stale team backfill status | [ ] COMPLETE / [x] INCOMPLETE | 112-team backfill not run | Pre-May-1 preferred. Does NOT block Stage 1. |
| Daily pipeline deployment validated | [x] YES | `Infrastructure/Pipeline Deployment Validation Spec.md` | Deployment spec filed; config structure confirmed. |

---

## Section 3: G0 Status and May 1 Independence Confirmation

| Gate | Status at Filing | May 1 Impact |
|------|-----------------|--------------|
| April 28 G0 gate | [ ] GO / [ ] NO-GO / [x] OPEN (as of 17:25 April 28) | Stage 1 is **NOT BLOCKED** by G0 result. FORGE G0-Contingent Action Card filed; Branch A and Branch B both preserve Stage 1 launch authorization. |
| ecnl_verified backfill | [ ] COMPLETE / [x] PENDING | Does NOT affect Stage 1 TYSA crawl. Backfill is ECNL-only; TYSA is not an ECNL source. |
| Girls calibration baseline | [ ] CONFIRMED / [x] PENDING | Does NOT affect Stage 1. Girls calibration work is separate from TYSA ingestion. |

**FORGE declaration:** Stage 1 TYSA crawl is independent of G0, ecnl_verified backfill, and girls calibration gates. May 1 launch proceeds on TYSA config status only. Source: `Infrastructure/April 29 — FORGE G0-Contingent Action Card.md`.

---

## Section 4: ELO Pre-Launch Baseline Captured

| ELO Item | Status | Notes |
|----------|--------|-------|
| R5 pre-launch stddev baseline | [x] NOT YET CAPTURED | Required before first run per ELO monitoring plan. ELO must capture this before May 1 launch. Source: `Infrastructure/May 1 Pipeline Launch — ELO Rankings Monitoring Plan.md` |
| ELO monitoring plan active | [x] YES — filed | `Infrastructure/May 1 Pipeline Launch — ELO Rankings Monitoring Plan.md` — R1–R5 checks defined for Week 1 monitoring. |

**ELO action required before May 1:** Capture R5 rating variance stddev baseline before first Stage 1 run executes.

---

## Section 5: Known Open Items at Launch (Non-Blocking)

All items below are open but do NOT block Stage 1 launch. FORGE confirms this.

| Open Item | Why Not Blocking | Due Date |
|-----------|-----------------|----------|
| TYSA org-ID browser confirmation | Stage 1 can proceed same-day when received; if not by April 30, FORGE executes config add May 1 morning | ASAP / by May 1 launch |
| Archive GROUP BY fix deployment confirmation | Archive dry-run is independent of TYSA crawl pipeline | ASAP — run grep check at April 29 session open |
| Archive dry-run (`--dry-run` output) | Archive fix is independent of crawl pipeline | ASAP post-session |
| FM1/FM2 audit document updates | ECNL verification independent of Stage 1 TYSA crawl | Post G0 = GO |
| 112-team stale backfill | Optional coverage improvement; does not affect first Stage 1 run | Pre-May-3 preferred |
| ecnl_verified backfill completion | ECNL-specific; TYSA source is not ECNL | Post G0 = GO (same session) |
| TYSA Config Final Review Section 5 certification | FORGE signs within 1 hr of org-ID receipt; Stage 1 can run with Section 5 unsigned if org-ID received May 1 morning | Before or same-day May 1 |

---

## Section 6: SENTINEL Authorization Block

```
SENTINEL LAUNCH AUTHORIZATION — May 1 Stage 1

[ ] Section 1 configurations acceptable for launch
    (TYSA PENDING is acceptable — FORGE executes config add upon receipt,
    including same-day May 1 if necessary; EDP/USL Academy are Stage 2, not required)

[ ] Section 2 infrastructure status acceptable for launch
    (GROUP BY UNCONFIRMED, FM1/FM2 PENDING G0, stale backfill INCOMPLETE
    are all confirmed non-blocking for Stage 1 by FORGE)

[ ] Section 3 G0 independence confirmed
    (Stage 1 is NOT BLOCKED regardless of G0 outcome)

[ ] Section 4 ELO R5 baseline captured before launch
    (ELO must confirm this is done before Section 6 is signed)

[ ] Section 5 open items reviewed and accepted as non-blocking

SENTINEL VERDICT:
[ ] AUTHORIZED — Stage 1 may launch May 1
[ ] HOLD — reason: ___________

Signed: SENTINEL
Date/Time: ___________
```

---

## References

- `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md` — config detail and Section 5 certification
- `Infrastructure/May 1 Pipeline Launch — ELO Rankings Monitoring Plan.md` — R1–R5 monitoring framework, R5 baseline requirement
- `Infrastructure/May 1 Deployment — Day-Of Runbook.md` — execution steps for May 1 session
- `Infrastructure/Pipeline Deployment Validation Spec — April 2026.md` — validation query patterns
- `Infrastructure/April 29 — FORGE G0-Contingent Action Card.md` — G0 independence confirmation
- `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md` — GREEN ≥5,000 games, YELLOW 500–4,999, RED <500

*FORGE — 2026-04-28*
