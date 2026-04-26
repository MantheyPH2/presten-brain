---
title: Boys Option A — Verdict Filing Document
type: verdict-gate
author: ELO
date: 2026-04-25
status: part-1-complete
due: 2026-05-05
related: "[[Club Rankings — Boys-Conditional Implementation Timeline]]", "[[Boys Option A Spot Check — Presten Execution Package]]", "[[Boys Option A — Results Interpretation Guide]]"
tags: [boys, calibration, club-rankings, verdict, dss, evo-draw]
---

# Boys Option A — Verdict Filing Document

> **[Part 1 written 2026-04-25 — before any Boys Option A results arrive. Criteria locked. Do not revise Part 1 thresholds after results are received.]**
>
> **What this decides:** Whether Boys club rankings are included in the DSS demo (Branch A: full Boys + Girls implementation, 10–11h FORGE session) or excluded (Branch B: Girls-only Club Rankings, 7.5h). No-result by May 5 EOD = Branch B by default.

---

## Part 1: PASS/FAIL Criteria

_[Written before results. Criteria locked on: 2026-04-25.]_

### What Boys Option A Is

Boys Option A is the decision to include Boys club rankings in DSS by validating Boys calibration with a three-query spot check. The spot check tests whether Boys ratings are plausible relative to Girls ratings (Criterion A), whether Boys GA coverage is sufficient to support the cal=100 calibration (Criterion B), and whether MLS NEXT Boys ratings — the best-validated Boys calibration anchor — fall within the expected elite distribution (Criterion C).

Option A does NOT change any calibration parameters. It is a read-only validation. PASS = Boys ratings are plausible for DSS. FAIL = Boys ratings have an undetected issue; show Girls-only Club Rankings.

---

### Criterion A: Boys vs. Girls Distribution (Query 1)

Boys and Girls average ratings should track within 75–100 points of each other across all U13–U17 age groups. Boys MLS NEXT calibration (cal=160/135) is higher than Girls GA (cal=140), so a modest Boys-higher divergence (40–80 pts) is structurally expected. A gap > 100 pts indicates Boys ratings are inflated and not credible for a public demo.

**PASS threshold:**
- `|Boys avg_rating − Girls avg_rating| ≤ 100 pts` for every age group (U13–U17)
- Boys `team_count ≥ 200` for U13–U17 (results are meaningful)
- Borderline (90–105 pts) with `team_count < 200`: ELO may classify as data noise, not FAIL, with one-sentence explanation

**FAIL threshold:**
- Any age group where `|Boys avg − Girls avg| > 100 pts` AND `Boys team_count ≥ 200`
- Any age group where `|Boys avg − Girls avg| > 105 pts` AND `Boys team_count ≥ 300`

**Inconclusive threshold:**
- `90–105 pts divergence` in a single age group where `team_count 200–299`: ELO investigates before declaring FAIL. If no anomaly found within one session, classify PASS with note.
- If two or more age groups are borderline simultaneously: FAIL default.

**Direction asymmetry note:** Boys-higher divergence (Boys > Girls) suggests Boys calibration inflated. Girls-higher divergence (Girls > Boys by > 100 pts) is more unexpected and may indicate the GA ASPIRE fix overcorrected Girls upward — flag this to SENTINEL immediately regardless of overall verdict.

---

### Criterion B: Boys GA Game Volume (Query 2)

The Boys GA calibration value (cal=100) requires at least 500 GA games per age group to be a meaningful signal rather than rating noise from individual game outcomes.

**PASS threshold:**
- `Boys GA games ≥ 500` for all U13B–U17B age groups

**FAIL threshold:**
- This criterion does NOT trigger Girls-only fallback by itself. If any age group is below 500, it is logged as a **coverage note** — Boys rankings for those age groups should not be cited as a calibration-validated example during the demo. The overall verdict is still governed by Criterion A and C.

**Inconclusive threshold:** Not applicable. Query 2 is a coverage check, not a binary gate.

---

### Criterion C: MLS NEXT Boys Distribution (Query 3)

MLS NEXT is the best-empirically-validated Boys calibration anchor (5,800 cross-league games, cal=160 Homegrown / cal=135 Academy confirmed). The MLS NEXT Boys distribution should be recognizably elite: median in the 1,400–1,600 range, top decile ≥ 1,700, bottom decile ≤ 1,300.

**PASS threshold (all must hold for each age group U13B–U17B):**
- `median_rating` in range 1,400–1,600
- `top_10pct_rating ≥ 1,700`
- `bottom_10pct_rating ≤ 1,300`
- `mlsnext_team_count ≥ 15` (if fewer than 15 teams, data is too thin to be diagnostic — treat as inconclusive, not fail)

**FAIL threshold:**
- `median_rating < 1,400` in any age group with `mlsnext_team_count ≥ 20`
- `top_10pct_rating < 1,700` in any age group with `mlsnext_team_count ≥ 20`

**Inconclusive threshold:**
- Any out-of-range value with `mlsnext_team_count < 15`: ELO notes as data-sparse; does not trigger Girls-only fallback.

---

### Overall Verdict Rule

| Criteria Met | Verdict |
|-------------|---------|
| A PASS + B (any coverage) + C PASS | **PASS — Activate Branch A** |
| A FAIL (any age group, full criteria) | **FAIL — Activate Branch B** |
| A PASS + C FAIL (any age group, full criteria) | **FAIL — Activate Branch B** |
| A INCONCLUSIVE (one borderline age group, ELO investigation pending) | **INCONCLUSIVE → default Branch B** unless ELO investigation completes before May 5 EOD and clears the borderline |
| Results not received by May 5 EOD | **NO RESULT → default Branch B** |

**INCONCLUSIVE default to Branch B is non-negotiable.** The May 5 EOD deadline is firm. If the verdict is not PASS by that date, Branch B activates automatically.

---

## Part 2: Results and Verdict

_ELO fills on verdict date. No earlier than results receipt; no later than May 5 EOD._

```
Verdict Date: ___________
Results Source: [file paths: boys-option-a-query1-*.md, query2-*.md, query3-*.md]
Results confirmed received: YES / NO
```

### Criterion A Results

```
Query 1 output file: ___________
Age groups checked: U13B U14B U15B U16B U17B

| Age Group | Boys avg | Girls avg | Gap | team_count (Boys) | Within threshold? |
|-----------|----------|-----------|-----|-------------------|-------------------|
| U13       |          |           |     |                   |                   |
| U14       |          |           |     |                   |                   |
| U15       |          |           |     |                   |                   |
| U16       |          |           |     |                   |                   |
| U17       |          |           |     |                   |                   |

Criterion A verdict: PASS / FAIL / INCONCLUSIVE
Note: ___________
```

### Criterion B Results

```
Query 2 output file: ___________

| Age Group | Boys GA games | ≥ 500? | Coverage note |
|-----------|--------------|--------|---------------|
| U13       |              |        |               |
| U14       |              |        |               |
| U15       |              |        |               |
| U16       |              |        |               |
| U17       |              |        |               |

Criterion B verdict: (coverage note only — does not gate overall verdict)
Age groups with thin coverage (< 500): ___________
Demo guidance for thin-coverage age groups: Avoid citing Boys GA-dominant teams as calibration-validated examples for these age groups.
```

### Criterion C Results

```
Query 3 output file: ___________

| Age Group | mlsnext_count | median | top_10pct | bottom_10pct | In range? |
|-----------|--------------|--------|-----------|--------------|-----------|
| U13       |              |        |           |              |           |
| U14       |              |        |           |              |           |
| U15       |              |        |           |              |           |
| U16       |              |        |           |              |           |
| U17       |              |        |           |              |           |

Criterion C verdict: PASS / FAIL / INCONCLUSIVE
Note: ___________
```

### Overall Verdict Declaration

```
OVERALL VERDICT: PASS / FAIL / INCONCLUSIVE / NO RESULT
Club Rankings Branch Activated: A / B
Branch activation reason: ___________
SENTINEL authorization request: Activate Club Rankings Branch [A/B].
  FORGE scope: [Branch A = full Boys + Girls club rankings, ~10–11h session / Branch B = Girls-only club rankings, ~7.5h session]
  FORGE session target: [date from Club Rankings — Boys-Conditional Implementation Timeline.md]
ELO notes: ___________
```

---

## SENTINEL Authorization Block

_SENTINEL fills after reading Part 2._

```
Verdict reviewed: [ ]
Part 1 criteria confirmed as pre-results: [ ]
Club Rankings Branch [A/B]: AUTHORIZED
FORGE scoped for [Branch scope]: [ ]
FORGE execution window: [date range from Conditional Timeline]
May 9 readiness gate dependency: [note any change to May 9 gate from this verdict]
Authorization date: ___________
```

---

## References

- `Rankings/Club Rankings — Boys-Conditional Implementation Timeline.md` — two-branch timeline this verdict activates
- `Rankings/Boys Option A Spot Check — Presten Execution Package.md` — queries Presten runs to produce the results this document scores
- `Rankings/Boys Option A — Results Interpretation Guide.md` — detailed interpretation of borderline cases for Criteria A, B, C
- `Rankings/Boys Calibration Status Summary — April 2026.md` — background on Boys calibration state
- `Rankings/Club Rankings — Girls-Only Fallback Spec.md` — Branch B specification activated if verdict = FAIL/INCONCLUSIVE/NO RESULT

*ELO — 2026-04-25*
