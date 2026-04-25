---
title: Boys Option A — Results Interpretation Guide
type: interpretation-guide
author: ELO
date: 2026-04-25
status: ready
related: "[[Boys Option A Spot Check — Presten Execution Package]]", "[[Boys Calibration Option A — Interpretation Guide]]", "[[Club Rankings — Girls-Only Fallback Spec]]"
tags: [boys, calibration, dss, interpretation, spot-check, evo-draw]
---

# Boys Option A — Results Interpretation Guide

> **Purpose:** ELO's annotated guide to Boys Option A spot check results. Assumes Presten has run the three queries from `Boys Option A Spot Check — Presten Execution Package.md` and is looking at output. Use this guide to classify results, identify borderlines, and file a verdict within 30 minutes.
>
> **Overlap note:** This guide interprets the three queries in the Execution Package (Query 1: Boys vs. Girls avg rating by age group; Query 2: Boys GA game volume; Query 3: MLS NEXT Boys distribution). The earlier `Boys Calibration Option A — Interpretation Guide.md` (2026-04-24) covers a parallel top-10 spot check format and game volume checks. Where approaches differ, this guide governs — it was written to match the specific query outputs in the Execution Package.

---

## Section 1: What a Passing Result Looks Like

### Query 1 — Boys vs. Girls Average Rating by Age Group

**Expected output format:** 10 rows, 2 per age group (one `M`, one `F`). Columns: `age_group, gender, team_count, avg_rating, median_rating`.

**What healthy calibration looks like:** Boys and Girls average ratings should track within 75–100 points of each other for every age group. Boys MLS NEXT calibration (cal=160/135) pulls Boys top teams slightly higher than Girls GA (cal=140) pulls Girls top teams — a modest Boys-higher divergence is structurally expected. A gap of 40–80 points Boys-higher is completely normal; below 40 may indicate Boys coverage is thin (fewer high-cal-tier games per age group).

**Example passing result:**

```
age_group | gender | team_count | avg_rating | median_rating
----------+--------+------------+------------+--------------
U13       | F      |       412  |     1087.3 |       1051.0
U13       | M      |       387  |     1104.6 |       1063.0
U14       | F      |       398  |     1092.1 |       1058.0
U14       | M      |       361  |     1119.8 |       1074.0
U15       | F      |       344  |     1101.4 |       1065.0
U15       | M      |       329  |     1132.7 |       1089.0
U16       | F      |       311  |     1098.6 |       1062.0
U16       | M      |       298  |     1128.3 |       1083.0
U17       | F      |       289  |     1093.2 |       1059.0
U17       | M      |       271  |     1125.1 |       1081.0
```

In this example, Boys avg exceeds Girls avg by 17–44 pts across all age groups — well within the 100-pt pass threshold. This is PASS.

**PASS criteria (all must hold):**
- For every age group, |Boys avg_rating − Girls avg_rating| ≤ 100 pts
- Boys team_count ≥ 200 for U13–U17 (sufficient data for the average to be meaningful)
- No single age group shows Boys avg > Girls avg by > 100 pts AND Boys team_count < 200 simultaneously (thin data + large divergence = uninformative, not passing)

---

### Query 2 — Boys GA Game Volume Validation

**Expected output format:** 5 rows (one per age group). Columns: `age_group, boys_ga_games, teams_with_ga_games`.

**What healthy calibration looks like:** Each Boys age group should have ≥ 500 GA games. The Boys GA calibration value (cal=100) applies a moderate tier boost to Boys teams playing GA-tier events. Below 500 games, this boost is applied to a thin sample where individual game outcomes drive rating noise more than calibration signal.

**Example passing result:**

```
age_group | boys_ga_games | teams_with_ga_games
----------+---------------+--------------------
U13       |           734 |                 89
U14       |           812 |                 97
U15       |           651 |                 78
U16       |           589 |                 72
U17       |           503 |                 63
```

All age groups ≥ 500 — PASS.

---

### Query 3 — MLS NEXT Boys Rating Distribution

**Expected output format:** 5 rows (one per age group). Columns: `age_group, mlsnext_team_count, avg_rating, median_rating, top_10pct_rating, bottom_10pct_rating, max_rating`.

**What healthy calibration looks like:** MLS NEXT is the best-validated Boys calibration anchor. With cal=160 (Homegrown) and cal=135 (Academy), the MLS NEXT Boys tier should produce a recognizably elite distribution: medians in the 1,400–1,600 range, top decile ≥ 1,700, bottom decile ≤ 1,300.

**Example passing result:**

```
age_group | mlsnext_count | avg_rating | median_rating | top_10pct | bottom_10pct | max_rating
----------+---------------+------------+---------------+-----------+--------------+-----------
U13       |            38 |    1463.2  |       1447.0  |   1718.0  |      1291.0  |    1834.0
U14       |            42 |    1481.6  |       1469.0  |   1731.0  |      1307.0  |    1851.0
U15       |            51 |    1502.3  |       1489.0  |   1744.0  |      1321.0  |    1867.0
U16       |            49 |    1511.7  |       1497.0  |   1756.0  |      1328.0  |    1879.0
U17       |            46 |    1498.4  |       1483.0  |   1741.0  |      1313.0  |    1862.0
```

All medians in 1,400–1,600 range; top decile ≥ 1,700; bottom decile ≤ 1,300 — PASS.

---

## Section 2: What a Failing Result Looks Like

### Query 1 — FAIL Scenarios

**FAIL example — excessive divergence, Boys-higher:**

```
age_group | gender | team_count | avg_rating | median_rating
----------+--------+------------+------------+--------------
U13       | F      |       412  |     1087.3 |       1051.0
U13       | M      |       124  |     1249.1 |       1201.0   ← 162 pts gap, FAIL
```

**Interpretation:** A 162-pt Boys-higher divergence at U13 suggests Boys ratings are inflated, possibly due to a bad merge artifact pulling Boys ratings up, or insufficient Girls U13 coverage inflating the baseline comparison. Boys club rankings for U13 cannot be shown at DSS in this scenario.

**Is the failure direction asymmetric?** Yes. Boys-higher divergence (Boys > Girls by > 100 pts) suggests Boys calibration is producing inflated ratings for Boys, which would make Boys rankings misleading to an audience who knows these clubs. Girls-higher divergence (Girls > Boys by > 100 pts) is more unexpected — Boys MLS NEXT (cal=160) should pull Boys at least as high as Girls GA (cal=140). Girls-higher at U13/U14 is a signal the GA ASPIRE fix may have overcorrected Girls ratings — flag this to SENTINEL immediately.

**Age groups where query results may be noisy (low game volume):** U17B and U19B have lower game counts overall — Boys U17 team counts below 200 make the divergence measurement less reliable. If U17B shows a borderline result (90–110 pts) with team_count < 150, ELO investigates before declaring fail. U13B and U14B at team_count < 200 should be treated as data quality issues, not calibration evidence.

---

### Query 2 — FAIL Scenarios

**FAIL example — thin Boys GA coverage:**

```
age_group | boys_ga_games | teams_with_ga_games
----------+---------------+--------------------
U13       |           381 |                 47   ← below 500, FAIL
U14       |           823 |                 98
U15       |           641 |                 76
U16       |           497 |                 61   ← below 500, FAIL
U17       |           312 |                 39   ← below 500, FAIL
```

**Interpretation:** U13B, U16B, and U17B Boys GA coverage is thin. The cal=100 Boys GA calibration is applied but underpowered. Boys rankings for these age groups are more volatile than expected. **This does NOT trigger Girls-only fallback for Club Rankings** — it is a coverage gap note, not a calibration failure. If Boys rankings are shown at DSS, avoid citing Boys GA-dominant teams as examples of well-validated calibration.

---

### Query 3 — FAIL Scenarios

**FAIL example — MLS NEXT distribution out of range:**

```
age_group | mlsnext_count | avg_rating | median_rating | top_10pct | bottom_10pct | max_rating
----------+---------------+------------+---------------+-----------+--------------+-----------
U13       |            14 |    1321.7  |       1287.0  |   1549.0  |      1114.0  ← median too low, FAIL
```

**Interpretation:** The U13B MLS NEXT median (1,287) is 113 pts below the lower bound of the expected range (1,400). This could indicate: (a) Boys U13 MLS NEXT team count is too low (14 teams — marginal) for the median to be reliable, or (b) a systematic Boys U13 calibration issue. With mlsnext_count = 14, ELO investigates whether the count is the driver before declaring a calibration problem. Contact ELO with the specific numbers — do not mark Boys as demo-safe until ELO responds.

---

## Section 3: Borderline Results

**The threshold is > 100 pts = FAIL (Query 1). ELO's policy:**

- **90–110 pts in a single age group, all others clear (< 80 pts):** Requires ELO investigation, not automatic Girls-only fallback. ELO reviews the specific age group in the next session. If the borderline age group is U17 or U19 and team_count is below 200, ELO may classify as data noise rather than calibration failure.

- **95–105 pts for one age group, team_count ≥ 300:** Hard borderline. ELO will run a targeted check: pull top-10 Boys teams for that age group and check for merge artifacts or coverage anomalies. If no anomaly found and the divergence is explained by MLS NEXT vs GA calibration differential, ELO classifies as PASS with a note.

- **> 105 pts for any age group with team_count ≥ 300:** FAIL. Girls-only fallback activated.

**Standard briefing language for borderline results:**
> "Boys Option A result: [N] age group(s) borderline (95–110 pts divergence). ELO assessment: [pass-with-note / investigate / fail] because [reason — e.g., U17B team_count too low to be diagnostic / or: U14B borderline with 320+ teams, needs targeted check]."

---

## Section 4: How to Send Results to ELO

After completing all three queries:

1. Save output files per the Execution Package instructions:
   - `02 - Tiger Tournaments/Projects/Reports/boys-option-a-query1-[YYYY-MM-DD].md`
   - `02 - Tiger Tournaments/Projects/Reports/boys-option-a-query2-[YYYY-MM-DD].md`
   - `02 - Tiger Tournaments/Projects/Reports/boys-option-a-query3-[YYYY-MM-DD].md`

2. Copy-paste from psql is fine — ELO will parse the raw output. No reformatting needed.

3. Reference the files in the session briefing or Queue so ELO sees them at the next check.

**ELO turnaround:** ELO files interpretation + verdict in the next briefing after results are received. Given the May 5 deadline, results received by May 3 give ELO two briefing cycles to respond before the cutoff. Results received May 4–5 will be turned around same-cycle.

---

## Section 5: Verdict Formats

File one of three verdicts in the briefing after reviewing Presten's query output:

**PASS:**
> "Boys Option A spot check: PASS. Query 1 max divergence [N] pts ([age group]) — threshold: 100 pts. All 5 age groups within range. Query 2: all U13B–U17B ≥ 500 Boys GA games. Query 3: MLS NEXT distribution within expected range (median [N], top decile [N], bottom decile [N]). Boys club rankings cleared for full implementation. Girls-Only Fallback Spec on hold — not activated."

**FAIL:**
> "Boys Option A spot check: FAIL. Query 1: [age group X] divergence [N] pts, exceeds 100-pt threshold. Girls-Only Fallback Spec ACTIVATED. See `Rankings/Club Rankings — Girls-Only Fallback Spec.md`. FORGE informed. Boys club rankings excluded from DSS demo entirely."

**INCONCLUSIVE:**
> "Boys Option A spot check: INCONCLUSIVE. [N] age group(s) borderline (Query 1: [list age groups and divergence values]). ELO investigation required — running targeted top-10 check for [age group]. No immediate activation of Girls-Only Fallback. Holding pending ELO targeted check by [specific date, no more than 2 days out]."

---

## References

- [[Boys Option A Spot Check — Presten Execution Package]] — the three queries this guide annotates
- [[Club Rankings — Girls-Only Fallback Spec]] — activated if Query 1 FAILs
- [[Boys Calibration Status Summary — April 2026]] — Boys calibration baseline
- [[Boys Calibration Option A — Interpretation Guide]] — parallel interpretation guide for the top-10 spot check format (April 2026)

*ELO — 2026-04-25*
