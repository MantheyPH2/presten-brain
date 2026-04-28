---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-28
due: 2026-04-29
priority: high
status: pending
tags: [elo, task, ecnl, migration, cp1, cp2, checkpoint, sentinel-gate]
---

# Task: ECNL Migration — CP1/CP2 Checkpoint Results Document

## What to Build

A standalone checkpoint results document ELO files immediately after CP1 and CP2 are evaluated on April 29. This is separate from the Option Comparison Matrix's Section 4 (running verdict tracker) — it is a clean, standalone output that SENTINEL reads to confirm checkpoint outcomes without navigating the full comparison matrix.

## Why This Exists

The ECNL migration Option 1 recommendation (filed 2026-04-28) can be disqualified by CP1 (ECNL RL staleness > 30 days) or degraded by CP2 (ecnl_verified backfill < 70%). SENTINEL has pre-authorized Option 1 implementation subject to these checkpoints passing. If either fails, SENTINEL must immediately reconsider authorization and potentially redirect FORGE's pipeline spec activation.

The Option Comparison Matrix Section 4 is an analysis document that embeds the checkpoint results alongside other considerations. SENTINEL needs the checkpoint results in a standalone document that can be referenced without context — especially under session pressure, when a fast authorization revision may be needed.

## Trigger

File within 30 minutes of both CP1 and CP2 being evaluated on April 29. If only CP1 runs (session ends before CP2), file a partial document and note CP2 as pending.

## Required Sections

### Document Header

```yaml
---
type: elo-checkpoint-results
subject: ECNL Migration CP1 + CP2
session_date: 2026-04-29
status: [both-passed | cp1-failed | cp2-failed | both-failed | cp2-pending]
option1-status: [on-track | disqualified | degraded]
sentinel-action-required: [none | review-authorization | urgent-escalation]
---
```

### CP1 — ECNL RL Data Staleness

- **Checkpoint:** ECNL RL most recent game must be within 30 days of April 29
- **Query result:** Most recent ECNL RL game date = [date]
- **Days elapsed:** [N] days
- **Threshold:** ≤ 30 days
- **CP1 Verdict:** PASS / FAIL

**If FAIL:**
> CP1 FAIL means Option 1 (No Re-tag) is disqualified. ECNL RL data is too stale for ELO to assert the Path A/B distinction has current signal. ELO's recommendation must revert to Option 2 or Option 3 pending SENTINEL and Presten review.
> Action: ELO notifies SENTINEL via queue (priority: urgent) within 30 minutes.

**If PASS:**
> Option 1 remains eligible. Proceed to CP2.

---

### CP2 — ecnl_verified Backfill Coverage

- **Checkpoint:** Teams with ecnl_verified = true must represent ≥ 70% of ECNL-eligible teams
- **Query result:** ecnl_verified = true: [N] teams / [total ECNL-eligible] teams = [%]
- **Threshold:** ≥ 70%
- **CP2 Verdict:** PASS / FAIL

**If FAIL:**
> CP2 FAIL means the ecnl_verified field cannot be relied upon to distinguish Path A from Path B teams at the required coverage level. Option 1's no-re-tag approach becomes higher risk — teams that should be tagged may not be. ELO flags this to SENTINEL (priority: high) but does not automatically disqualify Option 1 unless SENTINEL directs.
> Note: FORGE's `task-2026-04-27-ecnl-verified-backfill-sql-package.md` addresses backfill — confirm whether the April 28 session ran the backfill SQL.

**If PASS:**
> Option 1 coverage threshold met. Both CPs passed.

---

### Combined Verdict

| CP | Result | Option 1 Impact |
|----|--------|----------------|
| CP1 | [PASS/FAIL] | [None / Disqualified] |
| CP2 | [PASS/FAIL] | [None / Elevated risk] |

**Overall:** Option 1 status = [ON-TRACK / DISQUALIFIED / ON-TRACK WITH ELEVATED RISK]

### SENTINEL Action Required

State exactly what SENTINEL should do based on the combined result:
- Both PASS: "No action — Option 1 authorized path unchanged."
- CP1 FAIL: "Urgent review required — Option 1 disqualified. Revise authorization."
- CP2 FAIL only: "Review elevated risk position. Confirm Option 1 authorization stands or redirect to Option 2."

## File When Done

`02 - Tiger Tournaments/Projects/Rankings/ECNL Migration — CP1 CP2 Checkpoint Results.md`

Mark this task `status: completed` in frontmatter.
File to SENTINEL queue immediately if any CP fails (priority: urgent for CP1, high for CP2 only).
Update Option Comparison Matrix Section 4 (running verdict tracker) in the same briefing session.
