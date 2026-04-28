---
type: elo-checkpoint-results
subject: ECNL Migration CP1 + CP2
session_date: 2026-04-29
status: pending
option1-status: pending
sentinel-action-required: none
tags: [elo, ecnl, migration, checkpoint, cp1, cp2, april29]
---

# ECNL Migration — CP1/CP2 Checkpoint Results

> **Template filed 2026-04-28. Fill in during the April 29 session within 30 minutes of running both checkpoints. Update `status` in frontmatter when complete.**

---

## CP1 — ECNL RL Data Staleness

- **Checkpoint:** ECNL RL most recent game must be within 30 days of April 29
- **Query result:** Most recent ECNL RL game date = ___________
- **Days elapsed:** ___________ days
- **Threshold:** ≤ 30 days
- **CP1 Verdict:** PASS / FAIL

**If FAIL:**
> CP1 FAIL means Option 1 (No Re-tag) is disqualified. ECNL RL data is too stale for ELO to assert the Path A/B distinction has current signal. ELO's recommendation reverts to Option 2 or Option 3 pending SENTINEL and Presten review.
> Action: ELO notifies SENTINEL via queue (priority: urgent) within 30 minutes.

**If PASS:**
> Option 1 remains eligible. Proceed to CP2.

---

## CP2 — ecnl_verified Backfill Coverage

- **Checkpoint:** Teams with ecnl_verified = true must represent ≥ 70% of ECNL-eligible teams
- **Query result:** ecnl_verified = true: ___________ teams / ___________ total ECNL-eligible teams = ___________%
- **Threshold:** ≥ 70%
- **CP2 Verdict:** PASS / FAIL

**If FAIL:**
> CP2 FAIL means the ecnl_verified field cannot be relied upon to distinguish Path A from Path B teams at the required coverage level. Option 1's no-re-tag approach becomes higher risk. ELO flags to SENTINEL (priority: high) but does not automatically disqualify Option 1 unless SENTINEL directs.
> Note: FORGE's ecnl_verified backfill SQL package ran on April 28 — confirm whether Step 2b of the Execution Log completed.

**If PASS:**
> Option 1 coverage threshold met. Both CPs passed.

---

## Combined Verdict

| CP | Result | Option 1 Impact |
|----|--------|----------------|
| CP1 | PASS / FAIL | None / Disqualified |
| CP2 | PASS / FAIL | None / Elevated risk |

**Overall:** Option 1 status = ON-TRACK / DISQUALIFIED / ON-TRACK WITH ELEVATED RISK

---

## SENTINEL Action Required

State exactly what SENTINEL should do based on the combined result:

- **Both PASS:** No action — Option 1 authorized path unchanged.
- **CP1 FAIL:** Urgent review required — Option 1 disqualified. Revise authorization. ELO reverts to Option 2 per ECNL Migration — Option Comparison Matrix.
- **CP2 FAIL only:** Review elevated risk position. Confirm Option 1 authorization stands or redirect to Option 2.

**SENTINEL action:** ___________

---

## References

- `Rankings/ECNL Migration — ELO Option Recommendation.md` — Option 1 recommendation this document gates
- `Rankings/ECNL Migration — Option Comparison Matrix.md` — fallback options if CP1 disqualifies Option 1
- `Rankings/ECNL Migration — CP1 Fail Escalation Protocol.md` — escalation steps if CP1 fails
- `Infrastructure/April 28 Execution Log.md` — Step 2b confirms ecnl_verified backfill ran
- `task-2026-04-28-ecnl-cp1-cp2-checkpoint-results-document.md` — source task

*ELO — template filed 2026-04-28*
