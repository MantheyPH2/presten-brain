---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: high
due: 2026-05-07
deliverable: "02 - Tiger Tournaments/Projects/Product & Planning/DSS Demo — SENTINEL Authorization Document.md"
---

# Task: DSS Demo — SENTINEL Authorization Document

## Context

ELO filed the May 9 DSS Demo Pre-Flight Checklist (2026-04-27): 8 checks, GREEN/YELLOW/RED thresholds per check, copy-paste certification statements. SENTINEL will run this checklist on May 8.

What does not exist: the authorization artifact. The checklist verifies individual conditions. But when SENTINEL finishes the checklist and turns to Presten to say "demo is cleared," there is no document that captures:
- The specific date and state of the system that was certified
- Which checks passed and which (if any) had caveats
- SENTINEL's formal authorization statement
- What conditions, if any, were accepted despite YELLOW status

Without this artifact, the DSS demo authorization exists only as briefing text. If Presten or a stakeholder asks "when and by whom was this demo authorized?", the answer requires reconstructing from briefing history. That is not acceptable for an external-facing demo.

This document is the authorization instrument SENTINEL signs on May 8. ELO writes the template now; SENTINEL fills it in on May 8 after running the pre-flight checklist.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Product & Planning/DSS Demo — SENTINEL Authorization Document.md`

This is a fill-in template. ELO writes all sections and thresholds. SENTINEL fills in the verdict fields on May 8.

---

### Header (SENTINEL fills)

```
DSS Demo Authorization Document
Authorization Date: ___________
Authorization Time: ___________
Authorizing Agent: SENTINEL
Demo Date: 2026-05-09
Demo Audience: DSS
System Snapshot Reference: [path to pre-demo ratings snapshot]
```

---

### Section 1: Pre-Flight Checklist Summary

ELO writes the 8-check summary structure. SENTINEL fills in verdict fields.

| # | Check | Threshold (GREEN) | Result | Status |
|---|-------|-------------------|--------|--------|
| 1 | GA ASPIRE fix confirmed in production | event_tier = 'ga_aspire' rows > 0; 0 misclassified events | | |
| 2 | Rating distribution anomalies | No age group avg rating outside ±2 SD of historical baseline | | |
| 3 | ECNL teams present | ≥ 95% of known ECNL teams have current-season ratings | | |
| 4 | Girls Club Rankings populated | Girls U13–U17 club aggregates populated; ≥ 3 teams per club minimum | | |
| 5 | Brier score in acceptable range | Girls Brier score ≤ 0.22; Boys Brier (if included) ≤ 0.25 | | |
| 6 | USARank comparison current | ELO's USARank delta < 15 positions for top-100 teams (most recent comparison) | | |
| 7 | Demo scenario spot-checks | All reference teams within expected rank bands (per May 9 Pre-Flight Checklist) | | |
| 8 | Boys rankings present (conditional) | Girls-only: N/A. If Boys included: ≥ 90% of Boys U13–U17 teams have ratings | | |

---

### Section 2: Red-Line Assessment

ELO writes the 4 red-line conditions that trigger NO-GO regardless of overall checklist state. SENTINEL marks each.

| Red-Line Condition | Status |
|--------------------|--------|
| GA ASPIRE fix absent from production (Check 1 RED) | |
| ECNL teams missing from ratings (Check 3 RED — < 85% present) | |
| Any reference demo team rated > 15 positions from expected rank band | |
| Girls Club Rankings empty or fewer than 10 clubs populated | |

**Red-line verdict:** If any red-line is marked FAIL → Authorization = NO-GO. Escalate to Presten immediately. Demo cannot proceed without explicit Presten override.

---

### Section 3: SENTINEL Authorization Decision

```
Pre-Flight Checks:
  GREEN: [N/8]
  YELLOW (accepted with caveat): [N/8]
  RED (unresolved): [N/8]

Red-Line Conditions:
  All PASS: YES / NO
  Any FAIL: [list if yes]

YELLOW Caveats Accepted (if any):
  Check [N]: [description of condition and why SENTINEL accepts at YELLOW]

Authorization Decision: AUTHORIZED / NOT AUTHORIZED / CONDITIONAL

If AUTHORIZED:
  "I, SENTINEL, certify that the Evo Draw ranking system meets the DSS demo authorization criteria as of [date/time]. The system state documented in this authorization is the reference state for the May 9 DSS demo."

If CONDITIONAL:
  "SENTINEL authorizes the May 9 DSS demo subject to the following conditions: [list]. These conditions must be resolved before demo start or Presten must acknowledge risk."

If NOT AUTHORIZED:
  "SENTINEL has identified the following unresolved issues that prevent DSS demo authorization: [list]. Demo is on hold pending Presten decision."

SENTINEL signature: SENTINEL — [date] @ [time]
```

---

### Section 4: System State at Authorization

ELO writes the fields; SENTINEL fills them in on May 8.

```
Girls ratings version: [date of most recent recompute]
Boys ratings version: [date of most recent recompute, or "excluded from demo"]
Event Strength Phase 1 active: YES / NO
Club Rankings: Girls [N/5 age groups populated] | Boys [N/5 or "excluded"]
Last pipeline run: [date]
Games in DB: [total count from verification query]
Teams with current-season ratings: [N]
```

---

### Section 5: Demo Risk Register

ELO writes known risks that do not trigger NO-GO but Presten should be aware of. These are the "known knowns" that SENTINEL has accepted as demo risks.

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Boys rankings excluded (Path A) — audience may ask | Medium | Low | Presten aware; "Boys coming in Phase 2" is the answer |
| USARank delta may not show top-100 comparison if recent games not yet processed | Low | Medium | Run final USARank check morning of May 9 |
| Event Strength Phase 1 not yet active (if May 5 authorization delayed) | Low | Medium | Demo script does not rely on Phase 1 being active; mention as "in development" |
| [ELO adds any additional known risks] | | | |

---

## Definition of Done

- Document written at `Product & Planning/DSS Demo — SENTINEL Authorization Document.md`
- All 5 sections present
- Section 1 table has all 8 checks with GREEN thresholds filled in (SENTINEL fills Results and Status on May 8)
- Section 2 red-line conditions are explicit and tied to Section 1 check numbers
- Section 3 authorization text is written — not placeholder prose. SENTINEL selects AUTHORIZED / NOT AUTHORIZED / CONDITIONAL and fills the date.
- Section 5 risk register has at least 3 known risks documented
- Summary in briefing: "DSS Authorization Document template filed at `Product & Planning/DSS Demo — SENTINEL Authorization Document.md`. Ready for SENTINEL fill-in on May 8. Red-line count: 4. Authorization states: AUTHORIZED / CONDITIONAL / NOT AUTHORIZED."

## Why This Matters

The DSS demo is Evo Draw's first external presentation. SENTINEL needs to be able to answer one question unambiguously: "Is the system ready?" The Pre-Flight Checklist determines whether it's ready. This document records that determination in a format that is auditable, sharable with Presten, and clear about what exactly was certified and when. A briefing entry saying "everything looks good" is not the same as a signed authorization document with specific metrics. If the demo goes well and a stakeholder asks "how do you ensure ranking quality?", Presten can point to this document.

## References

- `Rankings/May 9 DSS Demo — Pre-Flight Checklist.md` — the checklist this document summarizes
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — feature status inputs
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — Girls-only scope (Path A)
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — calibration state reference
