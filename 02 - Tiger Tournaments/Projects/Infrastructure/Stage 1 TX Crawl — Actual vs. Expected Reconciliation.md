---
title: Stage 1 TX Crawl — Actual vs. Expected Reconciliation
tags: [infrastructure, gotsport, stage1, texas, pipeline, reconciliation, stage2-gate, evo-draw]
created: 2026-04-28
updated: 2026-04-28
author: FORGE
status: template-ready — fill in within 24 hours of Stage 1 completing first verified TX crawl run
task: task-2026-04-28-stage1-actual-vs-expected-reconciliation
due: within 24 hours of Stage 1 first verified run (target ~2026-05-02)
---

# Stage 1 TX Crawl — Actual vs. Expected Reconciliation

> **Filing trigger:** Stage 1 completes its first verified TX crawl run after the May 1 pipeline deployment. File within 24 hours.
>
> **Expected output spec:** `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md`
> **Post-run checklist:** `Infrastructure/Stage 1 TX Crawl — Post-Run Validation Checklist.md`
> **Stage 1→2 transition criteria:** `task-2026-04-27-stage1-to-stage2-transition-protocol.md`
>
> SENTINEL reads this report and issues written Stage 2 authorization or flag within 24 hours.

---

## Pre-Filing Gate

- [ ] Stage 1 post-run validation checklist completed
- [ ] Expected output spec open for comparison (pull thresholds into Section 2)
- [ ] No active pipeline run in progress at time of filing

---

## Section 1: Run Metadata

| Field | Value |
|-------|-------|
| Stage 1 TX crawl execution date | [date] |
| Pipeline run start time | [time] |
| Pipeline run completion time | [time] |
| TYSA org-ID used | [org_id] |
| Other org-IDs included in Stage 1 run | [list or NONE] |
| Pipeline script version / commit | [hash or N/A] |
| Run triggered by | [cron / manual / Presten] |

---

## Section 2: Game Volume — Actual vs. Expected

Pull thresholds from `Infrastructure/Stage 1 TX Crawl — Expected Output Spec.md`.

| Metric | Expected (from output spec) | Actual | Delta | Pass / Fail |
|--------|----------------------------|--------|-------|-------------|
| Total TX games ingested | ≥ [threshold] | [actual] | [delta] | |
| Unique teams in TX games | ≥ [threshold] | [actual] | [delta] | |
| Date range of games ingested | [range from spec] | [actual range] | | |
| ECNL TX games | ≥ [threshold] | [actual] | [delta] | |
| MLS NEXT TX games | ≥ [threshold] | [actual] | [delta] | |

**If any metric FAILS:** Describe whether it is a data gap, config error, or expected miss. Note whether it blocks Stage 2 eligibility.

---

## Section 3: Data Quality Assessment

| Check | Result | Notes |
|-------|--------|-------|
| Duplicate game records | [count] | Dedup mechanism applied: YES / NO |
| Teams matched to existing entities | [%] | |
| Unmatched new teams created | [count] | |
| Unexpected org-ID patterns (wrong state/teams) | YES / NO | If YES: describe |
| ecnl_verified flag coverage for ECNL TX teams | [%] | Expected: 100% post-Step-2b backfill |

**Data quality overall:** [CLEAN / FLAG — one sentence summary]

---

## Section 4: Validation Checklist Result

Reference: `Infrastructure/Stage 1 TX Crawl — Post-Run Validation Checklist.md`

**Overall result:** ALL PASSED / [N] ITEMS FAILED

Failed items (if any):

| Item | Severity | Description |
|------|----------|-------------|
| [checklist item] | HIGH / MEDIUM / LOW | [description] |

---

## Section 5: Stage 2 Eligibility Verdict

Reference: `task-2026-04-27-stage1-to-stage2-transition-protocol.md`

| Criterion | Met? | Supporting fact |
|-----------|------|-----------------|
| Game volume threshold met | YES / NO | [actual vs. threshold] |
| Data quality threshold met | YES / NO | [dedup rate, match %, ecnl_verified %] |
| No critical pipeline errors | YES / NO | [error log summary] |
| SENTINEL Stage 2 authorization | PENDING | SENTINEL authorizes after reading this report |

**FORGE Stage 2 Recommendation:** READY / NOT READY / CONDITIONAL

> [One sentence. State recommendation and primary supporting fact. Example: "Stage 1 ingested 42,000 TX games with 98.3% team match rate and 0 critical errors — FORGE recommends SENTINEL authorize Stage 2 expansion."]

---

## Section 6: Stage 2 Authorization Request

> FORGE requests SENTINEL issue written Stage 2 authorization. Stage 1 TX crawl completed [DATE]. Game volume: [N] games ([vs. threshold]). Data quality: [CLEAN / FLAG]. Checklist: [ALL PASSED / N FAILED]. FORGE recommendation: [READY / NOT READY / CONDITIONAL]. See Sections 2–5 for detail.

**SENTINEL response required within 24 hours of this report.**

---

## Filing Confirmation

- [ ] Document filed at `Infrastructure/Stage 1 TX Crawl — Actual vs. Expected Reconciliation.md`
- [ ] Stage 2 recommendation reported to SENTINEL in same briefing session
- [ ] Task `task-2026-04-28-stage1-actual-vs-expected-reconciliation.md` status updated to `completed`

---

*FORGE — template pre-staged 2026-04-28 | Fill in within 24 hours of Stage 1 first verified run (~2026-05-02)*
