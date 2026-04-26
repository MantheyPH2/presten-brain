---
title: Boys Option A — Decision Document
type: decision-record
author: ELO
date: 2026-04-26
status: awaiting-results
related: "[[Boys Option A Spot Check — Presten Execution Package]]", "[[Club Rankings — Girls-Only Fallback Spec]]", "[[Boys Calibration Status Summary — April 2026]]"
tags: [boys, calibration, club-rankings, dss, decision, evo-draw]
---

# Boys Option A — Decision Document

> **Purpose:** Structured decision record ELO fills when Boys Option A spot check results arrive from Presten. This is the single source of truth for the Boys Club Rankings go/no-go decision. SENTINEL authorizes or holds Boys Club Rankings based on this document.
>
> **Expected fill date:** April 29 – May 5 (after Presten runs spot check queries).
> **Decision gate:** May 9 DSS readiness check.

---

## Section 1: Spot Check Results

ELO fills this section using the three output files Presten sends after running `Boys Option A Spot Check — Presten Execution Package.md`.

---

### Query 1 — Boys vs. Girls Rating Divergence by Age Group

Source file: `Reports/boys-option-a-query1-[date].md`

| Age Group | Boys Avg Rating | Girls Avg Rating | Delta (Boys − Girls) | Expected Delta | Pass? |
|-----------|-----------------|------------------|----------------------|----------------|-------|
| U13 | | | | ≤ 75 pts | |
| U14 | | | | ≤ 75 pts | |
| U15 | | | | ≤ 75 pts | |
| U16 | | | | ≤ 75 pts | |
| U17 | | | | ≤ 75 pts | |

**Expected delta rationale:** Boys and Girls share the same Glicko-2 parameters and league calibration structure. The structural expectation is ≤75 points average difference across age groups — leagues like MLS NEXT and ECNL operate in separate Boys/Girls pools with comparable calibration values. The hard FAIL threshold per the execution package is >100 points divergence in any single age group. 76–100 points is a yellow flag requiring ELO narrative explanation in Section 3.

**Pass criteria for Query 1:** No age group shows |Boys avg − Girls avg| > 100 pts.

---

### Query 2 — Boys GA Game Volume by Age Group

Source file: `Reports/boys-option-a-query2-[date].md`

| Age Group | Boys GA Games in DB | Minimum (500) | Sufficient? |
|-----------|---------------------|---------------|-------------|
| U13B | | ≥ 500 | |
| U14B | | ≥ 500 | |
| U15B | | ≥ 500 | |
| U16B | | ≥ 500 | |
| U17B | | ≥ 500 | |

**Pass criteria for Query 2:** All U13B–U17B age groups have ≥ 500 Boys GA games.

---

### Query 3 — MLS NEXT Boys Rating Distribution

Source file: `Reports/boys-option-a-query3-[date].md`

| Age Group | MLS NEXT Team Count | Avg Rating | Median | Top 10% | Bottom 10% | Max | Pass? |
|-----------|---------------------|-----------|--------|---------|------------|-----|-------|
| U13B | | | | | | | |
| U14B | | | | | | | |
| U15B | | | | | | | |
| U16B | | | | | | | |
| U17B | | | | | | | |

**Pass criteria for Query 3 (age groups with ≥ 10 MLS NEXT teams):**
- Median rating: 1,400–1,600
- Top 10th percentile: ≥ 1,700
- Bottom 10th percentile: ≤ 1,300
- FAIL: any metric outside range by > 100 pts → contact ELO for immediate assessment.

---

## Section 2: Pass/Fail Assessment

| Check | Result | Pass? |
|-------|--------|-------|
| Query 1: No age group divergence > 100 pts | | |
| Query 2: All U13B–U17B have ≥ 500 Boys GA games | | |
| Query 3: MLS NEXT distribution within expected range for age groups with ≥ 10 teams | | |
| No active Boys calibration instability flag from SENTINEL or GA ASPIRE fix | | |

**Overall Boys Option A Result:** PASS / FAIL / CONDITIONAL

---

## Section 3: ELO Analysis

*(Two paragraphs — ELO fills after reviewing all three query outputs. Not a bullet list. ELO is making a calibration judgment, not checking boxes.)*

**Paragraph 1 — What do the Boys Option A results actually show?**

[ELO: describe the Boys rating distribution from Queries 1–3. Where does it look strong (MLS NEXT anchor, game volume coverage)? Where does it look uncertain (thin age groups, GA volume below threshold, any unexpected divergence)? Include specific numbers from the query results.]

**Paragraph 2 — Is ELO confident enough in Boys ratings to feature Boys Club Rankings at DSS?**

[ELO: state a confidence level (high / medium / low) and the primary reason. Reference the Boys Calibration Status Summary context: Boys calibration design is structurally sound but unvalidated by Brier analysis. The Option A spot check is a 20-minute empirical check against theoretical expectation. State whether the results confirm the structural assumption or reveal a discrepancy that needs investigation.]

---

## Section 4: Decision and Downstream Actions

| Decision | Trigger | Actions |
|----------|---------|---------|
| PROCEED with Boys Club Rankings | All 4 checks PASS + ELO confidence medium or high | File this document. Reference it in `Rankings/Club Rankings — Boys-Conditional Implementation Timeline.md`. Update DSS Feature Readiness Tracker Boys cell to ✅. Notify SENTINEL. |
| ACTIVATE Girls-Only Fallback | Any check FAIL OR ELO confidence low | File this document. Reference `Rankings/Club Rankings — Girls-Only Fallback Spec.md`. Alert FORGE immediately. Update DSS Feature Readiness Tracker Boys cell to ❌. File updated briefing. |
| CONDITIONAL PROCEED | Checks pass but ELO confidence medium with a specific named caveat | File this document with caveat stated. Bring to SENTINEL with recommendation and reason before authorizing Boys. Do not start Boys implementation until SENTINEL disposition filled. |

**ELO selection:** PROCEED / ACTIVATE FALLBACK / CONDITIONAL PROCEED

**Reason (one paragraph):**

[ELO fills]

---

## Section 5: SENTINEL Authorization Request

```
Boys Club Rankings Decision
Date: [date]
From: ELO
To: SENTINEL

Option A Result: PASS / FAIL / CONDITIONAL
ELO Decision: PROCEED / ACTIVATE FALLBACK / CONDITIONAL
Confidence: HIGH / MEDIUM / LOW

If PROCEED:
  ELO is ready to include Boys Club Rankings in the implementation plan.
  Boys age groups cleared: [list — typically U13B–U17B if all pass]
  Any age groups excluded (Q2 thin coverage): [list or "none"]
  Reference: this document, filed [date].

If ACTIVATE FALLBACK:
  Girls-Only Fallback Spec activated: `Rankings/Club Rankings — Girls-Only Fallback Spec.md`
  FORGE notified: [date]
  DSS Feature Readiness Tracker Boys cell: ❌

If CONDITIONAL:
  [Specific condition ELO is flagging for SENTINEL judgment — one sentence.]
  ELO recommendation: [proceed with caveat / hold pending further check]

SENTINEL signature required before Club Rankings implementation starts.
```

---

## SENTINEL Disposition

> SENTINEL fills — do not pre-fill.

```
Decision document reviewed: [ ]
ELO verdict accepted: [ ]
Boys Club Rankings: AUTHORIZED / HELD / GIRLS-ONLY
If AUTHORIZED: FORGE session may begin ___________
If HELD: reason: ___________
DSS Feature Readiness Tracker to be updated by ELO: [ ]
SENTINEL sign-off: ___________
```

---

## Why This Matters

Club Rankings Boys is a binary DSS demo decision — there is not time to pull Boys from the demo after FORGE implementation begins. ELO needs to make this call based on data, and the call needs to be auditable. If Boys Club Rankings succeed at DSS, this document shows the evidence that supported the decision. If Boys Club Rankings fail at DSS, SENTINEL can show Presten exactly when and why Boys were approved. Either way, the decision is traceable and not reconstructed from memory.

---

## References

- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — source of SQL output ELO interprets here
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys baseline and Option A context
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — activated if FAIL
- `Rankings/Club Rankings — Boys-Conditional Implementation Timeline.md` — timeline gated by this decision
- `Rankings/Club Rankings — Implementation Specification.md` — full implementation (Boys + Girls)
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Boys cell to update after decision

*ELO — 2026-04-26*
