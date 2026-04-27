---
title: Boys Option A — Pre-Verdict Evidence Checklist
type: evidence-checklist
author: ELO
date: 2026-04-27
status: active — ELO completes before filing Boys Option A verdict
related: "[[Boys Option A — Decision Document]]", "[[Boys Option A — Query Pack Validation Note]]", "[[Post-DSS Calibration Roadmap — June 2026]]"
tags: [boys, calibration, club-rankings, option-a, checklist, evo-draw]
---

# Boys Option A — Pre-Verdict Evidence Checklist

> **Purpose:** ELO completes this checklist before filing the verdict in `Boys Option A — Decision Document.md`. Written April 27 — before query results are seen — so PASS/FAIL criteria are committed in advance.
>
> **Spot check window:** April 29 – May 5, 2026.
> **Verdict must not be filed before:** May 5 OR explicit Presten instruction to close early.

---

## Section 1: Evidence Completion Gate

ELO completes all items before filing the verdict in the Decision Document.

**Required before verdict:**

- [ ] Query 1 (Boys vs. Girls rating divergence by age group) executed by Presten and results received by ELO
- [ ] Query 2 (Boys GA game volume by age group) executed and results received
- [ ] Query 3 (MLS NEXT Boys rating distribution) executed and results received
- [ ] Query 1 pivot confirmed complete — ELO separated M/F rows and calculated per-age-group delta (confirmed aligned in Query Pack Validation Note, 2026-04-27)
- [ ] Spot check window has been open ≥ 5 calendar days **OR** Presten has explicitly confirmed the window is closed

**If any of the first four items are not complete:** ELO does not file the verdict. ELO files an interim note in the Decision Document:

> "Evidence collection in progress. Expected completion: [date]. Verdict deferred."

**If Presten closes the window early:** ELO may file with available evidence and notes in the verdict:

> "Window closed early per Presten instruction on [date]. Verdict based on [N of 3 queries] available."

---

## Section 2: Data Quality Check

Before interpreting any query results, ELO confirms:

**Q1 quality:**
- [ ] Boys age groups U13B, U14B, U15B, U16B, U17B all present in Q1 output
- [ ] Non-zero result count for Boys (gender='M') rows
- [ ] Non-zero result count for Girls (gender='F') rows (required for delta calculation)

**Q2 quality:**
- [ ] `event_tier IN ('ga', 'ga_aspire')` filter returned > 0 Boys games. If zero, Boys GA exposure is minimal — note explicitly: "Boys GA game volume insufficient for meaningful Option A calibration assessment. Q2 result is ADEQUATE / LOW for [age group]."

**Q3 quality:**
- [ ] Q3 returned rows across ≥ 3 age groups. Single-age-group result is insufficient for cross-tier consistency assessment.

**If any quality check fails (zero rows in a critical query):** ELO does not file PASS or FAIL. ELO files:

> "Verdict indeterminate — insufficient Boys GA game volume for meaningful calibration assessment. Escalating to SENTINEL."

---

## Section 3: Verdict Criteria (Pre-Committed — Written April 27)

These thresholds are fixed before query results are seen. ELO does not modify them after results arrive.

**PASS criteria:**

1. **Q1:** No age group shows |Boys avg rating − Girls avg rating| > **100 points**. All age groups at ≤75 pts divergence = clean PASS. Any age group at 76–100 pts = yellow flag requiring ELO narrative explanation in Decision Document Section 3.

2. **Q2:** All five age groups (U13B–U17B) show ≥ **500 Boys GA games** in DB. Age groups below 500 are flagged ADEQUATE/LOW and their Q1 divergence result is treated as lower-confidence.

3. **Q3:** For each age group with ≥ 10 MLS NEXT Boys teams:
   - Median rating: **1,400–1,600**
   - Top 10th percentile: **≥ 1,700**
   - Bottom 10th percentile: **≤ 1,300**
   - Hard FAIL: any metric outside these ranges by > 100 pts.

4. **No active Boys calibration instability flag** from SENTINEL or from the April 28 GA ASPIRE fix (Boys should have 0 ± 5 pts change from that fix — confirmed in April 29 G4 gate).

**FAIL criteria:**

Results do not meet Q1 PASS threshold (any age group divergence > 100 pts) AND ELO's narrative assessment concludes Boys calibration differs materially from Girls calibration structure in a way not explained by structural Boys/Girls differences.

**CONDITIONAL PROCEED:**

Q1/Q2/Q3 all pass thresholds, but ELO identifies a specific named concern (e.g., thin game volume in a particular age group, or borderline divergence at 80–100 pts for one age group) that warrants a caveat in the Authorization Request. ELO brings this to SENTINEL before Boys implementation begins.

**INDETERMINATE:**

Data quality check failure in Section 2 (zero rows in Q1 or Q2 for a critical age group). Does not produce PASS or FAIL. Requires SENTINEL escalation.

---

## Section 4: Verdict Filing Protocol

When ELO is ready to file:

1. Complete all items in Section 1 (evidence gate) ✅
2. Complete all items in Section 2 (data quality check) ✅
3. Confirm Section 3 criteria were written before results arrived — check this document's creation date (2026-04-27) vs. verdict filing date ✅
4. Fill `Boys Option A — Decision Document.md`: paste Q1/Q2/Q3 results into Section 1 tables, write Section 3 analysis (two paragraphs), select PROCEED / ACTIVATE FALLBACK / CONDITIONAL in Section 4.
5. In the Decision Document's verdict section, include: "Pre-Verdict Evidence Checklist completed: [date]. All required queries executed. Quality checks: passed / [specific failure if any]."
6. File ELO briefing note: "Boys Option A verdict: PASS / FAIL / INDETERMINATE. Key evidence: [1–2 sentences from Q1 pivot result]. Post-DSS implication: [what this means for the Post-DSS Calibration Roadmap — Option A or Option B path]."

---

## References

- `Rankings/Boys Option A — Decision Document.md` — verdict document this checklist gates
- `Rankings/Boys Option A — Query Pack Validation Note.md` — confirms queries are aligned with Decision Document tables (cleared 2026-04-27)
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — the 3 queries Presten runs
- `Rankings/Post-DSS Calibration Roadmap — June 2026.md` — Option A or B path depends on this verdict
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — activated on FAIL

*ELO — 2026-04-27*
