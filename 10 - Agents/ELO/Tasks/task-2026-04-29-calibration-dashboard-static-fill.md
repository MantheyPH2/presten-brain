---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-29
priority: high
due: 2026-04-30 EOD
status: pending
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Pre-May-9 Calibration State Dashboard.md (static sections complete)"
topic: calibration-dashboard-static-fill
tags: [elo, task, calibration, dashboard, dss, may9]
---

# Task: Pre-May-9 Calibration Dashboard — Static Section Pre-Fill

## Context

The Pre-May-9 Calibration State Dashboard template was filed (task-2026-04-28-pre-may9-calibration-state-dashboard). That template specifies what the dashboard should contain. This task is the execution: fill in every section that does NOT require May 1–8 game data.

SENTINEL reviews the dashboard on May 8 before the May 9 DSS gate. If ELO waits until May 8 to build it, there is no buffer for corrections. Pre-filling the static sections now means ELO only needs to add the May 1–8 data observations on May 8 — reducing that session's scope dramatically.

## What to Fill

Open `02 - Tiger Tournaments/Projects/Rankings/Pre-May-9 Calibration State Dashboard.md` (or create it if the template task only produced the spec). Fill these sections:

### Section 1 — Current Calibration Values (All Leagues)

Pull from `Calibration Values — League Hierarchy Reconciliation.md` and `Division Calibration.md`. Record every league's current calibration value as of April 29. This is a static snapshot — it will not change between now and May 9 unless ELO makes a change.

Format:
| League | Cal Value | Status | Last Changed | Notes |
|--------|-----------|--------|--------------|-------|

Mark any pending changes (e.g., MLS NEXT tier split deferred to May 18–20, GA ASPIRE Girls fix pending April 29 session).

### Section 2 — Pending Calibration Changes (May 1–17 Window)

List all known calibration changes that are approved or pending for the May window, with their deployment status:

| Change | Target Date | Authorization Status | Risk if Missed |
|--------|------------|---------------------|----------------|
| Girls GA ASPIRE fix | April 29 session | NOT APPLIED — G0 = NO-GO | May 14 deadline |
| MLS NEXT tier split | May 18–20 | Spec ready, deferred | Post-DSS impact |
| U13/U14 K-factor fix | May 17 | DO NOT DEPLOY before May 17 | Brier pre-check required |

### Section 3 — Known Risks as of April 29

List every open calibration risk and its current severity. Pull from `May 9 DSS Gate — Risk Register.md`. Add any new risks identified since that document was filed.

### Section 4 — Authorization Gate Status

| Gate | Status | Condition to Open |
|------|--------|------------------|
| G0 (Girls GA ASPIRE) | NO-GO | Presten runs GA ASPIRE fix in psql |
| G1–G4 | HELD | After G0 = GO |
| Event Strength Phase 1 | BLOCKED | After G0 = GO + SENTINEL authorization |
| Boys Option A | AWAITING | Presten runs B-OA-1/2/3 queries |
| ECNL Migration Option | PENDING AUTH | Presten decides by April 30 EOD |

### Section 5 — Data-Dependent Sections (Placeholders Only)

Leave these sections with placeholder headers and "(ELO fills May 8)" notation:
- Calibration stability assessment (May 1–8 game data)
- Rating distribution shift from first pipeline run
- Rank bands distribution check

---

## Definition of Done

- Sections 1–4 are complete with no blank cells (use "PENDING" or "OPEN" for unknown values — never blank)
- Section 5 has headers and placeholders so ELO knows exactly what to fill May 8
- Document is self-contained enough that SENTINEL can read it on May 8 without asking ELO for context
- Confirm filing in next ELO briefing

## References

- `task-2026-04-28-pre-may9-calibration-state-dashboard.md` — template spec
- `Rankings/May 9 DSS Gate — Risk Register.md` — risk register
- `Rankings/Calibration Values — League Hierarchy Reconciliation.md` — calibration values
- `Rankings/Division Calibration.md` — division calibration
