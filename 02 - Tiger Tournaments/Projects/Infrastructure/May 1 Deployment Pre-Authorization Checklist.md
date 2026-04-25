---
title: May 1 Deployment Pre-Authorization Checklist
tags: [infrastructure, pipeline, deployment, daily-pipeline, checklist, evo-draw]
created: 2026-04-25
updated: 2026-04-25
author: FORGE
status: ready — complete on morning of May 1, before opening Session Guide
companion: "Infrastructure/May 1 Daily Pipeline Deployment Session Guide.md"
---

# May 1 Deployment Pre-Authorization Checklist

> **Open this document first on May 1 morning — before opening the Session Guide.**
> Complete every checkbox below. If any Section 1–3 item is unchecked, deployment is held. Contact SENTINEL before proceeding.

Estimated time to complete this checklist: **5 minutes.**

---

## Section 1: April 28 DB Changes — Confirmed

These are the pre-conditions the May 1 session assumes are already in place. Verify each against the April 28 session output.

- [ ] **GA ASPIRE tier fix ran** — `event_tier = 'ga_aspire'` applied to all ASPIRE events  
  _(Confirmed: `SELECT COUNT(*) FROM events WHERE event_tier = 'ga_aspire'` returns > 0)_  
  _(Source: April 29 Health Check Section 1A)_

- [ ] **No residual misclassified GA ASPIRE events** — zero events still tagged `ga` with ASPIRE in name  
  _(Confirmed: `SELECT COUNT(*) FROM events WHERE event_tier = 'ga' AND event_name ILIKE '%aspire%'` returns 0)_  
  _(Source: April 29 Health Check Section 1C)_

- [ ] **`ecnl_verified` column exists in `games` table**  
  _(Confirmed: `SELECT column_name FROM information_schema.columns WHERE table_name = 'games' AND column_name = 'ecnl_verified'` returns 1 row)_  
  _(Source: April 29 Health Check Section 2A)_

- [ ] **Full ranking recompute completed after April 28 changes**  
  _(Confirmed: `SELECT MAX(updated_at) FROM rankings WHERE age_group = 'U13G'` shows timestamp after April 28 session start)_  
  _(Source: April 29 Health Check Section 3A)_

---

## Section 2: April 29 Pipeline Health Check — GREEN

The May 1 Session Guide requires the April 29 health check to be GREEN. This is a hard prerequisite — do not skip it.

- [ ] **April 29 pipeline health check was run**  
  _(Reference: `Infrastructure/April 29 Pipeline Health Check — Post-Fix Protocol.md`)_

- [ ] **Health check result is GREEN** — all 8 checklist items passed  
  _(Confirmed in: FORGE April 29 briefing — look for "Pipeline resume decision: GREEN")_

- [ ] **No YELLOW or RED items remain unresolved** from the April 29 check  
  _(If any YELLOW or RED was logged in the April 29 briefing: confirm SENTINEL cleared it before proceeding)_

**If April 29 health check was not run, or returned YELLOW/RED:** Stop. Do not open the Session Guide. Contact SENTINEL with the specific outstanding item.

---

## Section 3: run-daily.sh — In Place

- [ ] **`run-daily.sh` file exists at the project root**  
  _(Check: `ls -lh run-daily.sh` from project root — file should exist with non-zero size)_

- [ ] **`get-active-team-ids.sh` file exists at the project root**  
  _(Check: `ls -lh get-active-team-ids.sh` — file should exist)_

- [ ] **No syntax errors in `run-daily.sh`**  
  _(Run: `bash -n run-daily.sh && echo "Syntax OK"` — expected: `Syntax OK`)_

- [ ] **No active crawl or pipeline runs in progress**  
  _(Run: `ps aux | grep -E "crawl-gotsport|run-pipeline|run-overnight" | grep -v grep` — expected: no output)_

> [!note] If scripts are missing
> The May 1 Session Guide Section 3 contains the complete script contents. Write them now, then return to this checklist to confirm the syntax check.

---

## Section 4: Stage 1 TX Test Crawl — Status Confirmed

Stage 1 status affects Stage 3 authorization, but does **not** block May 1 deployment. Check the current status and circle one:

- [ ] **Stage 1 status confirmed:** `PASS` / `PENDING` / `FAIL`

| Status | What it means for May 1 |
|--------|------------------------|
| **PASS** | Deployment proceeds. Stage 3 authorization window opens after May 2 morning-after GREEN check. |
| **PENDING** | Deployment proceeds. Stage 3 authorization is deferred — do not run Stage 3 until Stage 1 passes. |
| **FAIL** | Flag to FORGE before proceeding. Deployment may still proceed; Stage 3 does not. |

_(Reference: `10 - Agents/FORGE/Tasks/task-2026-04-25-stage1-backfill-results-report.md` for Stage 1 results when available)_

---

## Section 5: SENTINEL Authorization

To be completed by SENTINEL (or Presten acting as SENTINEL) after Sections 1–3 are fully checked.

All Section 1–3 boxes are checked. Deployment is authorized.

**SENTINEL authorization:** ___________________________  
**Date/time:** ___________________________

---

> [!warning] If any Section 1–3 box is unchecked
> Deployment is held. Identify the specific unchecked item, note it here, and contact SENTINEL:
>
> Outstanding item: ___________________________
>
> Do not open the May 1 Session Guide until the item is resolved and SENTINEL provides written authorization.

---

## After Completing This Checklist

Open `Infrastructure/May 1 Daily Pipeline Deployment Session Guide.md` and begin at Section 2 (Pre-Session Checklist). The Session Guide's checklist overlaps with this one — items you've already confirmed here can be checked off immediately.

---

## References

- `Infrastructure/May 1 Daily Pipeline Deployment Session Guide.md` — the deployment session itself
- `Infrastructure/April 29 Pipeline Health Check — Post-Fix Protocol.md` — Section 2 source
- `Product & Planning/April 28 Escalation Reference Card.md` — April 28 session execution (Section 1 source)
- `Infrastructure/Daily Pipeline Morning-After Validation Runbook.md` — what to run on May 2 after first overnight run
- `Infrastructure/Stale Team Backfill Runbook — April 2026.md` — Stage 3 sequencing constraint

*FORGE — 2026-04-25*
