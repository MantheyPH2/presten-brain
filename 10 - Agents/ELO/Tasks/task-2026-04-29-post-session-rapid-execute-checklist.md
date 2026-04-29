---
type: agent-task
assigned_by: SENTINEL
assigned_to: ELO
date: 2026-04-29
time: "11:15"
priority: high
due: "April 30 EOD (ready before session runs)"
status: pending
topic: post-session-rapid-execute-checklist
---

# Task: Post-Session Rapid-Execute Checklist

## Objective

Create a single document covering everything ELO must do within 60 minutes of Presten completing the April 29 session. Five separate tracked items update simultaneously on session execution. A rapid-execute checklist eliminates coordination risk and ensures no step is missed.

## Background

When the April 29 session finally runs, the following happen in rapid succession:
1. GA ASPIRE UPDATE executed → G0 re-evaluation possible
2. ecnl_verified ALTER TABLE + backfill → ECNL CP1 can proceed
3. Rankings recompute → Boys Option A queries can run
4. G0 confirmed → Event Strength Pre-Draft [TO FILL] cells fillable
5. Stddev baseline query can run → May 1 Baseline Section 2 fillable

Without a checklist, ELO risks completing steps 1–2 and missing steps 3–5 in the same session. Or completing the session without notifying FORGE that G0 is confirmed. This document is the single source of truth for post-session ELO actions.

## Deliverable

**File:** `Rankings/Post-April-29-Session — ELO Rapid-Execute Checklist.md`

## Required Content

### Header

State: "Execute this checklist immediately after Presten completes the April 29 DB session. Target: all steps complete within 60 minutes. Steps are ordered by dependency — do not skip ahead."

### The Checklist (ordered)

Present as numbered steps with time estimates and output file for each:

**Step 1 — Confirm GA ASPIRE Fix Applied (5 min)**
- Query: verify the GA ASPIRE event_tier UPDATE was executed (row count confirmation)
- If applied: record confirmation in `Rankings/May 1 Pipeline Launch — ELO Ratings Baseline.md` Section 5 exception note as RESOLVED
- If not applied: flag in SENTINEL Queue immediately before proceeding

**Step 2 — Confirm G0 (10 min)**
- Run G0 gate checks per `Rankings/April 29 — Gate Checks Execution Log Template.md`
- Record G0 result (GO / NO-GO with reason)
- If GO: proceed to Step 3
- If NO-GO: stop and notify SENTINEL via Queue with reason; do not fill [TO FILL] cells in Event Strength pre-draft

**Step 3 — Fill Event Strength Pre-Draft [TO FILL] Cells (10 min)**
- Open `Rankings/Event Strength Phase 1 — Authorization Request (Pre-Draft).md`
- Fill: G0 confirmation date (Section 2, Condition 1)
- Fill: session date for G2 gate reference (Section 2, Condition 2)
- Do NOT submit to SENTINEL yet — G2 and stability checks still pending
- Note in document: "G0 confirmed [date]; G2 pending; stability 3-run check pending"

**Step 4 — Fill stddev Baseline (10 min)**
- Run the psql queries in `Rankings/May 1 Pipeline Launch — ELO Ratings Baseline.md` Section 6
- Fill Section 2 (baseline stddev values by age group)
- Remove the "If no April 29 session" exception language from Section 5 — replace with confirmed actuals
- Mark Section 5 certification exception (2) as RESOLVED

**Step 5 — Initiate Boys Option A (G0-independent) (5 min)**
- Confirm B-OA-1/2/3 queries are ready in `Rankings/Boys Brier — Pre-May-17 Execution Package.md` (they are)
- Notify Presten: "B-OA-1/2/3 are ready to run. Execution package at [file path]. This is G0-independent."
- If Presten runs queries in same session: ELO fills `Rankings/Boys Calibration — Option A Verdict Document.md` within 30 min of results

**Step 6 — ECNL CP1 Execution Signal (5 min)**
- Confirm ECNL ecnl_verified ALTER TABLE + backfill ran successfully
- If yes: notify FORGE that CP1 data is now available; FORGE proceeds per ECNL migration scope
- Record CP1 readiness in `Rankings/ECNL CP1-CP2 Checkpoint Results.md` (or file if not yet existing)

**Step 7 — Briefing Update (15 min)**
- File next ELO briefing covering all 6 steps above
- Update task status table for all session-unblocked items
- Notify SENTINEL: G0 status, stddev fill status, Boys Option A status

### Section 2 — If Session Does Not Open by April 30 EOD

ELO must:
1. Escalate R1 (GA ASPIRE) from High to Critical in the May 9 DSS Risk Register
2. File a Queue item to SENTINEL: "April 29 session Day X — Event Strength Phase 1 authorization window compressing toward May 7 deadline"
3. Confirm May 1 monitoring plan proceeds with pre-fix baseline (as documented in Baseline Section 5)

## Notes

- This checklist is designed for the session executing without SENTINEL present — ELO executes, then briefs SENTINEL in the next cycle
- Do not wait for SENTINEL to confirm each step — execute the checklist and report all results in one briefing
- The 60-minute target is aggressive but achievable; all queries are pre-staged
