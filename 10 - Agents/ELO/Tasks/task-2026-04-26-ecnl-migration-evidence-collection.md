---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: medium
due: 2026-05-10
deliverable: "02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — Option Selection Decision Gate.md (all checkpoints completed)"
---

# Task: ECNL Migration — Evidence Collection Campaign (April 29 – May 10)

## Context

ELO filed `Rankings/ECNL Migration — Option Selection Decision Gate.md` on April 26, including a 7-checkpoint evidence collection schedule running April 29 through May 10. The decision gate document defines what evidence must be collected; no task was created to execute that collection campaign.

The ECNL June 1 migration is a hard deadline. If ELO does not have Option 1 vs. Option 2 evidence filed by May 10, SENTINEL cannot authorize an option selection, and the migration proceeds without a structured plan — risking rating discontinuity for ECNL teams at the start of the Fall season.

ELO's briefing (2026-04-26 06:39) confirmed: "ECNL RL staleness is a load-bearing April 29 data point. If ECNL RL > 30 days stale, Option 1 is disqualified." This means the first checkpoint on April 29 is the highest-stakes one.

This task: execute all 7 checkpoints and complete the decision gate document.

## What to Do

**Target file:** `02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — Option Selection Decision Gate.md`

Work through the evidence collection schedule. After each checkpoint, update the corresponding row in the decision gate document before filing the ELO briefing for that session.

---

### Checkpoint Schedule

| Checkpoint | Date | What to Collect | Decision Gate Row to Update |
|------------|------|-----------------|----------------------------|
| CP1 | April 29 | ECNL RL staleness: run the staleness verification SQL from `task-2026-04-26-ecnl-rl-staleness-execution-package.md`. Is ECNL RL ≤ 30 days? | Option 1 viability gate |
| CP2 | April 29 | ecnl_verified column exists and has correct default: confirm from Section 4 of Source Baseline update | ecnl_verified prerequisite row |
| CP3 | May 1 | First pipeline run ingests ECNL games: check games table for ECNL source rows with run_date = May 1 | Pipeline ingestion gate |
| CP4 | May 3 | ECNL game volume after 3 runs: is the per-run ECNL game count ≥ expected threshold from the Ingestion Path Audit? | Data volume gate |
| CP5 | May 5 | ecnl_verified flag accuracy spot check: sample 20 ECNL games, manually verify source tag matches ecnl_verified=TRUE assignment | Accuracy gate |
| CP6 | May 7 | Rating stability for ECNL teams in first-week pipeline: do ECNL team ratings track predictably with new games added? | Calibration continuity gate |
| CP7 | May 10 | Final option selection: all gates met → select Option 1 or Option 2; file decision | Final decision row |

---

### CP1 (April 29) — Critical Path

CP1 is the highest-priority checkpoint. The ECNL RL staleness check determines whether Option 1 (preferred path) is viable.

**If ECNL RL ≤ 30 days stale:** Option 1 remains viable. Continue collection campaign.

**If ECNL RL > 30 days stale:** Option 1 is disqualified. ELO must immediately:
1. Update the Decision Gate with "Option 1: DISQUALIFIED — ECNL RL staleness > 30 days"
2. Flag in the April 29 ELO briefing: "ECNL migration escalation — Option 2 is now the only viable path, requires Presten sign-off"
3. File a Queue item to SENTINEL flagging the escalation

Do not wait for subsequent checkpoints before flagging. This is a same-session action.

---

### Filing Protocol

After each checkpoint:
1. Run the queries or checks specified
2. Update the corresponding row in the Decision Gate document
3. Add a one-line entry to the ELO briefing: "ECNL migration CP[N]: [result one sentence]"

Do not batch checkpoints. Each one is filed in the session it runs.

---

### May 10 Final Decision

After CP7, complete the final option selection block in the Decision Gate:

```
ELO option selection: Option 1 / Option 2
Basis: [cite which checkpoints confirm the selected option]
Known risks: [any outstanding concerns]
Presten sign-off required: YES (Option 2) / NO (Option 1)
Authorization request to SENTINEL: [date]
```

---

## Quality Standard

- CP1 on April 29 is non-negotiable. If the April 29 session is crowded with G1–G4 work, CP1 should be run first — it takes ≤ 10 minutes and resolves the highest-stakes question.
- No checkpoint result should be estimated from prior data. Run the actual queries.
- If a checkpoint cannot be run on its scheduled date (e.g., pipeline not yet up for CP3), note the delay in the ELO briefing and reschedule within 48 hours.
- The May 10 deadline is hard. If ELO reaches May 9 with outstanding checkpoints, run them all in the May 9 session.

## Why This Matters

The ECNL June 1 migration affects all ECNL teams' rating continuity. Option 1 (preferred, no Presten action required) has a disqualifying condition that must be checked April 29. Option 2 (requires Presten manual UPDATE sign-off) needs lead time to schedule the Presten session. If ELO does not run the evidence collection campaign, the option selection defaults to "figure it out in May" — which, given the team merges audit and DSS prep competing for time, likely means a rushed migration decision in late May.

## References

- `Rankings/ECNL Migration — Option Selection Decision Gate.md` — the document to populate
- `task-2026-04-26-ecnl-rl-staleness-execution-package.md` — CP1 execution package (SQL + procedure)
- `task-2026-04-24-ecnl-rating-continuity-spec.md` — Option 1 and Option 2 technical definitions
- `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md` — CP3 and CP4 data source
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — CP6 cross-reference
