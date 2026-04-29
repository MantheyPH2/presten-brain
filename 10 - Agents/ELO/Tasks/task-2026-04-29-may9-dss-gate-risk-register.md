---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-29
priority: medium
due: 2026-05-02
status: pending
deliverable: "02 - Tiger Tournaments/Projects/Rankings/May 9 DSS Gate — Risk Register.md"
tags: [elo, task, may9, dss, risk, sentinel-gate]
---

# Task: May 9 DSS Gate Risk Register

## Context

The May 9 DSS go/no-go gate is 10 days away. ELO's Pre-May-9 Calibration State Dashboard (due May 8) will show current status of each item. What is missing is a forward-looking risk register: what could go wrong between now and May 9 that would flip the gate to NO-GO, and what is the mitigation for each.

SENTINEL will use this register alongside the May 8 dashboard to issue the May 9 verdict. It also helps Presten understand what time-sensitive decisions in the next 10 days are load-bearing for the demo.

## What to Build

Produce: `02 - Tiger Tournaments/Projects/Rankings/May 9 DSS Gate — Risk Register.md`

A structured risk register covering all material risks to the May 9 DSS go/no-go.

## Required Format

### Header

```yaml
---
type: elo-risk-register
topic: may9-dss-gate
date: 2026-04-29
gate_date: 2026-05-09
status: active
sentinel_review_due: 2026-05-08
---
```

### Risk Register Table

One row per identified risk. Minimum 8 risks.

| Risk ID | Risk Description | Probability | Impact | DSS Block? | Mitigation | Owner | Deadline |
|---------|-----------------|-------------|--------|-----------|------------|-------|----------|
| R1 | | Low/Med/High | Low/Med/High | YES/NO | | ELO/FORGE/Presten | |
| R2 | | | | | | | |
| ... | | | | | | | |

**Probability definitions:**
- Low: unlikely given current trajectory
- Medium: possible if a trigger fires
- High: likely without active mitigation

**Impact definitions:**
- Low: degrades demo quality but demo proceeds
- High: forces demo postponement

**DSS Block?**: YES means this risk, if it materializes, prevents the May 9 demo.

### Risks to Include (minimum — ELO may add more)

ELO must assess and include all of the following at minimum:

1. **Girls GA ASPIRE fix not applied before May 9** — April 28 session still not run; if fix never applies, girls rankings are based on mis-labeled data
2. **Boys Option A verdict: FAIL** — if Boys Option A fails and the fallback path isn't implemented before May 9, boys rankings may be uncalibrated
3. **ECNL migration delayed past May 9** — if ECNL option decision is made too late for FORGE to implement + ELO to validate
4. **Event Strength Phase 1 not authorized in time** — if G0 remains NO-GO through May 7, Event Strength activation window closes pre-DSS
5. **Brier score above threshold after GA ASPIRE fix** — if the fix improves girls accuracy but Brier still doesn't meet DSS authorization threshold
6. **Team merges audit not complete for high-priority items** — if a material team merge error is discovered at the May 9 demo
7. **May 1 pipeline instability carrying through May 9** — if Week 1 pipeline runs produce anomalous data that propagates into rankings
8. **USARank comparison not complete** — if ELO cannot demonstrate accuracy vs. the established benchmark at the demo

### Risk Summary

Below the table: a 3–5 sentence summary. How many DSS-blocking risks are there? What is ELO's overall assessment of the current risk level (LOW / MEDIUM / HIGH)? What is the single highest-priority mitigation Presten should focus on this week?

### SENTINEL Handoff Note

> ELO filed this risk register on [date]. It will be updated on May 8 alongside the Pre-May-9 Calibration State Dashboard. Any risk that moves from LOW to HIGH probability before May 8 will trigger an out-of-cycle SENTINEL notification.

---

## Definition of Done

- All 8 minimum risks are assessed with probability, impact, and mitigation
- Every DSS-blocking risk has a named mitigation with an owner and deadline
- Summary is clear and actionable
- ELO confirms filing by May 2 in next briefing
- ELO updates the register when material risk changes occur before May 8

## References

- `Rankings/Pre-May-9 Calibration State Dashboard.md` — calibration state context
- `task-2026-04-26-may9-readiness-final-assessment-spec.md` — readiness assessment
- `task-2026-04-26-may9-dss-readiness-verdict-document.md` — verdict document
- `task-2026-04-28-may9-dss-calibration-readiness-brief.md` — calibration readiness brief
- `task-2026-04-25-dss-demo-spec.md` — DSS demo requirements
