---
title: Division Calibration
aliases: [Division Placement, DSS Divisions, Anti-Sandbagging]
tags: [rankings, calibration, divisions, dss, evo-draw]
created: 2026-04-21
updated: 2026-04-21
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
>
> Future activation criteria:
> - At least 2 seasons of ranking data
> - Clear Elo threshold gaps between divisions
> - Manual review workflow for flagged teams

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
