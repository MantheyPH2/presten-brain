---
type: elo-recovery-brief
subject: Boys Option A — Fail Recovery Brief
author: ELO
date_pre_written: 2026-04-28
date_filed: —
status: pre-written — file on April 29 if FAIL verdict
trigger: Boys Option A FAIL on April 29 (hard FAIL, not YELLOW)
tags: [boys, option-a, calibration, fallback, dss, club-rankings, evo-draw]
---

# Boys Option A — Fail Recovery Brief

> **Filing instructions:** This document is pre-written on April 28. ELO files it at `Rankings/Boys Option A — Fail Recovery Brief.md` on April 29 within 30 minutes of a confirmed FAIL verdict from the Boys Option A spot check results. Do NOT file if the verdict is YELLOW (inconclusive) or PASS. If Option A PASSES, mark `task-2026-04-28-boys-option-a-fail-recovery-brief.md` as `status: completed` with a note "Option A PASS — brief not filed, not needed."

---

## 1. Verdict Summary

Boys Option A: **FAIL.** Spot check results received April 29 met the FAIL criteria defined in `Rankings/Boys Option A — Pre-Verdict Evidence Checklist.md`. Specific triggering condition: [ELO fills in actual query result and threshold on April 29 — e.g., "Query 1 divergence in U15B = 118 pts, exceeding the 100-pt hard FAIL threshold" or "Query 2: U14B Boys GA games = 312, below the 500-game minimum"].

Verdict filed: 2026-04-29.

---

## 2. DSS Feature Readiness Tracker — Update Instructions

ELO makes the following updates to `02 - Tiger Tournaments/Projects/Product & Planning/DSS Feature Readiness Tracker — May 2026.md` within 30 minutes of filing this brief:

| Row | Change |
|-----|--------|
| Boys calibration status | Change to **DEFERRED** |
| Boys Option A row | Change to **FAIL** (date: 2026-04-29) |
| DSS Club Rankings — Boys | Change to **NOT INCLUDED (OPTION A FAIL)** |
| Boys re-attempt window | Fill in: **POST-DSS, TENTATIVELY MAY 20+** |

Girls Club Rankings row: **unchanged** — Girls scope is unaffected by Boys Option A result.

---

## 3. Club Rankings — Boys Scope Impact

**Boys Club Rankings is excluded from the DSS demo.**

Girls-Only Fallback Spec activates per `Rankings/Club Rankings — Girls-Only Fallback Spec.md`. That spec is self-executing — no new decisions are required from Presten or SENTINEL to activate it. ELO confirms activation by updating the Club Rankings May Launch Conditions document (filed 2026-04-28) with a note:

> "Boys Club Rankings deferred. Boys Option A FAIL on 2026-04-29. Girls-Only Fallback Spec active per `Rankings/Club Rankings — Girls-Only Fallback Spec.md`. Girls implementation proceeds on May 2 schedule without Boys scope."

Girls launch timeline (`Rankings/Club Rankings — May Launch Conditions.md` Section 5) is unaffected: earliest Girls launch remains May 9 pending May 2–8 implementation and validation.

---

## 4. Calibration Sprint — Scope Revision

The post-DSS calibration sprint plan (`Rankings/Post-DSS Calibration Roadmap — June 2026.md` / `task-2026-04-27-post-dss-calibration-sprint-plan.md`) included a Boys branch. On FAIL, ELO **appends** the following section to the existing sprint plan document — ELO does NOT rewrite the document:

---

**[APPENDED 2026-04-29 — Boys Option A FAIL — Scope Revision]**

Boys Option A returned FAIL on 2026-04-29. Boys calibration work is removed from Week 1 and Week 2 of the post-DSS sprint.

- **Week 1 scope (as revised):** Girls full calibration run only (unchanged from original plan)
- **Week 2 scope (as revised):** Girls calibration validation and monitoring only; Boys branch removed
- **Boys re-attempt criteria:** ≥ 20 additional Boys games ingested post-Stage 1 crawl and May 1 pipeline run. Earliest re-attempt: May 15–20 window.
- **Decision authority:** ELO proposes re-attempt readiness; SENTINEL authorizes session.

---

## 5. Event Strength Phase 1 — Boys Impact

Boys teams **remain included** in Event Strength Phase 1 computation despite Boys Option A FAIL. Event strength applies to all teams in the ranking pool regardless of calibration validation status. The Phase 1 computation uses event-level game data, not Boys-specific calibration values. No change to Phase 1 scope required.

This is documented here to prevent any confusion about whether "Boys calibration FAIL" triggers Boys exclusion from event strength — it does not.

---

## 6. Next Boys Option A Attempt

| Item | Detail |
|------|--------|
| Minimum additional Boys games required | ≥ 20 post-Stage 1 crawl and May 1 pipeline run |
| Earliest re-attempt date | 2026-05-15 (estimate — contingent on game accumulation) |
| Decision authority | ELO proposes readiness; SENTINEL authorizes |
| Query pack | Re-run the same three queries from `Rankings/Boys Option A Spot Check — Presten Execution Package.md` |
| Pass threshold | Unchanged from April 29 run (see original decision doc) |
| FORGE dependency | None — re-attempt is a query-only operation |

ELO files a SENTINEL queue item (Section 7) immediately with re-attempt readiness criteria. Presten does not need to take action until ELO confirms re-attempt is ready.

---

## 7. SENTINEL Notification

**Pre-written queue entry — ELO files this immediately after filing this brief:**

```yaml
---
type: agent-queue
agent: ELO
category: alert
priority: high
date: 2026-04-29
status: pending
answer: ""
---
```

**Body:**

Boys Option A FAIL on April 29. DSS Club Rankings scope reduced to Girls-only (Girls-Only Fallback Spec activated per `Rankings/Club Rankings — Girls-Only Fallback Spec.md`). Calibration sprint Boys branch deferred to May 15+ re-attempt window. No immediate action required from Presten. Girls Club Rankings on track for May 9 go/no-go. Boys re-attempt criteria: ≥ 20 additional Boys games ingested — ELO will flag when met. DSS Feature Readiness Tracker updated: Boys row = FAIL, Boys Club Rankings = NOT INCLUDED.

---

## References

- `Rankings/Boys Option A — Pre-Verdict Evidence Checklist.md` — FAIL thresholds that triggered this brief
- `Rankings/Boys Option A — April 29 Quick-Check Card.md` — source of spot check queries
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — fallback spec activated by FAIL
- `Rankings/Club Rankings — May Launch Conditions.md` — document to annotate with Boys deferred status
- `task-2026-04-27-boys-option-a-post-pass-implementation-timeline.md` — PASS path (not activated)
- `Rankings/Post-DSS Calibration Roadmap — June 2026.md` — sprint plan to append Boys scope revision to
- `task-2026-04-28-boys-option-a-fail-recovery-brief.md` — source task

*ELO — 2026-04-28 (pre-written) | Filed on April 29 only if FAIL verdict confirmed*
