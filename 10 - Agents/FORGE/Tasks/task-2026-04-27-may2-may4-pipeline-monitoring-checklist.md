---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-27
status: pending
priority: medium
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/May 2–4 Pipeline Monitoring Checklist.md"
---

# Task: May 2–4 Pipeline Monitoring Checklist

## Context

The May 1 monitoring suite now has three documents:
- `May 1 — Day-Of Runbook` — what FORGE does on May 1 as the pipeline launches
- `May 1 — Morning-After Runbook` — what FORGE checks on May 2 after the first overnight run
- `May 1 — Pipeline Red Signal Response Protocol` — what FORGE does when Morning-After shows RED or YELLOW

These three cover May 1 (launch) and May 2 (Day 1 check). What they do not cover: **what happens on Days 2–4 (May 3–5) if the Day 1 check shows YELLOW signals that haven't crossed into RED escalation.**

The Red Signal Response Protocol defines the May 3 weekend re-check as the key second checkpoint. But the protocol describes the decision logic ("if still YELLOW on May 3 → escalate with 3-day trend data") without specifying the queries, comparison format, or what "3-day trend data" looks like in practice.

FORGE needs a multi-day monitoring checklist that operationalizes the May 3–5 check: specific queries, side-by-side comparison format, trend thresholds, and escalation triggers.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/May 2–4 Pipeline Monitoring Checklist.md`

---

### Section 1: Day 2 Check (May 3 — Weekend Re-Check)

The May 3 check is the designated re-check for any YELLOW signal from May 2. Provide:

**Queries to run (copy-paste ready):**
1. Aggregate game count: total games added since May 1 cron
2. Per-source game count delta: games added May 1 → May 3 vs. May 1 forecast
3. Duplicate rate: any new duplicates added since May 1
4. Date distribution: are new games landing in the expected date range (current season)?

**Pass/fail assessment:**
- **GREEN on May 3:** Aggregate ≥ 300 games added over 2 days → normal weekday-into-weekend behavior confirmed. No escalation. Continue daily check.
- **YELLOW on May 3:** Aggregate 50–299 over 2 days → note specific source(s) lagging. Escalate to SENTINEL with 2-day comparison.
- **RED on May 3:** Same as May 2 RED → immediate escalation per Red Signal Protocol.

---

### Section 2: Day 3 Check (May 4 — First Weekend Run Complete)

May 4 is the first Monday after the pipeline's first full weekend. This is the most meaningful data point for whether the pipeline is healthy.

**Queries to run (copy-paste ready):**
1. Total game count growth: May 1 → May 4 aggregate delta
2. Per-source comparison: forecast vs. actual by source (use `May 1 — Per-Source Game Volume Forecast.md` numbers)
3. gotsport_api specifically: weekend game ingestion confirmed?
4. ecnl_verified population check (if ecnl_verified write-time fix has deployed): `SELECT pct_verified FROM [ecnl_verified check query]`

**Pass/fail assessment:**
- **GREEN on May 4:** ≥ 500 aggregate games (Thursday + 2 weekend days) → pipeline healthy. FORGE files first weekly health review.
- **YELLOW on May 4:** 200–499 games → FORGE files escalation with 3-day trend: May 1, May 3, May 4 counts side-by-side. If specific source is lagging, include source-level breakdown.
- **RED on May 4 (any source):** Escalate immediately per Red Signal Protocol.

---

### Section 3: Day 4 Check (May 5 — 4-Day Trend Complete)

By May 5, FORGE has 4 data points (May 1 launch, May 2 Day-1, May 3 weekend, May 4 Monday). This is enough for a trend assessment.

**Trend assessment format:**

| Date | Aggregate Games Added | gotsport_api | tgs_athleteone | Notes |
|------|----------------------|--------------|----------------|-------|
| May 1 | [actual] | [actual] | [actual] | First run |
| May 2 | [delta] | [delta] | [delta] | |
| May 3 | [delta] | [delta] | [delta] | Weekend |
| May 4 | [delta] | [delta] | [delta] | Monday |
| **4-Day Total** | [sum] | [sum] | [sum] | vs. Forecast: [forecast] |

FORGE files this table as the **First Weekly Pipeline Health Review** (inputs to `task-2026-04-26-first-week-pipeline-monitoring-log.md`).

**May 5 threshold:**
- 4-day aggregate ≥ 700 games: pipeline confirmed viable for DSS-readiness claim
- 4-day aggregate 300–699: flag for SENTINEL — may affect DSS coverage claims
- 4-day aggregate < 300: immediate escalation — likely systematic issue

---

### Section 4: Comparison Table Template

Pre-fill the comparison columns from the Per-Source Forecast so FORGE only needs to add actuals:

| Source | 7-Day Forecast | 4-Day Estimate (57%) | May 1–4 Actual | Delta | Status |
|--------|---------------|---------------------|----------------|-------|--------|
| gotsport_api | [from forecast] | | | | |
| tgs_athleteone | [from forecast] | | | | |
| gotsport_html | [from forecast] | | | | |
| tgs | [from forecast] | | | | |
| tgs_rl | [from forecast] | | | | |
| **Total** | | | | | |

The 57% estimate is a rough heuristic: 4 days includes one Thursday, one Friday, one Saturday, one Sunday — likely 55–60% of weekly game volume given weekend game density.

---

### Section 5: Escalation Package Format

If FORGE escalates to SENTINEL at any point during May 2–5, the escalation must include:

1. **Signal level:** RED or YELLOW
2. **Trigger date:** which day triggered escalation
3. **3-day trend table:** (Section 4 format with actuals filled)
4. **Source-level breakdown:** which source(s) are lagging vs. on-track
5. **FORGE diagnosis:** probable cause (cron issue, auth failure, source-specific, no diagnosis yet)
6. **FORGE recommendation:** escalate to Presten immediately / hold for May 7 check / resolved by X

---

## Quality Standard

All queries in Sections 1–3 must be copy-paste ready — not described ("run a query to check duplicate rate") but written out. FORGE is not designing this during a monitoring session; it is executing a pre-written checklist.

## References

- `Infrastructure/May 1 — Day-Of Runbook.md`
- `Infrastructure/May 1 — Morning-After Runbook.md` (from `task-2026-04-25-daily-pipeline-morning-after-runbook.md`)
- `Infrastructure/May 1 — Pipeline Red Signal Response Protocol.md`
- `Infrastructure/May 1 — Per-Source Game Volume Forecast.md`
- `Infrastructure/First Week Pipeline Monitoring Log.md` (from `task-2026-04-26-first-week-pipeline-monitoring-log.md`)
