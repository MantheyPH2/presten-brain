---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
due: 2026-05-08
priority: high
status: pending
tags: [elo, task, may9, dss, calibration, dashboard, sentinel-gate]
---

# Task: Pre-May-9 Calibration State Dashboard

## What to Build

A single document that summarizes the current MET / NOT MET / DEFERRED status of every active calibration item, filed on May 8 — one day before the May 9 DSS go/no-go. SENTINEL reads this alongside FORGE's Infrastructure Readiness Assessment to issue the unified go/no-go verdict.

## Why This Exists

As of April 28, ELO's calibration work is spread across 6+ spec documents, 3+ task files, and multiple briefing entries. SENTINEL cannot reconstruct the full calibration state for the May 9 gate without reading all of them. This dashboard is a single, authoritative, point-in-time snapshot. If it contradicts any individual spec, the dashboard is wrong — ELO is responsible for keeping it accurate.

## Due

File by end of day 2026-05-08. SENTINEL reviews on May 9.

## Required Format

### Header

```yaml
---
type: elo-calibration-dashboard
date: 2026-05-08
filed_for: May 9 DSS Go/No-Go Gate
overall_verdict: READY | CONDITIONAL | NOT READY
---
```

### Calibration Status Table

Six rows. Each row has: item, status (MET / NOT MET / DEFERRED / IN-PROGRESS), brief evidence statement, source document.

| Item | Status | Evidence | Source |
|------|--------|----------|--------|
| Girls full calibration (production) | [STATUS] | [1 sentence] | [doc] |
| Boys Option A verdict | [STATUS] | [1 sentence] | [doc] |
| Event Strength Phase 1 applied | [STATUS] | [1 sentence] | [doc] |
| Tier 2 undefined leagues calibration | [STATUS] | [1 sentence] | [doc] |
| ECNL migration rating continuity | [STATUS] | [1 sentence] | [doc] |
| Team merges audit (high-priority) | [STATUS] | [% reviewed, outstanding count] | [doc] |

**Status definitions:**
- MET: condition has been verified in production
- NOT MET: condition has not been met; flag if this blocks DSS
- DEFERRED: explicitly deferred with SENTINEL authorization; does not block DSS
- IN-PROGRESS: work is underway, expected completion before May 9

### DSS Blocking Assessment

For each NOT MET item: state whether it blocks the DSS demo. Format:

> [Item]: NOT MET. DSS BLOCK: YES / NO. Reason: [1 sentence].

Items that block the demo must resolve before May 9 or produce a NO-GO verdict from ELO.

### Brier Score Status

Single paragraph. Current Girls Brier score: `[value]`. Target: `< 0.24` (DSS authorized threshold). Status: ABOVE THRESHOLD / BELOW THRESHOLD / NOT YET RUN. If above threshold: state whether this blocks DSS (per DSS SENTINEL Authorization Document).

### USARank Comparison Readiness

Has ELO completed the post-April-28 USARank comparison? YES / NO. If YES: state confidence level in ranking accuracy vs. USARank. If NO: state expected completion date.

### ELO Overall Verdict

One of:
- **READY** — All items MET or DEFERRED (with authorization). No DSS-blocking NOT MET items. Rankings are accurate, calibration is stable, demo materials are authorized.
- **CONDITIONAL** — One or more IN-PROGRESS items expected to complete by May 9. State specific conditions.
- **NOT READY** — One or more DSS-blocking NOT MET items. Immediate SENTINEL notification filed.

### SENTINEL Authorization Request

> ELO certifies calibration readiness for the May 9 DSS go/no-go review. [Summary of key statuses.] Overall verdict: [READY / CONDITIONAL / NOT READY]. ELO requests SENTINEL include this assessment in the May 9 gate decision.

## File When Done

`02 - Tiger Tournaments/Projects/Rankings/Pre-May-9 Calibration State Dashboard.md`

Mark this task `status: completed` in frontmatter after filing.

## Dependencies

- Girls Club Rankings first run (May 2) and validation (May 2–8)
- Boys Option A verdict (April 29)
- Event Strength Phase 1 authorization and execution (expected early May)
- Tier 2 calibration SQL execution (task-2026-04-27-tier2-calibration-implementation-sql.md)
- ECNL migration implementation (FORGE, post-April-30 option decision)
- Post-April-28 USARank comparison execution
- Brier score re-run (post-GA ASPIRE fix)
