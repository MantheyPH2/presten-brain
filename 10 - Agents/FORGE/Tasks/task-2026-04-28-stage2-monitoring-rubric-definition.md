---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: open
priority: medium
due: 2026-05-05 (before Week 1 monitoring data arrives May 3)
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Authorization — Week 1 Monitoring Rubric.md"
---

# Task: Stage 2 Authorization — Week 1 Monitoring Rubric

## Context

Stage 2 expansion (adding EDP, USL Academy, additional GotSport org-IDs) requires SENTINEL authorization after 3 pipeline runs (approximately May 3). The Stage 2 Expansion Trigger Document exists and sets criteria in general terms. The Stage 2 Preauth Criteria task was filed April 25.

What does not exist: a structured, numeric rubric that tells SENTINEL exactly how to evaluate Week 1 data when filing the authorization decision. Without this, SENTINEL's Week 1 review requires re-reading 4 documents and synthesizing thresholds on the fly.

This task produces a single rubric document that SENTINEL reads on May 4–5 to issue a GO/NO-GO on Stage 2. FORGE populates it from existing thresholds in filed documents — no new decisions required.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/Stage 2 Authorization — Week 1 Monitoring Rubric.md`

---

### Section 1: Stage 2 Authorization Decision Context

| Item | Value |
|------|-------|
| Authorization decision date | ~May 4–5 (after 3 Stage 1 runs: May 1, 2, 3) |
| Authorization issued by | SENTINEL |
| Stage 2 go-live target | ~May 17–20 |
| Sources in Stage 2 | EDP (Source #1), USL Academy (Source #2), TYSA expansion org-IDs |

---

### Section 2: Game Volume Rubric (populate from Pipeline Week 1 Actuals Template)

Pull the GREEN/YELLOW/RED floors from `Infrastructure/Pipeline Week 1 — Game Volume Actuals Report.md`.

| Source | Expected Daily Games | GREEN Floor | YELLOW Range | RED Trigger |
|--------|---------------------|-------------|--------------|-------------|
| TYSA | | | | |
| (Additional Stage 1 sources if added) | | | | |
| **Total Stage 1 Volume** | | | | |

**SENTINEL threshold for Stage 2 authorization:**
- GREEN overall → Stage 2 authorized as scheduled
- YELLOW → SENTINEL reviews specific source before authorizing
- RED any single source → Stage 2 held pending FORGE investigation

---

### Section 3: Pipeline Health Rubric (populate from April 29 Infrastructure Confirmation Report structure)

| Health Check | GREEN Condition | YELLOW Condition | RED Condition |
|-------------|----------------|-----------------|--------------|
| No failed crawls in Week 1 | 0 failed runs | 1 failed run (recovered) | 2+ failed runs |
| Duplicate rate | | | |
| Stale game rate (games not updated >48 hrs) | | | |
| Error log volume | | | |

Pull thresholds from `task-2026-04-24-source-health-monitoring-queries.md` or equivalent.

---

### Section 4: Ranking Signal Health Rubric (from ELO monitoring plan)

Pull R1–R5 from `Infrastructure/May 1 Pipeline Launch — ELO Rankings Monitoring Plan.md`.

| ELO Check | Check ID | GREEN Threshold | SENTINEL Action if YELLOW/RED |
|-----------|----------|-----------------|-------------------------------|
| | R1 | | |
| | R2 | | |
| | R3 | | |
| | R4 | | |
| | R5 (stddev baseline) | | |

**SENTINEL threshold for Stage 2 authorization:** All R1–R5 GREEN → authorized. Any R-check RED → Stage 2 held; ELO must file resolution before SENTINEL re-evaluates.

---

### Section 5: Stage 2 Source Readiness Pre-Checks (fill from EDP and USL Academy build plans)

Before Stage 2 can launch, these source-level gates must pass regardless of Week 1 volume data:

| Source | Gate | Status at Authorization | Notes |
|--------|------|------------------------|-------|
| EDP (Source #1) | Day 0 config activated (browser session) | | |
| EDP (Source #1) | Gate 1 criteria per build plan | | |
| USL Academy (Source #2) | Day 0 config activated (browser session) | | |
| USL Academy (Source #2) | Boys threshold: 50 pts during Option A window (April 29–May 5) | | |
| USL Academy (Source #2) | ELO written confirmation required before Gate 2 | | |

---

### Section 6: SENTINEL Authorization Block (fills on authorization decision)

```
SENTINEL STAGE 2 REVIEW — ~May 4–5, 2026

Section 2 Game Volume: [ ] GREEN  [ ] YELLOW  [ ] RED
Section 3 Pipeline Health: [ ] GREEN  [ ] YELLOW  [ ] RED
Section 4 ELO Rankings: [ ] GREEN  [ ] YELLOW  [ ] RED
Section 5 Source Readiness:
  EDP: [ ] READY  [ ] NOT READY
  USL Academy: [ ] READY  [ ] NOT READY

SENTINEL VERDICT:
[ ] STAGE 2 AUTHORIZED — target go-live May 17–20
[ ] STAGE 2 HOLD — reason: ___________
[ ] STAGE 2 CONDITIONAL — conditions: ___________

Signed: SENTINEL
Date: ___________
```

---

## Definition of Done

- Sections 1–5 populated from existing filed documents (no new decisions required from FORGE)
- All numeric thresholds pulled from authoritative source documents (cite each)
- Section 6 SENTINEL authorization block blank and ready
- Filed by May 5 so SENTINEL can use it for the Week 1 review

## Why This Matters

Without this rubric, SENTINEL's Week 1 review requires reading 4+ documents and manually reconstructing thresholds at authorization time. This task converts that effort into a 5-minute rubric review. Stage 2 is the first pipeline expansion — the authorization should be clean, not synthesized under time pressure.

## References

- `Infrastructure/Pipeline Week 1 — Game Volume Actuals Report.md`
- `Infrastructure/May 1 Pipeline Launch — ELO Rankings Monitoring Plan.md`
- `task-2026-04-25-stage2-expansion-preauth-criteria.md`
- `task-2026-04-24-source-health-monitoring-queries.md`
- `Infrastructure/Non-GotSport Source #1 — Implementation Build Plan.md`
- `Infrastructure/Non-GotSport Source #2 — USL Academy Implementation Build Plan.md`
