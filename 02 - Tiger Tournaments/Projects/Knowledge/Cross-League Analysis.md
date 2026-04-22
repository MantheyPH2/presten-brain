---
title: Cross-League Analysis
aliases: [Win Rate Analysis, League Comparison, 575K Study]
tags: [knowledge, analysis, cross-league, win-rates, evo-draw]
created: 2026-04-21
updated: 2026-04-21
games_analyzed: 575000
---

# Cross-League Analysis

Results from our analysis of **575K+ games** comparing team performance across league boundaries. This data validates the [[League Hierarchy]] tier assignments and [[Ranking Engine]] calibration values.

---

## Key Head-to-Head Win Rates

| Matchup                      | Win Rate | Interpretation                    |
| ---------------------------- | -------- | --------------------------------- |
| [[MLS NEXT]] vs all others   | Dominant | Confirms Tier 1 #1 boys status   |
| [[ECNL RL]] vs NPL          | **55.1%**| Confirms Tier 2 peer status      |
| [[Pre-ECNL]] vs NPL         | **68.4%**| Surprisingly strong for Tier 3   |
| State Cup vs NPL             | **51.4%**| Very competitive — near parity   |

---

## Detailed Findings

### ECNL RL vs NPL: 55.1%
- [[ECNL RL]] beats NPL in 55.1% of cross-league matchups
- This is a statistically significant but moderate advantage
- **Conclusion:** ECNL RL and NPL are correctly classified as Tier 2 peers
- Both receive **55-point calibration** in the [[Ranking Engine]]
- The 5% edge doesn't warrant a calibration difference

### Pre-ECNL vs NPL: 68.4%
- [[Pre-ECNL]] beats NPL in 68.4% of cross-league matchups
- This is a surprisingly dominant result for a Tier 3 program

> [!info] Why Pre-ECNL Overperforms
> Pre-ECNL clubs are the youth arms of [[ECNL]] member clubs — elite organizations investing in development from U10. The 68.4% reflects **organizational quality** rather than tier equivalence. These are younger players (U10-U12) from top clubs, not older players at a higher competitive tier. The calibration stays at 25 because the age/developmental factor dominates.

### State Cup vs NPL: 51.4%
- State Cup teams beat NPL at 51.4% — essentially a coin flip with a slight edge
- Suggests State Cup finalists are at Tier 2 level
- Supports the [[League Hierarchy]] placing state leagues at Tier 3 overall (with top teams reaching Tier 2)

### MLS NEXT Dominance
- [[MLS NEXT]] dominates all other tiers in head-to-head play
- Validates the highest calibration value (160 points)
- The gap between MLS NEXT (160) and ECNL (120) is the largest in the system and is empirically justified

---

## Calibration Validation

The win rates confirm the [[League Hierarchy|calibration values]] are well-set:

| League Pair        | Win Rate Gap | Calibration Gap | Aligned? |
| ------------------ | ------------ | --------------- | -------- |
| MLS NEXT vs ECNL   | Large        | 40 (160→120)    | Yes      |
| ECNL vs ECNL RL    | Moderate     | 65 (120→55)     | Yes      |
| ECNL RL vs NPL     | 5.1%         | 0 (55→55)       | Yes      |
| NPL vs Pre-ECNL    | -18.4%       | 30 (55→25)      | Yes*     |

*Pre-ECNL's overperformance is explained by organizational quality at younger ages, not tier equivalence.

---

## Methodology

- Dataset: 575K+ visible games (`is_hidden=false`)
- Cross-league games identified by team [[Dedup Strategy|merges]] and event tier classification
- Win rate = (Team A wins + 0.5 * draws) / total games
- Minimum sample size thresholds applied per matchup

---

## Related Notes
- [[League Hierarchy]] — tier definitions validated by this analysis
- [[Ranking Engine]] — calibration values derived from these findings
- [[ECNL RL]] — 55.1% win rate vs NPL
- [[Pre-ECNL]] — 68.4% win rate vs NPL
- [[Competitors]] — this analysis is a key differentiator (Gap #1)
- [[Data Quality]] — completeness of the underlying data
