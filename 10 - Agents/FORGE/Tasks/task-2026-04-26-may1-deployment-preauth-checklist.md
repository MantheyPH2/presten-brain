---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-25
priority: high
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/May 1 Deployment Pre-Authorization Checklist.md"
---

# Task: May 1 Deployment Pre-Authorization Checklist

## Context

The May 1 Daily Pipeline Deployment Session Guide is complete and covers the deployment session itself. But before Presten opens that guide on May 1, there is a required gate: the April 29 pipeline health check must be GREEN. There are also several other pre-conditions (April 28 DB changes applied, GA ASPIRE fix confirmed, run-daily.sh in place) that the May 1 guide assumes are true but does not verify.

SENTINEL needs one document Presten opens on the morning of May 1 — before the deployment session — that confirms all pre-conditions are met. If any box is unchecked, deployment does not proceed.

This is not a new process — all the criteria already exist in prior docs. This document consolidates them into a checkbox format.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/May 1 Deployment Pre-Authorization Checklist.md`

Format: a checkbox list Presten fills in on May 1 morning before opening the Session Guide.

### Required Sections

#### Section 1: April 28 DB Changes — Confirmed

- [ ] GA ASPIRE UPDATE ran (`event_tier = 'ga_aspire'` for all ASPIRE events)
- [ ] Full ranking recompute completed successfully
- [ ] April 28 verification queries passed (per Reference Card Section 3)

#### Section 2: April 29 Pipeline Health Check — GREEN

- [ ] Pipeline health check ran on April 29 (per `April 29 Pipeline Health Check — Post-Fix Protocol.md`)
- [ ] Status returned GREEN (all checks pass)
- [ ] No YELLOW or RED items remaining unresolved

#### Section 3: run-daily.sh — In Place

- [ ] `run-daily.sh` file exists at expected path (read from actual project structure)
- [ ] Cron job is configured and points to correct path
- [ ] No syntax errors (run `bash -n run-daily.sh` or equivalent)

#### Section 4: Stage 1 Status — Confirmed

- [ ] Stage 1 TX test crawl status confirmed: PASS / PENDING / FAIL (circle one)
  - If PASS: Stage 3 authorization window opens as of May 2 GREEN morning check
  - If PENDING: Stage 3 authorization deferred; deployment proceeds but Stage 3 not authorized
  - If FAIL: Flag to FORGE before proceeding. Deployment may still proceed; Stage 3 does not.

#### Section 5: SENTINEL Authorization

Single line: "All Section 1–3 boxes checked. Deployment authorized. SENTINEL: ___"

If any Section 1–3 box is unchecked, deployment is held. Presten contacts SENTINEL before proceeding.

---

### What Not to Include

Do not re-document the deployment steps — those are in the May 1 Session Guide. This document's only job is the pre-flight gate. It should be completable in 5 minutes.

## Quality Standard

Each checkbox must reference its source document so Presten can verify if uncertain. For example: "(Confirmed in April 29 Health Check output)" or "(Step 5 of April 28 Reference Card)."
