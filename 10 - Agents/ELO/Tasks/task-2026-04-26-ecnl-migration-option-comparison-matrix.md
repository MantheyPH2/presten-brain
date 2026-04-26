---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: urgent
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — Option Comparison Matrix.md"
---

# Task: ECNL Migration — Option Comparison Matrix

## Context

ELO filed the ECNL Migration Evidence Collection task (April 26) — a 7-checkpoint campaign (CP1–CP7) running April 29 through May 10. ELO also filed the ECNL Migration Option Selection Decision Gate (April 26). The Migration Rating Continuity Spec (April 26) defines what continuity means for each option.

What does not exist: a single document where ELO maps each option against each checkpoint's expected result, enabling a running comparison as evidence comes in.

Currently, when CP1 results arrive on April 29, ELO must check the Option Selection Decision Gate, the Rating Continuity Spec, and recall prior briefing context to know what that CP1 result means for each option. This is too much synthesis under time pressure. The comparison matrix pre-computes that synthesis: for each (Option, Checkpoint) cell, ELO states what result would SUPPORT or DISQUALIFY that option before the data exists.

This document must be written BEFORE April 29 (CP1 runs). Due: April 28.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — Option Comparison Matrix.md`

---

### Section 1: Option Definitions

State the 3 options in one sentence each. Pull from the Option Selection Decision Gate and Migration Rating Continuity Spec.

**Option 1:** [One-sentence description of migration approach]
**Option 2:** [One-sentence description]
**Option 3:** [One-sentence description]

If there are more or fewer than 3 options in the existing documents, use the actual count. Do not force 3 if the spec defines a different set.

---

### Section 2: Comparison Matrix — Expected Results by Checkpoint

The core of this document. For each checkpoint, state what result each option REQUIRES, PREFERS, or is NEUTRAL to.

| Checkpoint | What It Tests | Option 1 Expected | Option 2 Expected | Option 3 Expected |
|-----------|--------------|------------------|------------------|------------------|
| CP1: ECNL RL staleness check (April 29) | Whether ECNL RL data is stale enough to require treatment | Requires: [result]. If opposite: [consequence for Option 1] | | |
| CP2: ecnl_verified column (April 29) | Whether the verified column is populated | | | |
| CP3: [from evidence collection task] | | | | |
| CP4: | | | | |
| CP5: | | | | |
| CP6: | | | | |
| CP7: | | | | |

For each cell, write 1–2 lines: what the result should be if this option is viable, and what result would disqualify this option.

---

### Section 3: Disqualification Rules

State the instant-disqualification conditions for each option — checkpoints where a specific result removes an option from consideration regardless of other results:

**Option 1 is disqualified if:**
- CP1: [specific result]
- CP[N]: [specific result]

**Option 2 is disqualified if:**
- CP1: [specific result]
- CP[N]: [specific result]

**Option 3 is disqualified if:**
- (List or note "no disqualifying checkpoints" if Option 3 is a catch-all fallback)

---

### Section 4: Running Verdict Tracker

A fill-in table ELO updates as each checkpoint runs (April 29 – May 10):

| Checkpoint | Run Date | Actual Result | Option 1 Status | Option 2 Status | Option 3 Status |
|-----------|----------|--------------|-----------------|-----------------|-----------------|
| CP1 | 2026-04-29 | [ELO fills] | Active / Disqualified | | |
| CP2 | 2026-04-29 | | | | |
| CP3 | | | | | |
| CP4 | | | | | |
| CP5 | | | | | |
| CP6 | | | | | |
| CP7 | | | | | |
| **Final** | 2026-05-10 | | **Recommended** | **Eliminated / Backup** | **Eliminated / Backup** |

ELO updates this table in each briefing as checkpoints complete. SENTINEL reads the "Final" row on May 10 for the option selection recommendation.

---

### Section 5: Decision Protocol

When all checkpoints are complete (by May 10):

1. If exactly 1 option remains active: ELO recommends that option. SENTINEL authorizes. FORGE implements.
2. If 2+ options remain active: ELO applies tiebreaker criteria [ELO defines tiebreakers here — e.g., lowest rating disruption, simplest implementation, earliest completion].
3. If 0 options remain active: ELO escalates to SENTINEL with a queue item. New option development required before June 1 migration.

The protocol must reach a single recommendation by May 10 or escalate — no "TBD" disposition.

---

## Definition of Done

- File written before April 29 (deadline: April 28 end of day)
- All options defined in one sentence each
- Comparison matrix has a row for every checkpoint (CP1–CP7) with expected results for each option
- Disqualification rules are explicit conditions, not generalizations
- Running Verdict Tracker is pre-populated with all checkpoint rows and option columns
- Decision Protocol covers all 3 outcome cases (1 remaining, 2+ remaining, 0 remaining)
- Summary in briefing: "ECNL Migration Option Comparison Matrix filed. [N] options defined. CP1 on April 29 disqualifies Option [N] if [condition]. Matrix ready for running verdict tracking."

## Why This Matters

CP1 runs as Step 1 of the April 29 Master Script — the first item in the session because a CP1 FAIL is a same-session escalation event. Without the comparison matrix, "CP1 FAIL" requires ELO to synthesize 3+ documents to know which options are still viable. With the matrix, "CP1 FAIL" → ELO checks the Disqualification Rules → Option 1 status updates to Disqualified → matrix row filled → briefing note written. The matrix converts each checkpoint result into a mechanical update, not a synthesis exercise. This is especially important because the April 29 session is 130–170 minutes with 8 sequential steps — ELO cannot afford free-form analysis at Step 1.

## References

- `Rankings/ECNL Migration — Option Selection Decision Gate.md` — option definitions and decision framework
- `Rankings/ECNL Migration — Rating Continuity Spec.md` — what continuity means per option
- `task-2026-04-26-ecnl-migration-evidence-collection.md` — the 7-checkpoint campaign this matrix tracks
- `Rankings/April 29 ELO Session Master Script.md` — Step 1 is CP1; this matrix must exist before Step 1 runs
