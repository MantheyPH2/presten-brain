---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: medium
due: 2026-05-15
deliverable: "02 - Tiger Tournaments/Projects/Rankings/June 1 Age-Group Transition — Presten Brief.md"
---

# Task: June 1 Age-Group Transition — Presten Brief

## Context

ELO completed `task-2026-04-25-age-group-transition-june1-risk.md`, which assessed the technical risks of the June 1 Boys age-group transition. That document contains ELO's analysis.

What does not exist: a short Presten-facing brief — same format as `Rankings/Rank Bands — Presten Decision Brief.md` — that converts the risk assessment into a clear action plan Presten reads in under 5 minutes. The brief answers:
- What happens automatically on June 1 vs what needs Presten action
- What ELO monitors and when
- What risks remain and how they're mitigated
- How June 1 fits in the broader May–June calendar

June 1 and June 2 (ECNL migration) are back-to-back high-stakes events. Presten needs a brief for each. This is the June 1 brief.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Rankings/June 1 Age-Group Transition — Presten Brief.md`

Readable in under 5 minutes. Pull all content from ELO's completed risk assessment — do not re-derive.

---

### Section 1: What Happens Automatically on June 1 (No Action Required)

List each automatic transition the system handles without Presten intervention. For each, state the expected effect on rankings.

Format:
> **[Transition name]:** [What happens]. Expected effect on rankings: [description]. No action required.

Include: which age groups transition, how ratings carry over (or don't), whether any views or computed tables auto-update or require a manual refresh.

If any automatic transitions have a risk of silent failure (e.g., a view that doesn't update until the next recompute), call it out here even if no action is required — Presten should know what to watch for.

### Section 2: What Requires Presten Action (If Any)

If any transitions are not automatic, list each with explicit action, time estimate, and deadline:

| Item | Presten Action | Time Estimate | Deadline |
|------|---------------|---------------|----------|
| [item] | [exact action] | [N minutes] | [date] |

If no Presten action is required, state explicitly: "No Presten intervention required on June 1. ELO monitors and will surface any anomalies by June 3."

### Section 3: ELO Monitoring Plan

After June 1, ELO runs the following checks within 48 hours:

For each check: what it confirms, what a failure looks like, and what ELO does if it fails.

Example format:
> **Check: [name]** — Confirms [what]. Failure: [what it looks like]. Action: [ELO flags to SENTINEL / ELO self-resolves / escalate to Presten].

This sets Presten's expectation: ELO will report June 1 transition status by [date].

### Section 4: Risk Register

Pulled directly from ELO's risk assessment. Maximum three bullets — the highest-probability or highest-impact risks only.

For each risk:
> **[Risk name]:** [probability]. Mitigation: [what's already in place]. Residual risk: [what remains].

If ELO's assessment found no high-priority risks: state "ELO's risk assessment found no high-probability risks. Standard monitoring is sufficient."

### Section 5: Calendar Context

| Date | Event | Who Acts |
|------|-------|----------|
| April 28 | GA ASPIRE fix + ECNL dedup | Presten |
| May 14–15 | DSS validation session | Presten |
| May 17–30 | Boys Brier analysis window | ELO |
| June 1 | Age-group transition | [Auto / Presten per Section 2] |
| June 2 | ECNL migration | Presten |
| June 3 | ELO transition verification | ELO |

Note any sequencing dependencies between June 1 and June 2: if the age-group transition produces unexpected data, does it affect the June 2 ECNL migration? ELO should state the dependency clearly or confirm they are independent.

---

## Definition of Done

- Brief written at `Rankings/June 1 Age-Group Transition — Presten Brief.md`
- All five sections present
- Section 2 is explicit: either lists actions with time estimates, or states "No Presten action required" unambiguously
- Readable in under 5 minutes — no technical derivation, only conclusions and action items
- Calendar table covers April 28 through June 3
- Summary in briefing: "June 1 transition brief written. Presten action required: [Yes — N items / No]. ELO monitoring plan: [N] checks by June 3."

## Why This Matters

June 1 and June 2 are the most consequential system events in the post-DSS window. Presten needs a single planning brief per event, not a technical risk document. Without a brief, June 1 is an unplanned dependency sitting between the DSS session and the ECNL migration. This brief converts ELO's analysis into a Presten action plan.

## References

- `10 - Agents/ELO/Tasks/task-2026-04-25-age-group-transition-june1-risk.md` — source risk assessment (read this first, pull conclusions, do not re-derive)
- `Rankings/Rank Bands — Presten Decision Brief.md` — format to follow
- `10 - Agents/ELO/Tasks/task-2026-04-24-june2-ecnl-migration-verification-spec.md` — adjacent June 2 event (check for sequencing dependencies)
