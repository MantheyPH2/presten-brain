---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-29
priority: high
due: 2026-05-01 (pre-launch, before pipeline runs)
status: pending
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/May 1 — Official Source Baseline.md"
topic: may1-source-baseline-document
tags: [forge, task, pipeline, baseline, stage2]
---

# Task: May 1 Official Source Baseline Document

## Context

The Stage 2 Expansion Authorization Trigger Document (Section 1) requires "Stage 1 confirmed successful with actual game volumes." The Week 1 Monitoring Log captures running totals. But there is no single document that (a) states the forecast numbers, (b) captures the first-run actuals in one place, and (c) serves as the official Stage 2 authorization evidence.

This task pre-builds that document so FORGE can fill in actuals within 2 hours of the May 1 pipeline run completing — not after spending time designing the document format under post-run pressure.

## What to Build

Produce: `02 - Tiger Tournaments/Projects/Infrastructure/May 1 — Official Source Baseline.md`

A baseline document with forecast columns pre-filled and actuals columns blank, structured as the official evidentiary record for Stage 2 authorization.

---

## Required Sections

### Section 1 — Header

```yaml
---
type: pipeline-baseline
date_established: 2026-05-01
pipeline_run: first-production-run
stage: stage-1
stage2_authorization_evidence: true
status: pre-filled (actuals pending pipeline run)
---
```

### Section 2 — Per-Source Forecast vs. Actual Table

Fill in forecast values from `Infrastructure/May 1 — FORGE Launch Confidence Brief.md` and the per-source game volume forecast document. Leave actuals blank.

| Source | Org IDs in Config | Forecast Games (Week 1) | Actual Games (May 1 run) | Delta | Status |
|--------|------------------|------------------------|--------------------------|-------|--------|
| gotsport_api | [fill] | [fill from forecast] | [FORGE fills post-run] | — | — |
| tgs_athleteone | [fill] | [fill from forecast] | [FORGE fills post-run] | — | — |
| tgs | [fill] | [fill from forecast] | [FORGE fills post-run] | — | — |
| tgs_rl | [fill] | [fill from forecast] | [FORGE fills post-run] | — | — |
| TYSA | OPEN (pending Presten) | — | — | — | OPEN |

**Overall Stage 1 forecast total:** [fill] games

**Stage 2 authorization threshold:** ≥[fill]% of forecast achieved AND no critical source failures

### Section 3 — Source Health Flags

Pre-fill known risks:
- FM1/FM2 behavior: [state current status from FM1/FM2 audit package]
- Archive step: NOT running May 1 (pending dry-run authorization; not a launch blocker)
- TYSA: NOT in Stage 1 config (pending org-ID receipt)

### Section 4 — Post-Run Completion Protocol

FORGE fills this within 2 hours of May 1 pipeline completion:

1. Run per-source game count query (from Week 1 Monitoring Log Section 1)
2. Fill "Actual Games" column for each source
3. Calculate delta (actual vs. forecast %)
4. Set status: PASS / UNDER-THRESHOLD / FAILED for each source
5. Set overall Stage 1 status: CONFIRMED / NEEDS-REVIEW
6. Notify SENTINEL (update Stage 2 trigger document Section 1)

### Section 5 — Stage 2 Authorization Decision

SENTINEL fills this after reviewing Section 4:

```
Stage 1 actual result: [ ] CONFIRMED  [ ] NEEDS-REVIEW  [ ] FAILED
Stage 2 expansion: [ ] AUTHORIZED  [ ] HOLD  [ ] ESCALATE TO PRESTEN

Conditions for Stage 2: 
_______________________________

SENTINEL decision date: ___________
```

---

## Pre-Fill Instructions

1. Pull forecast numbers from `task-2026-04-27-may1-per-source-game-volume-forecast.md` and the Launch Confidence Brief
2. Fill all known-static values (org IDs, thresholds, risks) now
3. Leave "Actual" column blank — FORGE fills post-run
4. Leave Section 5 blank — that is SENTINEL's gate

## Definition of Done

- All forecast columns pre-filled
- Known risks documented
- Post-run protocol is clear and executable in 30 minutes
- Section 5 is blank and ready for SENTINEL
- Document is filed and referenced in next FORGE briefing
