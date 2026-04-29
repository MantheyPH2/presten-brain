---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-29
priority: high
due: 2026-04-30 EOD
hard_deadline: true
status: pending
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Brier — Pre-May-17 Execution Package.md"
tags: [elo, task, brier, boys-calibration, hard-deadline]
---

# Task: Boys Brier Pre-May-17 Execution Package

## SENTINEL Notice

This task has been assigned in four separate cycles with no vault-level output filed. The Boys Brier pre-May-17 execution package has **no external dependencies** — it is pure documentation. This is now a hard deadline.

**Due: April 30 EOD. No extensions.**

If this task is not delivered by April 30 EOD, ELO will be FLAGGED on the Boys Brier item in SENTINEL's next briefing and the pattern will be escalated to Presten.

---

## What to Build

Produce: `02 - Tiger Tournaments/Projects/Rankings/Boys Brier — Pre-May-17 Execution Package.md`

A self-contained, ready-to-execute package that Presten can pick up and run without referencing any other document.

## Why This Exists

The U13/U14 K-factor/RD fix (code change in `compute-rankings.js`) cannot deploy before May 17. Before deploying, a Boys Brier pre-check must confirm the fix improves predictive accuracy. If the Brier check is not pre-staged, the May 17 deployment decision will be delayed while ELO assembles the package under time pressure.

## Required Format

### Header

```yaml
---
type: elo-execution-package
topic: boys-brier-pre-may17
date: 2026-04-29
deployment_window: 2026-05-17
status: ready-for-execution
---
```

### Section 1: What This Is

2–3 sentences. What is a Brier score, what does the Boys Brier check measure, and what threshold must be met before the U13/U14 fix deploys.

### Section 2: Execution Window

When does this run? State: "Run anytime May 10–16 (before May 17 deploy window opens)." Note that it requires Presten psql access.

### Section 3: SQL Queries

**Copy-paste ready.** Three queries, each with a clear label:

1. **B-BR-1: Boys Brier Score by Age Group**
   - What it queries: predicted vs. actual game outcomes for boys age groups
   - Expected output: one row per age group with brier score
   - Pass threshold: [state ELO's threshold — e.g., < 0.24 or equivalent for boys]

2. **B-BR-2: U13/U14 Brier Pre-Fix Baseline**
   - What it queries: U13/U14 specific Brier score using current compute logic (pre-fix)
   - Expected output: single value
   - Purpose: comparison baseline

3. **B-BR-3: U13/U14 Brier Post-Fix Projection**
   - What it queries: simulated Brier score using the proposed K-factor/RD fix parameters
   - Expected output: single value
   - Pass criteria: lower than B-BR-2 result

If ELO cannot write exact SQL without DB access, write the query structure with placeholder values clearly labeled. The package must be usable without ELO present.

### Section 4: Pass/Fail Decision Gate

| Result | Condition | ELO Action | Deployment Decision |
|--------|-----------|------------|---------------------|
| PASS | B-BR-3 < B-BR-2 AND overall Boys Brier < [threshold] | File verdict document; notify SENTINEL | Proceed with May 17 deploy |
| CONDITIONAL | B-BR-3 < B-BR-2 BUT overall above threshold | File analysis; request SENTINEL review | Hold pending SENTINEL decision |
| FAIL | B-BR-3 ≥ B-BR-2 | File failure report; escalate to Presten | DO NOT DEPLOY May 17 |

### Section 5: Post-Execution Protocol

Step-by-step for what ELO does after Presten runs the queries:
1. Receive query results
2. File verdict in: `Rankings/Boys Brier — Pre-May-17 Verdict.md`
3. Notify SENTINEL within [X] hours
4. If PASS: confirm May 17 deploy window open
5. If FAIL or CONDITIONAL: file analysis report

### Section 6: SENTINEL Authorization Request

> ELO certifies this execution package is complete and accurate. This package may be executed at any time during May 10–16. ELO requests SENTINEL review of this package before May 10 to confirm pass/fail thresholds are appropriate.

---

## Definition of Done

- All six sections are complete
- SQL queries are copy-paste ready (with any placeholders clearly labeled)
- Pass/fail thresholds are stated explicitly
- Document is self-contained — Presten can execute without referencing other files
- ELO confirms filing by April 30 EOD in next briefing

## References

- `task-2026-04-22-brier-score-completion.md` — original Brier task context
- `task-2026-04-25-boys-brier-analysis-scope-spec.md` — scope spec
- `task-2026-04-28-boys-brier-pre-may17-execution.md` — prior (undelivered) task; use its scope
- `task-2026-04-23-u13-u14-calibration-fix.md` — the fix this Brier check gates
