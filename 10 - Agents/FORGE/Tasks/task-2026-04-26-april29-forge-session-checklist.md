---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-26
status: pending
priority: urgent
due: 2026-04-29
deliverable: "02 - Tiger Tournaments/Projects/Infrastructure/April 29 FORGE Session Checklist.md"
---

# Task: April 29 FORGE Session Checklist

## Context

FORGE's April 29 actions are as dense as ELO's, but they are scattered across briefing text rather than consolidated in a single executable document. FORGE must:

1. Confirm the April 28 execution log is filed and read SENTINEL's disposition
2. Run 3 Source Baseline Section 4 queries and update the baseline document
3. If FM1/FM2 results have landed — execute audit document updates per the Contingency Action Plan
4. If FM1/FM2 results have NOT landed — file an escalation Queue item to SENTINEL flagging the May 1 gate risk
5. Enter Row 1 in the First-Week Pipeline Monitoring Log (May 1 is 2 days away — FORGE pre-fills the template fields they can fill before May 1 runs)
6. Coordinate with ELO: confirm CP1 staleness result was filed and read it. If ECNL RL > 30 days, flag immediately to SENTINEL.

Without a session checklist, FORGE risks completing step 3 before confirming step 1's prerequisite (April 28 must be confirmed complete before FORGE updates audit docs against it), or skipping step 6 because it's ELO's action in briefing text.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Infrastructure/April 29 FORGE Session Checklist.md`

This is a single-use linear checklist. Presten or FORGE opens it at the start of the April 29 FORGE session and works top to bottom. Every step has a prerequisite, the action, and where to file the output.

---

### Header

```
Session Date: April 29, 2026
Prerequisite: April 28 execution log confirmed filed (check Presten notes or Execution Log file)
Total estimated time: ~45 minutes for steps 1–4. Steps 5–6 are <10 minutes each.
```

---

### Step 1: Confirm April 28 Execution Log (5 min)

- Open `Infrastructure/April 28 Execution Log.md`
- Verify all 3 Step ✅ boxes are filled
- Read SENTINEL Disposition section — is Phase 1 authorized or held?
- If HELD: stop here, escalate to SENTINEL before proceeding to Step 2
- **Prerequisite for Step 2:** All 3 steps confirmed ✅

---

### Step 2: Source Baseline Section 4 — Run 3 Queries (15 min)

Reference: `task-2026-04-26-post-april28-source-baseline-section4-update`

Run in this order:

**Query A — GA ASPIRE event_tier verification:**
Confirm that GA ASPIRE events now have `event_tier = 'ga_aspire'` (not `'ga'`). Count of rows where `event_tier = 'ga'` and event name contains 'ASPIRE' should be 0 after the fix.

**Query B — ecnl_verified column confirmation:**
Confirm column exists in games schema (`\d games` or `SELECT column_name FROM information_schema.columns WHERE table_name = 'games' AND column_name = 'ecnl_verified'`). Confirm default is FALSE.

**Query C — Post-recompute game count delta:**
Compare total rated games count to Pre-April-28 Baseline (Section 1 anchor). File: total games now, delta, and whether new games are above or below the Section 1 anchor.

File all three results in `Infrastructure/Source Ingestion Baseline Snapshot — Pre-Launch April 2026.md` → Section 4. Mark `task-2026-04-26-post-april28-source-baseline-section4-update` as completed.

---

### Step 3: FM1/FM2 Audit Document Updates (10 min if results in; skip if not)

- Check `Infrastructure/Pipeline Code Review Results.md`
- If path letters are filed: open `Infrastructure/FM1-FM2 Contingency Action Plan.md`, locate the matching row for FM1×FM2 paths, execute that row's action
- If path letters are NOT filed: file a Queue item at `10 - Agents/SENTINEL/Queue/pending-2026-04-29-fm1-fm2-still-unresolved.md` — flag that FM1/FM2 results are still missing and May 1 deployment confidence gate is not met. SENTINEL then escalates to Presten.

---

### Step 4: First-Week Monitoring Log — Pre-Population (10 min)

File: `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md`

Pre-fill Row 1 (May 1) with:
- Date: 2026-05-01
- Status: [leave blank — fill after May 1 run]
- Expected game count: [pull from Morning-After Runbook GREEN threshold]
- Notes: "First production run. GREEN/YELLOW/RED to be filled post-run."

Write one note in the Issues Log: "May 1 Row 1 — monitoring begins. No issues to report pre-run."

---

### Step 5: ELO Coordination — CP1 Staleness Check (5 min)

- Read ELO's April 29 briefing (check `10 - Agents/ELO/Briefings/` for the April 29 entry)
- Confirm ELO filed CP1 result from ECNL RL staleness check
- If CP1 = PASS (RL < 30 days): no FORGE action
- If CP1 = FAIL (RL ≥ 30 days): file a FORGE Queue item to SENTINEL — "CP1 FAIL: ECNL Option 1 disqualified. ECNL migration must escalate. No FORGE config change for ECNL RL source until ELO files Option 2 selection."

---

### Checklist Summary

| Step | Time | Prerequisite | File Output |
|------|------|-------------|-------------|
| 1: Confirm Execution Log | 5 min | April 28 session ran | None — gate check |
| 2: Source Baseline Section 4 | 15 min | Step 1 ✅ | Source Ingestion Baseline — Section 4 |
| 3: FM1/FM2 Audit Updates | 10 min | Path letters in Code Review Results | Audit docs OR Queue escalation |
| 4: Pre-populate Monitoring Log | 10 min | None | First-Week Monitoring Log Row 1 |
| 5: ELO CP1 coordination | 5 min | ELO April 29 briefing filed | Queue item if CP1 fails |

Total: ~45 minutes.

---

## Quality Standard

- Every step has an explicit prerequisite and a "what to do if prerequisite fails" branch
- No step ends in "investigate" — each has a defined output location
- Steps 3 and 5 each have a "results not in" branch that files a Queue escalation rather than leaving FORGE blocked silently

## Why This Matters

April 29 is a coordination-heavy day. ELO is running 8 analysis tasks simultaneously. FORGE's actions are gate inputs for ELO (Source Baseline Section 4) and SENTINEL (FM1/FM2 audit closure). If FORGE executes these out of order or skips the ELO coordination step, the April 29 gate reviews will be incomplete. This checklist costs 2 minutes to write; it prevents a 2-hour confusion session.

## References

- `Infrastructure/April 28 Execution Log.md`
- `Infrastructure/Source Ingestion Baseline Snapshot — Pre-Launch April 2026.md`
- `Infrastructure/Pipeline Code Review Results.md`
- `Infrastructure/FM1-FM2 Contingency Action Plan.md`
- `Infrastructure/First-Week Pipeline Monitoring Log — May 1–7.md`
- `task-2026-04-26-post-april28-source-baseline-section4-update`
