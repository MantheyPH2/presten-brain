---
title: DSS Demo — SENTINEL Authorization Document
type: authorization-document
author: ELO (template) | SENTINEL (fill-in on May 8)
date-template-filed: 2026-04-27
date-authorized: [SENTINEL fills on May 8]
status: template-ready
tags: [dss, demo, authorization, sentinel, may-2026, evo-draw]
related: "[[May 9 DSS Demo — Pre-Flight Checklist]]", "[[DSS Feature Readiness Tracker — May 2026]]", "[[Club Rankings — Girls-Only Fallback Spec]]", "[[Post-Fix Calibration Stability Monitoring Spec]]"
---

# DSS Demo — SENTINEL Authorization Document

> **Instructions for SENTINEL:** Complete this document on May 8, 2026 after running the Pre-Flight Checklist (`Rankings/May 9 DSS Demo — Pre-Flight Checklist.md`). Fill in all `___________` fields. Select the authorization decision in Section 3. Sign with the SENTINEL signature line. This document is the formal authorization artifact for the May 9 DSS demo — it replaces any informal "looks good" briefing language.

---

## Header

```
DSS Demo Authorization Document
Authorization Date:       ___________
Authorization Time:       ___________
Authorizing Agent:        SENTINEL
Demo Date:                2026-05-09
Demo Audience:            DSS (Digital Soccer Solutions)
System Snapshot Reference: [path to pre-demo ratings snapshot — Presten saves before May 9]
```

---

## Section 1: Pre-Flight Checklist Summary

> SENTINEL fills **Result** and **Status** columns on May 8 after running each check.

| # | Check | GREEN Threshold | Result (SENTINEL fills) | Status |
|---|-------|----------------|------------------------|--------|
| 1 | GA ASPIRE fix confirmed in production | `event_tier = 'ga_aspire'` rows > 0 AND 0 events still misclassified as `event_tier = 'ga'` for ASPIRE events | | |
| 2 | Rating distribution anomalies | No age group avg rating outside ±2 SD of April 29 post-fix baseline | | |
| 3 | ECNL teams present | ≥ 95% of known ECNL teams have current-season ratings (current season = 2025–2026) | | |
| 4 | Girls Club Rankings populated | Girls U13G–U17G club aggregates populated; minimum 3 teams per club for any club shown in demo | | |
| 5 | Brier score in acceptable range | Girls Brier score ≤ 0.22; Boys Brier (if included) ≤ 0.25 | | |
| 6 | USARank comparison current | ELO's USARank delta < 15 ranking positions for top-100 teams (most recent comparison run May 1–8) | | |
| 7 | Demo scenario spot-checks | All reference teams within expected rank bands per `Rankings/May 9 DSS Demo — Pre-Flight Checklist.md` spot-check list | | |
| 8 | Boys rankings present (conditional) | Girls-only path: mark N/A. If Boys included: ≥ 90% of Boys U13–U17 teams have current-season ratings | | |

**Check status key:**
- `GREEN` — threshold met
- `YELLOW` — below threshold but not red-line; SENTINEL may accept with caveat (document in Section 3)
- `RED` — threshold not met; evaluate against red-lines in Section 2

---

## Section 2: Red-Line Assessment

> These 4 conditions trigger NO-GO regardless of overall checklist state. SENTINEL marks each.

| # | Red-Line Condition | Tied to Check | Status (PASS / FAIL) |
|---|-------------------|--------------|--------------------|
| R1 | GA ASPIRE fix absent from production (Check 1 RED — any events still misclassified as `ga`) | Check 1 | |
| R2 | ECNL teams missing from ratings (Check 3 RED — < 85% of known ECNL teams present) | Check 3 | |
| R3 | Any reference demo team rated > 15 ranking positions outside expected rank band | Check 7 | |
| R4 | Girls Club Rankings empty or fewer than 10 clubs populated with ≥ 3 teams | Check 4 | |

**Red-line verdict:**

```
All 4 red-lines PASS: YES / NO
Any red-line FAIL: [list which, or "none"]

If any red-line = FAIL → Authorization = NOT AUTHORIZED
Escalate to Presten immediately. Demo cannot proceed without explicit Presten override.
```

---

## Section 3: SENTINEL Authorization Decision

> SENTINEL fills this section on May 8 after completing Sections 1 and 2.

```
Pre-Flight Checks:
  GREEN:                           [N/8]
  YELLOW (accepted with caveat):   [N/8]
  RED (unresolved):                [N/8]

Red-Line Conditions:
  All 4 PASS:   YES / NO
  Any FAIL:     [list, or "none"]

YELLOW Caveats Accepted (complete only if any check is YELLOW):
  Check [N]: [description of condition and why SENTINEL accepts at YELLOW]
  Check [N]: [description]

Authorization Decision:   AUTHORIZED / NOT AUTHORIZED / CONDITIONAL

─────────────────────────────────────────────────────────────

If AUTHORIZED:
  "I, SENTINEL, certify that the Evo Draw ranking system meets the DSS demo 
   authorization criteria as of [date/time]. The system state documented in 
   this authorization is the reference state for the May 9 DSS demo."

If CONDITIONAL:
  "SENTINEL authorizes the May 9 DSS demo subject to the following conditions:
   [list conditions]. These conditions must be resolved before demo start or 
   Presten must acknowledge risk in writing."

If NOT AUTHORIZED:
  "SENTINEL has identified the following unresolved issues that prevent DSS demo 
   authorization: [list]. Demo is on hold pending Presten decision."

─────────────────────────────────────────────────────────────

SENTINEL signature:   SENTINEL — [date] @ [time]
```

---

## Section 4: System State at Authorization

> ELO writes the fields; SENTINEL fills values on May 8 from verification queries.

```
Girls ratings version (date of most recent recompute):  ___________
Boys ratings version (date of most recent recompute,
  or "excluded from demo"):                             ___________

Event Strength Phase 1 active:   YES / NO
  (If YES: authorization date: ___________)
  (If NO: note "Phase 1 not active; not referenced in demo script")

Club Rankings:
  Girls: [N/5 age groups fully populated]
  Boys:  [N/5 age groups, or "excluded from demo (Path B)"]

Last pipeline run date:                  ___________
Total games in DB (from V1 query):       ___________
Teams with current-season ratings:       ___________
Girls ECNL teams with ratings (% of known set): ___________ %
```

---

## Section 5: Demo Risk Register

> Known risks that do not trigger NO-GO but Presten should be aware of entering the May 9 demo. SENTINEL reviews these and confirms awareness before demo start.

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Boys rankings excluded (Path B — Option A failed) — audience may ask | Medium | Low | Presten uses "Boys coming in Phase 2" language (see `Rankings/DSS Demo Talking Points — Rankings Features.md` Section 6) |
| USARank delta may not reflect games processed after May 7 if pipeline ran late | Low | Medium | Run final USARank spot-check morning of May 9; if delta exceeds threshold, Presten acknowledges caveat in demo |
| Event Strength Phase 1 not yet active (if May 5 calibration stability delayed) | Low | Medium | Demo script does not depend on Phase 1 being live; reference as "in final validation" rather than "active" |
| Calibration instability between May 5 and May 9 stability check | Low | High | Post-Fix Calibration Stability Monitoring Spec defines Level 3 escalation protocol; SENTINEL flags to Presten by May 8 if detected |
| Girls Club Rankings have < 10 clubs with ≥ 3 teams (R4 red-line risk) | Low | High | Pre-flight Check 4 catches this; if < 10 clubs, triggers red-line NO-GO |
| [SENTINEL adds any additional risks identified during May 8 pre-flight] | | | |

**Presten acknowledgment of risk register:** Presten reviews this section before demo start on May 9. Any HIGH-impact risks must be verbally acknowledged.

---

*Template filed 2026-04-27 by ELO. SENTINEL completes on May 8 after Pre-Flight Checklist run. This document supersedes any informal authorization language in briefings.*
