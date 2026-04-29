---
type: elo-verdict-document
topic: boys-option-a
date: 2026-04-29
status: shell-complete — awaiting query results
queries: [B-OA-1, B-OA-2, B-OA-3]
expected_results_window: "2026-04-29 to 2026-05-05"
filing_target: "< 30 minutes from results receipt"
tags: [boys, calibration, club-rankings, dss, verdict, elo, evo-draw]
---

# Boys Calibration — Option A Verdict Document

> **Shell status:** All static content complete. All pass thresholds pre-filled. Cells marked [TO FILL] require Presten's query results (B-OA-1, B-OA-2, B-OA-3). When results arrive, ELO inserts actuals and writes analysis. Filing time < 30 minutes from results receipt.

---

## Section 1 — Document Purpose

This document records the verdict on whether Boys Option A calibration parameters produce a rating distribution acceptable for DSS presentation and Boys Club Rankings inclusion. When Presten runs queries B-OA-1, B-OA-2, and B-OA-3, ELO interprets the results against pre-defined thresholds, reaches a verdict (APPROVE / CONDITIONAL / REJECT), and files this document. SENTINEL uses this document to authorize or hold Boys Club Rankings implementation. If the verdict is REJECT, the Girls-Only Fallback Spec activates immediately. All Boys DSS and Club Rankings decisions trace to this document.

---

## Section 2 — Option A Parameters Being Evaluated

These parameters are drawn from `Rankings/Boys Option A Spot Check — Presten Execution Package.md` and `Rankings/Boys Option A — Decision Document.md`. Do not re-derive.

**What "Option A" means:** A targeted spot-check of Boys rating calibration using three diagnostic queries to confirm the Boys rating distribution is structurally sound before authorizing Boys Club Rankings for DSS. Option A does not change calibration values — it validates that the existing Boys parameters (MLS NEXT = 160, ECNL Boys = 120, GA Boys = 100) are producing ratings within expected ranges.

| Parameter | Value | Source |
|-----------|-------|--------|
| Boys MLS NEXT calibration value | 160 (Academy/Homegrown split pending May 18) | League Hierarchy |
| Boys ECNL calibration value | 120 | League Hierarchy |
| Boys GA calibration value | 100 | League Hierarchy |
| Target age groups | U13B, U14B, U15B, U16B, U17B | Boys Option A scope |
| Pass threshold — rating divergence (B-OA-1) | Boys − Girls avg ≤ 75 pts per age group | Decision Document |
| Hard FAIL threshold — divergence (B-OA-1) | Boys − Girls avg > 100 pts in any age group | Decision Document |
| Pass threshold — game volume (B-OA-2) | ≥ 500 Boys GA games per U13B–U17B age group | Decision Document |
| Pass threshold — MLS NEXT distribution (B-OA-3) | Median 1,400–1,600; top 10% ≥ 1,700; bottom 10% ≤ 1,300 | Decision Document |

---

## Section 3 — Query Results

### B-OA-1: Boys vs. Girls Rating Divergence by Age Group

| Age Group | Boys Avg Rating | Girls Avg Rating | Delta (Boys − Girls) | Pass Threshold | Pass? |
|-----------|-----------------|------------------|----------------------|----------------|-------|
| U13 | [TO FILL: B-OA-1] | [TO FILL: B-OA-1] | [TO FILL] | ≤ 75 pts | [TO FILL] |
| U14 | [TO FILL: B-OA-1] | [TO FILL: B-OA-1] | [TO FILL] | ≤ 75 pts | [TO FILL] |
| U15 | [TO FILL: B-OA-1] | [TO FILL: B-OA-1] | [TO FILL] | ≤ 75 pts | [TO FILL] |
| U16 | [TO FILL: B-OA-1] | [TO FILL: B-OA-1] | [TO FILL] | ≤ 75 pts | [TO FILL] |
| U17 | [TO FILL: B-OA-1] | [TO FILL: B-OA-1] | [TO FILL] | ≤ 75 pts | [TO FILL] |

**Hard FAIL trigger:** Any single age group delta > 100 pts → Boys Club Rankings SUSPENDED.

### B-OA-2: Boys GA Game Volume by Age Group

| Age Group | Boys GA Games in DB | Pass Threshold | Sufficient? |
|-----------|---------------------|----------------|-------------|
| U13B | [TO FILL: B-OA-2] | ≥ 500 | [TO FILL] |
| U14B | [TO FILL: B-OA-2] | ≥ 500 | [TO FILL] |
| U15B | [TO FILL: B-OA-2] | ≥ 500 | [TO FILL] |
| U16B | [TO FILL: B-OA-2] | ≥ 500 | [TO FILL] |
| U17B | [TO FILL: B-OA-2] | ≥ 500 | [TO FILL] |

**Note:** Age groups below 500 Boys GA games produce volatile ratings with higher uncertainty. If any age group is below threshold, Boys rankings for that age group carry a coverage caveat.

### B-OA-3: MLS NEXT Boys Rating Distribution (for age groups with ≥ 10 MLS NEXT teams)

| Age Group | MLS NEXT Team Count | Avg Rating | Median | Top 10% | Bottom 10% | Max | Pass? |
|-----------|---------------------|------------|--------|---------|------------|-----|-------|
| U13B | [TO FILL: B-OA-3] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] |
| U14B | [TO FILL: B-OA-3] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] |
| U15B | [TO FILL: B-OA-3] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] |
| U16B | [TO FILL: B-OA-3] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] |
| U17B | [TO FILL: B-OA-3] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] | [TO FILL] |

**Pass criteria (for age groups with ≥ 10 MLS NEXT teams):**
- Median: 1,400–1,600
- Top 10th percentile: ≥ 1,700
- Bottom 10th percentile: ≤ 1,300
- FAIL: any metric outside range by > 100 pts

---

## Section 4 — Analysis

*ELO writes this section after reviewing all query results. This is calibration judgment, not box-checking.*

**Rating distribution health (B-OA-1):**
> [TO FILL: Interpretation of B-OA-1 distribution. For each age group: is the delta within expected range? Note that some Boys − Girls divergence is structurally expected (Boys MLS NEXT = 160 vs Girls GA = 140) — divergence ≤ 75 pts confirms this structural expectation is holding. Flag any age group where the delta exceeds 75 pts and explain whether this reflects coverage imbalance or systematic miscalibration. Reference: Boys calibration design runs through MLS NEXT (160) as Boys top tier; Girls top tier runs through GA (140) and ECNL (140). A moderate Boys < Girls gap at U13/U14 is consistent with Boys GA (100) being lower than Girls GA (140) in early age groups.]

**Game volume assessment (B-OA-2):**
> [TO FILL: Which age groups cleared the 500-game threshold? If any age group is below threshold, state: what the coverage gap means for those age groups' ratings (higher uncertainty, more volatile), and whether Boys rankings in that age group should carry a DSS caveat. Note that U13B and U14B Boys GA volume is typically lower than U15–U17 because MLS NEXT enrollment peaks at older age groups.]

**MLS NEXT distribution health (B-OA-3):**
> [TO FILL: For age groups with ≥ 10 MLS NEXT teams, assess the distribution shape. Does the median sit in the 1,400–1,600 range? Does the spread (top 10% vs bottom 10%) indicate the calibration is differentiating between MLS NEXT tiers? A compressed distribution (top 10% only slightly above median) suggests MLS NEXT is underweighted. An extreme high tail (max > 1,800) suggests a merge artifact or a team with an inflated game history. Note any anomalous teams by name if identified from the reference club spot check.]

**Threshold evaluation:**
> [TO FILL: Across all three queries, does Boys Option A meet its pass criteria? Summarize: how many age groups passed cleanly on B-OA-1, how many cleared B-OA-2, how many MLS NEXT distributions look healthy in B-OA-3. State ELO's confidence level (HIGH / MEDIUM / LOW) in the Boys ratings and the primary reason.]

---

## Section 5 — Verdict

| Verdict | Condition | Next Step |
|---------|-----------|-----------|
| **APPROVE** | All 4 checks PASS (B-OA-1: all deltas ≤ 75 pts; B-OA-2: all age groups ≥ 500 games; B-OA-3: MLS NEXT distribution in range; no active calibration instability flag) + ELO confidence MEDIUM or HIGH | File this document. Update DSS Feature Readiness Tracker Boys cell to ✅. Reference `Rankings/Club Rankings — Boys-Conditional Implementation Timeline.md`. Notify SENTINEL. Boys Brier pre-check runs May 10–16 per execution package. |
| **CONDITIONAL** | Checks pass but ELO confidence MEDIUM with a specific named caveat (e.g., one age group in yellow range, one coverage gap below threshold but other evidence is sound) | File this document with caveat stated in Section 4. Bring to SENTINEL before authorizing Boys. Do not start Boys Club Rankings implementation until SENTINEL disposition filled. |
| **REJECT** | Any hard FAIL (B-OA-1 delta > 100 pts in any age group) OR B-OA-3 critical miss (any metric outside range by > 100 pts) OR ELO confidence LOW | File this document. Activate `Rankings/Club Rankings — Girls-Only Fallback Spec.md`. Alert FORGE immediately. Update DSS Feature Readiness Tracker Boys cell to ❌. File updated briefing same session. |

**Actual verdict:** [TO FILL: APPROVE / CONDITIONAL / REJECT]

**Verdict rationale:** [TO FILL: 2–3 sentences. State the single most important finding that drove the verdict. Be direct. If APPROVE: what gives ELO confidence? If CONDITIONAL: what is the specific caveat and why isn't it a REJECT? If REJECT: which check failed and what does it indicate about the Boys calibration state?]

---

## Section 6 — Implications

**If APPROVE:**
> [TO FILL: Confirm which age groups are cleared (typically U13B–U17B if all pass). Confirm Boys Club Rankings implementation can begin per `Rankings/Club Rankings — Boys-Conditional Implementation Timeline.md`. State what ELO monitors post-authorization. Confirm Boys Brier pre-check window: May 10–16 per `Rankings/Boys Brier — Pre-May-17 Execution Package.md`.]

**If CONDITIONAL:**
> [TO FILL: State the specific condition ELO is flagging for SENTINEL judgment. State ELO's recommendation (proceed with caveat / hold pending further check). Note any age groups excluded from Boys authorization. Note impact on Boys Brier pre-check timeline if delayed.]

**If REJECT:**
> [TO FILL: State the fallback path. Girls-Only Fallback Spec is already filed and standing by. State: when does FORGE need to be notified (immediately)? What is the impact on May 17 deployment timeline? What would Boys need to show in a re-check before Boys Club Rankings could be reconsidered? Is there a post-DSS path to Boys re-inclusion?]

---

## Section 7 — SENTINEL Authorization Request

```
Boys Club Rankings Decision
Date: [TO FILL: date results received and analysis complete]
From: ELO
To: SENTINEL

Query results received: [TO FILL]
Option A Result: PASS / CONDITIONAL / FAIL
ELO Verdict: APPROVE / CONDITIONAL / REJECT
Confidence: HIGH / MEDIUM / LOW

If APPROVE:
  ELO is ready to include Boys Club Rankings in the implementation plan.
  Boys age groups cleared: [TO FILL: U13B–U17B or subset]
  Age groups with coverage caveat (B-OA-2 thin): [TO FILL or "none"]
  Reference: this document, filed [date].
  Next: SENTINEL authorization → FORGE implementation → Boys Brier pre-check May 10–16

If CONDITIONAL:
  Specific condition for SENTINEL: [TO FILL: one sentence]
  ELO recommendation: [proceed with caveat / hold pending further check]
  SENTINEL disposition required before Boys implementation begins.

If REJECT:
  Girls-Only Fallback Spec activated: Rankings/Club Rankings — Girls-Only Fallback Spec.md
  FORGE notified: [date]
  DSS Feature Readiness Tracker Boys cell: ❌
  Post-DSS re-inclusion path: [TO FILL or "not available before May 9"]

SENTINEL signature required before Club Rankings implementation starts.
```

---

## Section 8 — SENTINEL Disposition

> *SENTINEL fills — do not pre-fill.*

```
Decision document reviewed: [ ]
ELO verdict accepted: [ ]
Boys Club Rankings: AUTHORIZED / HELD / GIRLS-ONLY
If AUTHORIZED: FORGE session may begin ___________
If HELD: reason: ___________
If GIRLS-ONLY: Girls-Only Fallback Spec reference confirmed: [ ]
DSS Feature Readiness Tracker to be updated by ELO: [ ]
SENTINEL sign-off: ___________
```

---

## Section 9 — Filing Record

```
Query results received: [TO FILL: date/time]
Analysis completed:     [TO FILL: date/time]
Document filed:         [TO FILL: date/time]
Filed by:               ELO
```

---

## References

- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — SQL queries (B-OA-1, B-OA-2, B-OA-3)
- `Rankings/Boys Option A — Decision Document.md` — prior decision record structure (pre-existing)
- `Rankings/Boys Option A — April 29 Quick-Check Card.md` — Presten execution reference
- `Rankings/Boys Calibration Status Summary — April 2026.md` — Boys baseline context
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — activated if REJECT
- `Rankings/Club Rankings — Boys-Conditional Implementation Timeline.md` — implementation timeline gated on APPROVE
- `Rankings/Boys Brier — Pre-May-17 Execution Package.md` — next step after APPROVE
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Boys cell to update after verdict

*ELO — 2026-04-29 (shell)*
