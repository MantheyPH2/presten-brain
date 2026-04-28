---
title: Boys Option A — Rapid Interpretation Guide
type: rapid-interpretation-guide
author: ELO
date: 2026-04-27
status: ready
related: "[[Boys Option A — Results Interpretation Guide]]", "[[Boys Option A — Decision Document]]", "[[Boys Option A Spot Check — Presten Execution Package]]", "[[Club Rankings — Girls-Only Fallback Spec]]"
tags: [boys, calibration, dss, interpretation, spot-check, rapid, evo-draw]
---

# Boys Option A — Rapid Interpretation Guide

> **Purpose:** 30-minute interpretation protocol for Boys Option A results. When Presten sends query output, ELO opens this document, identifies the pattern in ≤ 30 minutes, and files a Decision Document entry. This is NOT a detailed analysis document — it is a classification tool.
>
> **Overlap note:** The full `Boys Option A — Results Interpretation Guide.md` (2026-04-25) contains annotated examples and calibration reasoning. This guide contains the decision tree only. If the pattern classification is ambiguous after 30 minutes, declare AMBIGUOUS (Pattern D) and reference the full guide for investigation.

---

## Section 1: What Boys Option A Tests

**Query output structure (3 queries, all from `Boys Option A Spot Check — Presten Execution Package.md`):**

- **Query 1:** 10-row table — `age_group, gender, team_count, avg_rating, median_rating`. 2 rows per age group (M and F), U13–U17. ELO computes `delta = Boys avg_rating − Girls avg_rating` per age group.
- **Query 2:** 5-row table — `age_group, boys_ga_games, teams_with_ga_games`. One row per age group (U13B–U17B).
- **Query 3:** 5-row table — `age_group, mlsnext_team_count, avg_rating, median_rating, top_10pct_rating, bottom_10pct_rating, max_rating`. One row per Boys MLS NEXT age group.

**Calibration hypothesis being tested:**

Boys GA calibration (cal=100) has not been Brier-validated. Girls GA was found miscalibrated at cal=140 for GA ASPIRE events (40-point error). The spot check tests whether Boys show a parallel calibration error: Boys ratings inflating above Girls ratings by more than the structural tier difference (MLS NEXT Boys at 160 vs GA Girls at 140) explains. If Boys avg significantly exceeds Girls avg in any age group, Boys calibration may be inflated similarly to how Girls GA ASPIRE was inflated before the April 28 fix.

---

## Section 2: Expected Output Patterns

**Time-box this section to 30 minutes. If classification is unclear after 30 minutes → Pattern D.**

### Pattern A — PASS (all three queries within thresholds)

| Query | PASS Condition |
|-------|---------------|
| Q1 | All 5 age groups: \|Boys avg − Girls avg\| ≤ 100 pts |
| Q1 (quality gate) | Boys team_count ≥ 200 for all U13B–U17B |
| Q2 | All 5 age groups: Boys GA games ≥ 500 |
| Q3 | Median 1,400–1,600; Top 10% ≥ 1,700; Bottom 10% ≤ 1,300 for age groups with ≥ 10 MLS NEXT teams |

**Expected healthy example:** Boys avg exceeds Girls avg by 17–80 pts across all age groups. A structural Boys-higher gap is expected given MLS NEXT Boys (cal=160) vs Girls GA (cal=140).

### Pattern B — SOFT FAIL (minor miscalibration, targeted fix possible)

| Signal | Threshold |
|--------|-----------|
| Q1 divergence | One age group shows delta 76–100 pts AND all others ≤ 75 pts |
| Q1 team count | Borderline age group has team_count 200–300 (thin but not disqualifying) |
| Q2 volume | One or two age groups below 500 Boys GA games (coverage gap, not calibration failure) |
| Q3 distribution | One age group median outside range by 50–100 pts |

**ELO action on SOFT FAIL:** File Decision Document within 24 hours. Include targeted adjustment recommendation (specific age group or tier, not full recalibration). SENTINEL reviews before Presten executes.

### Pattern C — HARD FAIL (significant miscalibration, Boys Club Rankings deferred)

| Signal | Threshold |
|--------|-----------|
| Q1 divergence | Any age group delta > 100 pts AND team_count ≥ 300 |
| Q3 distribution | Any age group median outside range by > 100 pts AND mlsnext_count ≥ 20 |
| Q1 direction | Girls avg exceeds Boys avg by > 50 pts in any age group (GA ASPIRE overcorrection signal — escalate immediately) |

**ELO action on HARD FAIL:** File SENTINEL queue alert within 2 hours of classification. Girls-Only Fallback Spec is ACTIVATED. Boys Club Rankings excluded from DSS. File Decision Document within 48 hours.

### Pattern D — AMBIGUOUS (cannot classify in 30 minutes)

| Condition | Trigger |
|-----------|---------|
| Q1 borderline for multiple age groups | 2+ age groups in 76–100 pts range |
| Q1 thin data + borderline result | Any age group: delta 90–110 pts AND team_count < 200 |
| Q2 + Q1 conflict | Q1 passes but Q2 shows severe volume deficit (< 200 games) in the same age group |
| Q3 edge case | mlsnext_count < 10 for an age group showing a concerning median |

**ELO action on AMBIGUOUS:** Do not file PASS or FAIL. File interim note in Decision Document: "Evidence collection in progress — ambiguous on [age group(s)]. Requesting one targeted check by [date max 2 days out]." Notify SENTINEL via queue item same session.

---

## Section 3: Decision Mapping

| Pattern | ELO Verdict | Decision Document Filing | SENTINEL Action |
|---------|------------|------------------------|----------------|
| A — PASS | Boys calibration confirmed viable | File within 24 hours of results | None — ELO proceeds with Boys Club Rankings |
| B — SOFT FAIL | Targeted adjustment recommended | File within 24 hours with adjustment spec | SENTINEL reviews adjustment before Presten executes |
| C — HARD FAIL | Boys recalibration required | File SENTINEL alert within 2 hrs; Decision Document within 48 hrs | Escalate to Presten; activate Girls-Only Fallback immediately |
| D — AMBIGUOUS | Additional data needed | File interim note within 4 hours | Notified — ELO requests 1 targeted query; 2-day resolution target |

---

## Section 4: Filing Protocol

When results arrive from Presten, execute in this exact order:

1. **Minutes 0–5:** Open the three query output files. Verify row structure matches expected format (10 rows for Q1, 5 rows for Q2, 5 rows for Q3). If format is unexpected, note and adapt.

2. **Minutes 5–20:** Compute Q1 deltas for each age group. Record:
   ```
   U13: Boys avg [N] − Girls avg [N] = delta [N] pts
   U14: Boys avg [N] − Girls avg [N] = delta [N] pts
   U15: Boys avg [N] − Girls avg [N] = delta [N] pts
   U16: Boys avg [N] − Girls avg [N] = delta [N] pts
   U17: Boys avg [N] − Girls avg [N] = delta [N] pts
   ```
   Check Q2 game counts against ≥ 500 threshold. Check Q3 median/decile values against expected ranges.

3. **Minutes 20–30:** Apply Section 2 pattern classification. Select Pattern A, B, C, or D. If the 30-minute clock expires without a clear pattern → Pattern D by default.

4. **Within 30 minutes of classification:** File one of these in the ELO briefing or queue:
   - `"Option A: Pattern A (PASS) — Decision Document filing within 24 hours."`
   - `"Option A: Pattern B (SOFT FAIL) — targeted adjustment identified, Decision Document within 24 hours."`
   - `"Option A: Pattern C (HARD FAIL) — Girls-Only Fallback ACTIVATED. SENTINEL alert filed."`
   - `"Option A: Pattern D (AMBIGUOUS) — interim note filed, targeted check requested, resolution by [date]."`

5. **Decision Document:** Fill `Rankings/Boys Option A — Decision Document.md` per Section 3 filing timeline. Mark the active PASS/FAIL branch.

6. If Pattern C: **also** update `Rankings/Product & Planning/DSS Feature Readiness Tracker — May 2026.md` Boys Club Rankings cell to ❌ and reference `Rankings/Club Rankings — Girls-Only Fallback Spec.md`.

---

## Section 5: Known Edge Cases

**Edge case 1 — Thin data at U17B:**
U17B often has lower Boys team counts. If U17B delta is 90–110 pts with team_count < 200, do not automatically classify as Pattern C. Classify as Pattern D (ambiguous, thin data) and run a targeted check for U17B before filing verdict.

**Edge case 2 — Q1 delta is Girls-higher (Girls > Boys):**
If any age group shows Girls avg significantly above Boys avg (Girls > Boys by > 50 pts), this is an unexpected signal. Boys MLS NEXT (cal=160) should pull Boys average above Girls GA (cal=140) in every age group. Girls-higher at > 50 pts may mean the GA ASPIRE fix overcorrected Girls ratings downward — classify as **Pattern C** and escalate to SENTINEL immediately, regardless of other Q1 results.

**Edge case 3 — Q2 age groups below 500:**
Low Boys GA volume does not auto-fail Boys Club Rankings. Classify as Pattern B (coverage gap) for the affected age groups. Note in Decision Document that those age groups' Boys GA calibration is underpowered and should not be cited as validation evidence at DSS.

**Edge case 4 — MLS NEXT tier split mid-deployment:**
Boys MLS NEXT tier split (Homegrown 160 / Academy 135) is scheduled May 18–20 — after the Option A spot check window (April 29 – May 5). Q3 results reflect the pre-split unified cal=160. Interpret Q3 against the pre-split expected ranges (median 1,400–1,600). Do not adjust thresholds for the pending split.

**Edge case 5 — Off-season data lean:**
If Presten runs the spot check in late April, the most recent games may be from fall 2025 — older than spring season games. Ratings lean slightly toward older game history. This normally does not materially affect the divergence (Q1), but if Q3 MLS NEXT medians appear slightly low (1,380–1,400), check whether this is an off-season effect before declaring Pattern B.

---

## Verdict Formats (copy-paste into briefing)

**PASS:**
> "Boys Option A: Pattern A — PASS. Q1 max delta [N] pts ([age group]) — threshold 100 pts. All 5 age groups within range. Q2: all age groups ≥ 500 Boys GA games. Q3: MLS NEXT distribution nominal. Decision Document filed. Boys Club Rankings authorized for DSS demo."

**SOFT FAIL:**
> "Boys Option A: Pattern B — SOFT FAIL. Q1: [age group] delta [N] pts (76–100 range). Targeted adjustment identified: [description]. Decision Document filed with adjustment spec. Awaiting SENTINEL review before Presten executes."

**HARD FAIL:**
> "Boys Option A: Pattern C — HARD FAIL. Q1: [age group] delta [N] pts, exceeds 100-pt threshold. Girls-Only Fallback ACTIVATED — `Rankings/Club Rankings — Girls-Only Fallback Spec.md`. FORGE informed. Boys Club Rankings excluded from DSS demo. SENTINEL alert filed."

**AMBIGUOUS:**
> "Boys Option A: Pattern D — AMBIGUOUS. [N] age group(s) borderline: [list]. Targeted check requested for [age group(s)]. Interim note in Decision Document. Resolution target: [date, max 2 days out]. No verdict filed until resolved."

---

## References

- [[Boys Option A — Results Interpretation Guide]] — detailed annotated interpretation (April 25 — use for investigation)
- [[Boys Option A — Decision Document]] — fill this using Section 4 filing protocol
- [[Boys Option A Spot Check — Presten Execution Package]] — the three queries this guide interprets
- [[Boys Option A — Pre-Verdict Evidence Checklist]] — evidence gate before final verdict
- [[Club Rankings — Girls-Only Fallback Spec]] — activated on Pattern C

*ELO — 2026-04-27*
