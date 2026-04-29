---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
status: completed
completed: 2026-04-29
priority: urgent
due: 2026-04-29 (must be ready before April 29 session opens)
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Option A — April 29 Execution Package.md"
---

# Task: Boys Option A — April 29 Execution Package

## Context

Boys Option A evaluation opens on April 29. The task exists (`task-2026-04-27-ga-coverage-audit-boys-query-package.md` and related boys calibration tasks), but no single execution-ready package exists that consolidates the queries, decision criteria, and outcome filing format.

On April 29, Presten will run Boys Option A queries. ELO needs to be ready to interpret results and file the Boys Option A decision document within the session. Without a pre-structured package, ELO spends the first 30 minutes of April 29 reconstructing the sequence from 4+ prior tasks.

This task produces that package now, from existing documents. G0 status does not affect Boys Option A — it is explicitly G0-independent.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Boys Option A — April 29 Execution Package.md`

---

### Section 1: Context — What Boys Option A Is

Pull from existing ELO calibration documents. Summarize in 3–5 sentences:
- What "Option A" means for boys calibration
- Why April 29 is the opening window
- What verdict is required (binary GO/NO-GO or a spectrum?)
- What changes in the model if Option A is confirmed

---

### Section 2: Query Execution Package (copy-paste ready)

Structure as an ordered table with exactly what Presten runs:

| Step | Query ID | Purpose | Query | Expected Output Range | GREEN / YELLOW / RED |
|------|----------|---------|-------|-----------------------|----------------------|
| 1 | B-OA-1 | | | | |
| 2 | B-OA-2 | | | | |
| ... | | | | | |

Pull queries from `task-2026-04-25-boys-brier-analysis-scope-spec.md`, `task-2026-04-24-boys-calibration-gap-analysis.md`, `task-2026-04-24-boys-calibration-option-a-interpretation.md`, and any other boys calibration task files.

**IMPORTANT:** Label which queries are "required for verdict" vs "supplemental context." Presten should be able to run only the required queries in a 15-minute session and get a valid verdict.

---

### Section 3: Decision Criteria

| Verdict | Conditions |
|---------|-----------|
| Boys Option A: PASS | [define exactly — what numbers confirm it] |
| Boys Option A: FAIL | [define exactly — what triggers failure] |
| Boys Option A: INCONCLUSIVE | [define — what triggers another cycle] |

If Option A FAILS: what is the recovery path? (Pull from `task-2026-04-28-boys-option-a-fail-recovery-brief.md`.)

---

### Section 4: Post-Verdict Filing Protocol

On verdict:

| Verdict | ELO Action | File Location | SENTINEL Notification |
|---------|-----------|--------------|----------------------|
| PASS | File Boys Option A Decision Document | `Rankings/Boys Option A Decision Document.md` | Notify in briefing |
| FAIL | Activate recovery brief | `Rankings/Boys Option A Fail Recovery Brief.md` | File SENTINEL queue alert |
| INCONCLUSIVE | Document results, schedule next window | `Rankings/Boys Option A Decision Document.md` (partial) | Notify in briefing |

---

### Section 5: May 1 Pipeline Interaction

Confirm Boys Option A evaluation does NOT require delaying May 1 Stage 1 pipeline launch. State the independence clearly: "Boys Option A evaluation is a calibration-layer analysis. It does not modify the May 1 TYSA crawl configuration."

Also confirm: during Boys Option A evaluation window (April 29–May 5), USL Academy Source #2 boys threshold is 50 pts (not standard). ELO notifies FORGE when Option A verdict is final.

---

## Definition of Done

- Sections 1–5 populated from existing ELO calibration task documents
- All queries are copy-paste ready (Presten runs them directly)
- Decision criteria are specific and numeric (no ambiguity at verdict time)
- Filed before April 29 open (tonight or early April 29)
- G0 independence explicitly confirmed in Section 5

## Why This Matters

Boys Option A is April 29's critical deliverable alongside the G0 gate. If ELO is not ready with a structured package, the April 29 session burns synthesis time instead of execution time. This package converts a 45-minute reconstruction exercise into a 10-minute run.

## References

- `task-2026-04-28-boys-option-a-fail-recovery-brief.md`
- `task-2026-04-28-boys-calibration-options-synthesis-brief.md` (blocked pending verdict, but consult for context)
- `task-2026-04-25-boys-brier-analysis-scope-spec.md`
- `task-2026-04-24-boys-calibration-gap-analysis.md`
- `task-2026-04-24-boys-calibration-option-a-interpretation.md`
- `task-2026-04-24-mlsnext-boys-april28-go-no-go.md`
