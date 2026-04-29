---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-28
status: completed
completed: 2026-04-28
priority: urgent
due: 2026-04-30 (must be filed before May 1 pipeline launch)
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/May 1 Stage 1 — Final Launch Authorization Checklist.md"
---

# Task: May 1 Stage 1 — Final Launch Authorization Checklist

## Context

May 1 is the first Stage 1 pipeline run. Multiple prerequisite documents exist (Config Final Review, Deployment Validation Spec, TYSA Checklist, Pipeline Health Check), but no single authoritative pre-launch gate document consolidates all launch conditions into a SENTINEL-ready authorization format.

FORGE has the Config Final Review (Section 5 pending TYSA org-ID). ELO has a Rankings Monitoring Plan. The archive dry-run is overdue. The G0 gate is still open.

This task produces the definitive May 1 launch checklist. SENTINEL will not authorize May 1 launch without this document filed and reviewed.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/May 1 Stage 1 — Final Launch Authorization Checklist.md`

---

### Header

```
Launch Window: May 1, 2026
Launch Type: Stage 1 — TYSA GotSport Crawl (first production run)
Authorization Status: [AUTHORIZED / HOLD / BLOCKED — SENTINEL fills before launch]
Filed by FORGE: [date/time]
Reviewed by SENTINEL: [date/time]
```

---

### Section 1: Required Configuration Complete

| Config Item | Status | Notes |
|-------------|--------|-------|
| TYSA org-ID confirmed and added | [ ] YES / [ ] NO / [ ] PENDING | Section 5 of Config Final Review |
| EDP Source #1 Day 0 config | [ ] YES / [ ] NO — depends on browser session | |
| USL Academy Source #2 Day 0 config | [ ] YES / [ ] NO — depends on browser session | |
| Stage 1 scope limited to TYSA only | [ ] CONFIRMED | |
| No ECNL migration changes in Stage 1 | [ ] CONFIRMED | Stage 1 is ECNL-independent |

### Section 2: Pipeline Infrastructure Ready

| Infrastructure Item | Status | Source | Notes |
|--------------------|--------|--------|-------|
| GROUP BY archive fix deployed | [ ] YES / [ ] NO / [ ] UNCONFIRMED | April 28 session | If NO: archive dry-run cannot run, but does NOT block Stage 1 |
| FM1 activation path confirmed | [ ] YES / [ ] NO / [ ] PENDING G0 | Pipeline Code Review Results | |
| FM2 activation path confirmed | [ ] YES / [ ] NO / [ ] PENDING G0 | Pipeline Code Review Results | |
| Stale team backfill status | [ ] COMPLETE / [ ] INCOMPLETE | Pre-May-1 preferred | Does NOT block Stage 1 |
| Daily pipeline deployment validated | [ ] YES / [ ] NO | Deployment Validation Spec | |

### Section 3: G0 Status and May 1 Independence Confirmation

| Gate | Status at Launch | May 1 Impact |
|------|-----------------|--------------|
| April 28 G0 gate | [ ] GO / [ ] NO-GO / [ ] DEFERRED | Stage 1 is **NOT BLOCKED** by G0 result |
| ecnl_verified backfill | [ ] COMPLETE / [ ] PENDING | Does NOT affect Stage 1 TYSA crawl |
| Girls calibration baseline | [ ] CONFIRMED / [ ] PENDING | Does NOT affect Stage 1 |

**FORGE declaration:** Stage 1 TYSA crawl is independent of G0, ecnl_verified backfill, and girls calibration gates. May 1 launch proceeds on TYSA config status only.

### Section 4: ELO Pre-Launch Baseline Captured

| ELO Item | Status | Notes |
|----------|--------|-------|
| R5 pre-launch stddev baseline | [ ] CAPTURED / [ ] NOT YET | Required before first run per ELO monitoring plan |
| ELO monitoring plan active | [ ] YES | Filed: `Infrastructure/May 1 Pipeline Launch — ELO Rankings Monitoring Plan.md` |

### Section 5: Known Open Items at Launch (Non-Blocking)

List everything that is open but does NOT block Stage 1. This is the "we're launching with eyes open" section. Pre-fill from known blockers:

| Open Item | Why Not Blocking | Due Date |
|-----------|-----------------|----------|
| Archive dry-run | Archive fix independent of crawl pipeline | ASAP post-session |
| FM1/FM2 audit updates | ECNL verification independent of Stage 1 | Post G0 = GO |
| 112-team stale backfill | Optional coverage improvement | Pre-May-3 preferred |
| TYSA Section 5 cert (if no org-ID by April 30) | Stage 1 will still run; org-ID may arrive same-day | May 1 |

### Section 6: SENTINEL Authorization Block

```
SENTINEL LAUNCH AUTHORIZATION:
[ ] Section 1 configs acceptable for launch
[ ] Section 2 infrastructure status acceptable for launch
[ ] Section 3 G0 independence confirmed
[ ] Section 4 ELO baseline confirmed
[ ] Section 5 open items reviewed and accepted as non-blocking

SENTINEL VERDICT: [ ] AUTHORIZED  [ ] HOLD — reason: ___________

Signed: SENTINEL
Date/Time: ___________
```

---

## Definition of Done

- All sections populated with current status (YES/NO/PENDING — no blanks)
- Section 6 SENTINEL authorization block formatted and empty (SENTINEL fills)
- Filed to deliverable path by April 30 EOD so SENTINEL can review before May 1 open
- If TYSA org-ID not received by April 30: Section 1 row 1 shows PENDING with launch proceed justification

## Why This Matters

May 1 is a milestone launch. Without a consolidated authorization document, SENTINEL is reviewing 4+ separate documents at launch time and cannot give a clean authorization signal. This checklist compresses that review to a single document with a single authorization block.

## References

- `Infrastructure/May 1 Stage 1 — Config Final Review and Activation Card.md`
- `Infrastructure/May 1 Pipeline Launch — ELO Rankings Monitoring Plan.md`
- `Infrastructure/Pipeline Deployment Validation Spec.md`
- `task-2026-04-28-april29-pipeline-health-check.md`
- `task-2026-04-28-tysa-config-add-execution-checklist.md`
