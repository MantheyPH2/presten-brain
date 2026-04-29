---
type: elo-execution-plan
topic: boys-option-a-post-verdict
date: 2026-04-29
status: pre-staged
author: ELO
tags: [elo, boys-calibration, option-a, execution-plan, evo-draw]
---

# Boys Option A — Post-Verdict Execution Plan

> Execute this plan within 60 minutes of receiving B-OA-1/2/3 query results from Presten. All steps are ordered by dependency — do not skip ahead. Pre-conditions: results received; all thresholds pre-filled below; no DB access required after results arrive.

---

## Section 1 — What ELO Is Deciding

Boys Option A = setting Boys GA ASPIRE calibration at cal=100, inheriting the Boys GA tier value. A PASS verdict means Boys GA ASPIRE teams are correctly rated under the current assumption and no pre-DSS recalibration is needed. A FAIL verdict means the cal=100 assumption is materially wrong and a corrected value (likely ~70, per Option B logic) must be scoped — but this does not block DSS; Boys GA ASPIRE correction is post-DSS per the Boys GA ASPIRE Calibration Decision document (`Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md`).

---

## Section 2 — The Three Query Results

| Query | What It Measures | Pass Threshold | Fail Threshold |
|-------|-----------------|----------------|----------------|
| B-OA-1 | Boys vs. Girls avg rating divergence by age group (U13–U17) | ALL age groups: absolute difference ≤ 100 pts between Boys and Girls avg | ANY age group: absolute difference > 100 pts — Boys ratings have systematic miscalibration |
| B-OA-2 | Boys GA/GA ASPIRE game volume by age group (U13–U17B) | All age groups: ≥ 500 Boys GA/GA ASPIRE games | Any age group: < 500 games — COVERAGE FLAG (not a full verdict FAIL; calibration claim is thin for that age group) |
| B-OA-3 | MLS NEXT Boys rating distribution by age group (for age groups with ≥ 10 MLS NEXT teams) | Median: 1,400–1,600; Top 10th percentile: ≥ 1,700; Bottom 10th percentile: ≤ 1,300 | Any age group: median or decile outside range by > 100 pts — contact ELO immediately |

---

## Section 3 — Decision Tree (30 minutes)

**Step 1 — Receive and Record Results (5 min)**

- Open `Rankings/Boys Calibration — Option A Verdict Document.md`
- Fill B-OA-1, B-OA-2, B-OA-3 result cells in the Query Results table
- Do not interpret yet — record raw numbers first

**Step 2 — Classify Verdict (10 min)**

```
IF B-OA-1 PASS AND B-OA-3 PASS (and B-OA-2 ≥ 500 all age groups):
  → VERDICT: APPROVE
  → Proceed to Step 3 (APPROVE path)

IF B-OA-1 FAIL (any age group > 100 pts divergence):
  → VERDICT: FAIL
  → Proceed to Step 3 (FAIL path)

IF B-OA-3 FAIL (any age group median/decile out of range by > 100 pts):
  → VERDICT: FAIL
  → Proceed to Step 3 (FAIL path)

IF B-OA-1 PASS AND B-OA-3 within 50–100 pts of threshold (marginal):
  → VERDICT: CONDITIONAL
  → Proceed to Step 3 (CONDITIONAL path)

IF B-OA-2 COVERAGE FLAG only (no B-OA-1 or B-OA-3 fail):
  → VERDICT: APPROVE WITH NOTE
  → Note coverage gaps in verdict document; proceed to APPROVE path
```

**Step 3 — Fill Verdict Document (10 min)**

Fill `Rankings/Boys Calibration — Option A Verdict Document.md`:
- Sections 3–5 (results, analysis, verdict)
- Analysis: interpret direction and magnitude of each query result
- Verdict: state APPROVE / CONDITIONAL / FAIL and 2–3 sentence rationale

**Step 4 — Update Vault Documents (5 min)**

- `Rankings/Calibration Values — League Hierarchy Reconciliation.md` — Boys GA ASPIRE row: mark "Option A confirmed [date]" if APPROVE; "Option A FAIL — revision pending" if FAIL
- `Rankings/Boys Calibration Status Summary — April 2026.md` — update Boys calibration state
- If APPROVE: add entry to `Rankings/Recent Changes 2024-2026.md` — "Boys GA ASPIRE cal=100 confirmed [date] per B-OA-1/2/3 spot check"
- If FAIL: update `Rankings/May 9 DSS Gate — Risk Register.md` R2 probability from Medium → High

**Step 5 — Notify SENTINEL (5 min)**

- Mark this task `status: completed` in `10 - Agents/ELO/Tasks/task-2026-04-29-boys-option-a-post-verdict-actions.md`
- State verdict in next ELO briefing with full summary
- If APPROVE: note Club Rankings boys-conditional timeline unblocked (per `task-2026-04-27-club-rankings-go-no-go-recommendation.md`)
- If FAIL or CONDITIONAL: file Queue item for SENTINEL within same session

---

## Section 4 — Downstream Implications

| Verdict | Impact on Boys Club Rankings | Impact on May 9 DSS | Impact on Calibration | Action |
|---------|------------------------------|--------------------|-----------------------|--------|
| APPROVE | Boys Club Rankings timeline proceeds per `Rankings/Club Rankings — Boys-Conditional Implementation Timeline.md` | Boys calibration confirmed demo-safe for U13–U17B | No change needed pre-DSS | File verdict, notify SENTINEL, update calibration docs |
| APPROVE WITH NOTE | Boys Club Rankings may have thin results for low-coverage age groups | Note coverage gaps in demo script; otherwise demo-safe | Boys GA coverage improvement scheduled 2026-27 season | File verdict with coverage note; advise demo script to caveat specific age groups |
| CONDITIONAL | Hold Boys Club Rankings pending Presten decision on marginal result | File analysis before May 9; SENTINEL reviews | ELO proposes revised cal value or threshold adjustment | File conditional analysis, post Queue item to SENTINEL; await Presten direction |
| FAIL | Boys Club Rankings delayed post-DSS; Girls-only Club Rankings fallback activates per `Rankings/Club Rankings — Girls-Only Fallback Spec.md` | Boys demo scope reduced to Girls; must notify SENTINEL immediately | ELO proposes Option B (cal=70) for post-DSS analysis | Escalate to Presten, file FAIL report, update risk register |

---

## Section 5 — Pre-Conditions

This plan executes as soon as Presten shares B-OA-1/2/3 results. This plan does NOT require:

- G0 gate open (Boys Option A is G0-independent)
- April 29 GA ASPIRE session execution
- Any FORGE pipeline results
- SENTINEL pre-authorization

The only blocker is results receipt. Once results arrive, ELO executes all 5 steps within 60 minutes.

---

## References

- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — B-OA-1/2/3 query definitions and thresholds
- `Rankings/Boys Calibration — Option A Verdict Document.md` — pre-staged verdict shell (complete this on results receipt)
- `Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md` — Option A decision (cal=100) and Option B fallback (~cal=70)
- `Rankings/Boys Calibration Status Summary — April 2026.md` — full Boys calibration context
- `Rankings/Club Rankings — Boys-Conditional Implementation Timeline.md` — Boys Club Rankings timeline depends on this verdict
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — fallback if verdict = FAIL

*ELO — 2026-04-29*
