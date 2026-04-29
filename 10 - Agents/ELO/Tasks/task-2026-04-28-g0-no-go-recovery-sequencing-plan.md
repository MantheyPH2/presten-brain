---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: pending
priority: urgent
due: 2026-04-28 (complete before 11 PM so plan is ready if G0 = NO-GO fires)
trigger: Produce before 11 PM; activate only if G0 = NO-GO
deliverable: "02 - Tiger Tournaments/Projects/Rankings/April 29 G0 NO-GO Recovery Plan.md"
---

# Task: G0 = NO-GO Recovery Sequencing Plan

## Context

At 11 PM tonight, ELO files either G0 = GO or G0 = NO-GO. If NO-GO, ELO needs to immediately provide SENTINEL and Presten with a clear recovery plan: what work proceeds on April 29, what slips, and the new sequencing through DSS on May 16.

Right now ELO's April 29 NO-GO response is fragmented across the April 29 decision tree document and individual task definitions. At 11 PM, ELO should not be constructing a recovery plan from scratch — it should be filing a pre-written one.

This task produces that plan. It can be written entirely from existing task documents. Activate (file) only if G0 = NO-GO fires tonight.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/April 29 G0 NO-GO Recovery Plan.md`

**Write the document now; file it only if G0 = NO-GO is confirmed tonight or by April 29 open.**

---

### Section 1: What April 29 Can Still Accomplish (G0-Independent Work)

These items proceed regardless of G0 = NO-GO:

| Item | Description | Expected Output |
|------|-------------|-----------------|
| Boys Option A queries | Run boys-specific win-rate and spread queries; assess whether Option A calibration change is justified | `Rankings/Boys Option A Decision Document.md` Section 2 |
| ECNL Option A prep | If Presten selects Option A (decision due April 30), ELO can pre-stage implementation steps | `Rankings/ECNL Migration Option A — Implementation Prep Package.md` |
| Pre-May-9 calibration dashboard | Section 1 (Girls baseline — snapshot from April 27 data) can be pre-filled | `Rankings/Pre-May-9 Calibration State Dashboard.md` |

---

### Section 2: What Slips (G0-Dependent Work)

| Item | Original Target | New Earliest | Slip Impact |
|------|----------------|-------------|-------------|
| GA ASPIRE post-fix verification (G1–G4) | April 29 | Day of actual session (TBD) | Girls calibration baseline unconfirmed |
| ECNL CP1 staleness check | April 29 | Day after session runs | ECNL migration Option A/B decision delayed |
| Girls calibration post-fix narrative | April 29 | Day after session + 24 hrs | ELO cannot confirm girls baseline for DSS |
| Event Strength Phase 1 SENTINEL request | April 30 | 24 hrs after session runs | May 1–9 activation window may close |
| Phase 1 activation (May 1–9) | May 1–9 | May 9 latest (compressed) | DSS demo risk if Phase 1 activates < 7 days before May 16 |
| ecnl_verified backfill (FORGE action) | April 29 | Day of session + 2 hrs | ECNL CP1 permanently blocked until backfill done |

---

### Section 3: Revised Timeline to DSS (May 16)

If session runs April 29 (1-day slip):

| Date | Action |
|------|--------|
| April 29 | Session runs; April 29 becomes effective G0 date |
| April 30 | ELO runs G1–G4 gates against April 29 results; ECNL CP1/CP2; Boys Option A verdict |
| May 1 | Event Strength Phase 1 SENTINEL authorization request filed |
| May 2–3 | SENTINEL authorizes Phase 1 (or defers) |
| May 4–6 | Phase 1 activation (girls only); EDP Source #1 Gates 1–2 |
| May 7–8 | Pre-May-9 dashboard filled; SENTINEL DSS readiness review |
| May 9 | DSS pre-flight; SENTINEL authorization |
| May 16 | DSS demo |

If session runs April 30 (2-day slip):

| Date | Action |
|------|--------|
| April 30 | Session runs; G1–G4 gates |
| May 1–2 | Event Strength Phase 1 SENTINEL request |
| May 3–4 | SENTINEL authorizes |
| May 5–7 | Phase 1 activation; monitoring |
| May 8–9 | Dashboard + pre-flight |
| **RISK:** Phase 1 activates only 9 days before DSS. Tight but within safe window. |

If session not run by May 1 (3+ day slip):

| Item | Impact |
|------|--------|
| Event Strength Phase 1 | DEFERRED to post-DSS. SENTINEL must make call. |
| Girls calibration baseline | Unconfirmed at DSS — ELO notes confidence as "preliminary" |
| ECNL migration | Option A/B choice slips; June 2 at risk if < 3 weeks of prep |

---

### Section 4: ELO's Recommendation if G0 = NO-GO

**If session runs before April 30:** Timeline compressible; all DSS items intact.

**If session runs May 1 or later:** SENTINEL should formally assess whether Event Strength Phase 1 stays on the DSS critical path or is deferred. ELO's recommendation: if session doesn't run by April 30, defer Phase 1 to post-DSS rather than activating 10 days before demo with compressed monitoring window.

---

### Section 5: Communication Checklist When Filing This Document

When ELO files G0 = NO-GO and this recovery plan:

- [ ] G0 = NO-GO gate document filed to `Rankings/April 28 G0 Gate Confirmation.md`
- [ ] SENTINEL queue alert filed to `10 - Agents/SENTINEL/Queue/pending-2026-04-28-g0-no-go-alert.md`
- [ ] This recovery plan filed at deliverable path
- [ ] ELO April 28 late briefing references all three filings
- [ ] Boys Option A query package confirmed ready for April 29

---

## Definition of Done

- Sections 1–5 fully populated from existing task documents
- Section 3 revised timeline has no blank cells
- Document filed at deliverable path **only if G0 = NO-GO fires**
- If G0 = GO fires before 11 PM, this document is not filed (mark task as superseded)

## Why This Matters

The 11 PM deadline is a real escalation point. If G0 = NO-GO, SENTINEL and Presten need an immediate clear view of what it means for DSS. An exploratory recovery analysis at 11 PM is worse than having this pre-written. Pre-writing it also forces ELO to reason through the cascade now, while there is time to identify mitigation options — not reactively at escalation time.

## References

- `task-2026-04-25-april29-post-fix-decision-tree.md` — decision tree for April 29 gate sequence
- `task-2026-04-26-april29-decision-tree-verdict-doc.md` — updated decision framework
- `task-2026-04-27-april29-pre-session-readiness-check.md` — what was ready as of April 27
- `task-2026-04-27-may9-dss-demo-pre-flight-checklist.md` — DSS critical path reference
