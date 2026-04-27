---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-27
status: completed
completed: 2026-04-27
priority: medium
due: 2026-05-04
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Pre-Verdict Evidence Checklist.md"
---

# Task: Boys Option A — Pre-Verdict Evidence Checklist

## Context

Boys Option A spot check results arrive between April 29 and May 5. ELO has the Decision Document template (pre-written PASS and FAIL branches) and the Query Pack Validation Note (confirmed aligned, cleared April 27). What ELO does not have is a formal checklist that must be completed before the verdict is filed.

The risk: ELO runs the 3 queries on April 29, sees results that look like a PASS or FAIL, and files the Decision Document verdict on the same day — before the spot check window is complete (window closes May 5). A 1-day verdict is not a spot check; it is a single-session result. The pre-verdict evidence checklist enforces the rigor that the "spot check" framing implies.

SENTINEL's specific concern: Option A PASS authorizes deploying Boys calibration changes post-DSS. These changes affect Boys rankings across all age groups. The authorization should be based on all available evidence within the window, not the first available query result.

## What to Produce

File: `02 - Tiger Tournaments/Projects/Rankings/Boys Option A — Pre-Verdict Evidence Checklist.md`

---

### Section 1: Evidence Completion Gate

ELO completes this section before filing the Boys Option A Decision Document verdict.

**Required before verdict:**

- [ ] Query 1 (Boys GA average rating by age group) executed by Presten and results received by ELO
- [ ] Query 2 (Boys GA ASPIRE vs. non-ASPIRE rating comparison) executed and results received
- [ ] Query 3 (Boys GA cross-age-group tier consistency) executed and results received
- [ ] Query 1 pivot completed (ELO separates M/F rows and calculates delta) — confirmed in Query Pack Validation Note
- [ ] Spot check window has been open for ≥ 5 calendar days OR Presten has explicitly confirmed the window is closed

**If any of the first four items are not complete:** ELO does not file the verdict. ELO files an interim note in the Decision Document: "Evidence collection in progress. Expected completion: [date]. Verdict deferred."

**If Presten closes the window early:** ELO may file with available evidence and notes in the verdict document: "Window closed early per Presten instruction on [date]. Verdict based on [N of 3 queries] available."

---

### Section 2: Data Quality Check

Before interpreting query results, ELO confirms:

**Q1 quality:**
- [ ] Boys age groups present in Q1 output (at least U13B, U14B, U15B, U16B, U17B)
- [ ] Result count is non-zero for Boys (gender='M') rows
- [ ] Result count is non-zero for Girls (gender='F') rows (needed for the delta calculation)

**Q2 quality:**
- [ ] `event_tier IN ('ga','ga_aspire')` filter returned > 0 Boys games — if zero, GA Boys exposure is minimal and Option A PASS/FAIL is a trivial result. Note this explicitly.

**Q3 quality:**
- [ ] Q3 returned rows across ≥ 3 age groups — single-age-group results are insufficient for cross-tier consistency assessment

If any quality check fails (zero rows in a critical query), ELO does not file PASS or FAIL. ELO files: "Verdict indeterminate — insufficient Boys GA game volume for meaningful calibration assessment. Escalate to SENTINEL."

---

### Section 3: Verdict Criteria (Pre-Commitment)

ELO states, before running queries, what specific thresholds it will use to define PASS vs. FAIL. These criteria are set here, not after results arrive.

**PASS criteria (ELO fills based on Boys Option A Decision Document):**

State the specific numeric threshold for PASS. Example structure (ELO writes the actual criteria):
- Boys GA teams in the top 3 quartiles have average ratings within [N] points of Girls GA teams in the same age group
- OR Boys GA cross-tier consistency matches the expected pattern: Tier 1 ECNL/MLS NEXT > Tier 2 GA > Tier 3 State
- AND no anomaly flag from Q3 (Q3 anomaly definition from Decision Document)

**FAIL criteria:**
- Results do not meet PASS threshold AND ELO's assessment is that Boys calibration differs materially from Girls calibration structure

**Indeterminate:**
- Insufficient data (Section 2 quality check failure)
- Results are within PASS threshold for some age groups but not others — requires further investigation before applying Option A parameters

ELO should write these criteria in plain terms. If ELO cannot write specific thresholds before seeing the data, that itself is a flag — calibration decisions should not be made without pre-specified criteria.

---

### Section 4: Verdict Filing Protocol

When ELO is ready to file the verdict:

1. Complete all items in Section 1 (evidence gate)
2. Complete all items in Section 2 (data quality check)
3. Confirm Section 3 criteria were written before query results were seen (check file creation date of this document vs. verdict filing date)
4. Fill Boys Option A Decision Document: select PASS or FAIL branch, fill in specific query values
5. In the Decision Document's verdict section, include one line: "Pre-Verdict Evidence Checklist completed: [date]. All required queries executed. Quality checks: passed / [specific failure]."
6. File ELO briefing: "Boys Option A verdict: PASS / FAIL / Indeterminate. Key evidence: [1–2 sentences from Q1 pivot result]. Post-DSS implication: [what this means for the post-DSS calibration roadmap]."

---

## Why This Matters

Option A PASS unlocks Boys calibration changes that affect all Boys rankings. A verdict filed before the window closes, based on a single session's data, does not meet the rigor implied by calling it a "spot check." More concretely: Boys rankings are less well-understood than Girls rankings (the entire Option A process exists because Boys calibration gaps were identified), which means there is higher prior probability of anomalous results that require more than one data point to interpret. The pre-verdict checklist doesn't slow down a clear PASS — it prevents a borderline result from being filed as PASS before ELO has enough data to know.

## Definition of Done

- File at deliverable path by May 4 (day before window closes)
- Sections 1–4 present
- Section 3 PASS criteria written with specific numeric thresholds (not "looks good")
- ELO references this checklist when filing the Boys Option A Decision Document verdict
- FORGE briefing note: "Boys Option A pre-verdict checklist filed. PASS threshold: [summary of Section 3 criteria]. Window closes: May 5."

## References

- `Rankings/Boys Option A — Decision Document.md` — the verdict document this checklist gates
- `Rankings/Boys Option A — Query Pack Validation Note.md` — confirms query-to-document alignment
- `task-2026-04-26-boys-option-a-spot-check-execution.md` — the execution task this checklist gates
- `Rankings/Post-DSS Calibration Roadmap — June 2026.md` — the downstream document that depends on this verdict
