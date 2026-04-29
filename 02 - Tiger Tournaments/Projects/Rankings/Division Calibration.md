---
title: Division Calibration
aliases: [Division Placement, DSS Divisions, Anti-Sandbagging]
tags: [rankings, calibration, divisions, dss, evo-draw]
created: 2026-04-21
updated: 2026-04-29
---

# Division Calibration

Division calibration is Phase 9 of the [[Ranking Engine]], applied to teams in **managed events** like the [[Dallas Spring Showcase]]. It adjusts Elo ratings based on the division a team has been placed into.

---

## DSS Division Hierarchy

From highest to lowest competitive level:

| Rank | Division       | DSS Teams |
| ---- | -------------- | --------- |
| 1    | Gold           | 53        |
| 2    | Silver Elite   | 24        |
| 3    | Silver         | 71        |
| 4    | Bronze         | 75        |
| 5    | Bronze II      | 66        |
| 6    | Recreational   | 19        |
|      | **Total**      | **308**   |

> [!info] Calibration Logic
> Teams in higher divisions receive a rating floor/boost proportional to their division rank. A Gold-division team should never rank below a Bronze-division team with similar raw Elo — the division placement is expert human judgment that the algorithm respects.

---

## Anti-Sandbagging

> [!warning] Code Exists but DISABLED
> Anti-sandbagging detection code has been written but is currently **disabled**. The intent is to flag teams that deliberately register in lower divisions despite having the talent for higher ones.
>
> **Why disabled:** Needs more Elo data to establish reliable thresholds. With only one season of data, the false-positive rate would be too high.

### Activation Assessment — 2026-04-29

**ELO data readiness evaluation:**

| Criterion | Required | Current | Met? |
|-----------|----------|---------|------|
| Seasons of ranking data | ≥ 2 | 1 (2025-26 DSS) | ❌ |
| Teams per division (minimum) | ≥ 20 | 24–75 | ✅ |
| Cross-division Elo gap stability (confirmed across seasons) | ≥ 2 seasons | unconfirmed | ❌ |
| Manual review workflow | Must exist | not built | ❌ |

**Verdict: DO NOT ACTIVATE.** Two of three hard criteria are unmet. With only one season of data (308 teams, 250 calibrated), within-division Elo variance reflects normal team spread rather than sandbagging patterns — the false-positive rate would be unacceptably high.

**Specific activation criteria:**

1. **≥ 2 seasons of managed-event data** — at least two DSS (or equivalent) cycles processed through the ranking engine, producing two complete division-placement → result cycles.
2. **Elo threshold gap confirmed stable** — median Elo gap between adjacent divisions exceeds 150 points across both seasons.
3. **Manual review workflow built** — a process exists for human review of flagged teams before any public flag is applied.

> [!info] Next Evaluation
> **Target re-evaluation: June 2026** — after the 2026 DSS data is processed. If 2025 and 2026 cycles are both in the system, criterion 1 is met. ELO will re-run this assessment in June 2026.
>
> **Target activation window (if criteria met): Summer 2026**, prior to 2026-27 fall event registration.

---

## Coverage

- **308 teams** with division placements (all from [[Dallas Spring Showcase]])
- **250 teams** successfully calibrated (had enough game history)
- **58 teams** with insufficient data for calibration
- Of those 58, **13 teams** had zero completed games — see [[Dallas Spring Showcase]]

---

## Related Notes
- [[Ranking Engine]] — Phase 9 where this calibration is applied
- [[Dallas Spring Showcase]] — the primary managed event using divisions
- [[League Hierarchy]] — league-level calibration (Phase 8)
- [[Data Quality]] — coverage gaps affecting calibration
