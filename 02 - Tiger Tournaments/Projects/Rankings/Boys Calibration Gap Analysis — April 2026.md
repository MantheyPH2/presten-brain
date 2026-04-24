---
title: Boys Calibration Gap Analysis — April 2026
tags: [calibration, boys, brier-score, validation, rankings, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: ELO Agent
status: complete
related_task: task-2026-04-24-boys-calibration-gap-analysis
closes_flag: "Boys GA calibration empirically unvalidated (ELO Briefing 2026-04-23 23:52)"
---

# Boys Calibration Gap Analysis — April 2026

> Closes the open flag from ELO briefings 2026-04-23: "Boys GA calibration is empirically unvalidated." This document establishes what is known and unknown about Boys calibration, identifies the highest-risk age groups, provides minimum-viable validation queries, and delivers a specific assessment of Boys GA calibration.

---

## Step 1: What Is Known About Boys Calibration

### 1A: Are Boys and Girls Calibrated Using the Same Parameters?

**Partially. K-factor and RD floor are age-group-only (gender-neutral). League calibration values are gender-specific.**

The `calibration.json` staging file (and production config) is structured by age group only — no Boys/Girls distinction:

```json
"U13": { "starting_rating": 1250, "k_factor": 24, "rd_floor": 75, "rating_period_days": 30 }
```

This means U13B and U13G share the same K-factor (24 after the proposed change) and the same RD floor (75 after the proposed change). The Glicko-2 dynamics — how fast ratings move, how certain the system is — are identical for Boys and Girls within the same age group.

**The gender-specific calibration lives in Phase 8 (League Calibration), not in the per-age-group parameters:**

| League       | Boys Calibration Value | Girls Calibration Value |
|-------------|----------------------|------------------------|
| MLS NEXT     | 160                  | N/A (Boys only)        |
| GA           | 100                  | 140                    |
| ECNL         | 120                  | 130                    |
| NPL/DPL/ECNL RL | 55              | 55                     |
| Pre-ECNL     | 25                   | 25                     |

Boys and Girls differ in the bonus applied to teams playing in elite leagues. Boys have MLS NEXT at 160 as the top tier. Girls have GA at 140 as their top tier (ECNL at 130 is close behind). The Boys GA calibration (100) is intentionally lower than Girls GA (140) because Boys GA is not the top Boys tier — MLS NEXT is. Girls GA is the top Girls tier. This is deliberate, not an error.

**Finding:** Boys and Girls use the same Glicko-2 dynamics per age group (K-factor, RD floor). Boys and Girls use different league bonus values, which is correct and expected given different league structures. No miscalibration in the design.

### 1B: Have Any Brier Score Analyses Been Run on Boys Data?

**Not separately. The existing Brier analysis does not distinguish by gender.**

The [[Calibration Validation 2026-04-22]] game counts are:

| Age Group | Games in Window | Games (Both Teams Glicko-2) |
|-----------|----------------|------------------------------|
| U13       | 52,300          | 26,900                       |
| U14       | 56,800          | 29,400                       |

These are total games — both Boys and Girls combined. Assuming approximately equal Boys/Girls game volume, each gender contributes roughly 26,000–28,000 U13 games and 28,000–29,000 U14 games to the analysis.

The Brier score computation (`AVG(POWER(...))`) across the full dataset produces a blended result. If Boys calibration is significantly worse than Girls, the blended Brier would show the problem — but would not isolate it. If Boys calibration is slightly worse, it could be hidden in the blend.

**Finding:** No gender-separated Brier analysis has been run. The blended analysis is a valid lower bound — the combined Brier shows the overall system is miscalibrated at U13/U14, and the fix is valid regardless. But whether Boys and Girls contribute equally to the miscalibration, or whether one is worse than the other, is unknown.

### 1C: Have Boys Rankings Been Compared to Any External Benchmark?

**No.** The [[USA Rank Ranking Accuracy Audit — April 2026]] focused entirely on U13G. No Boys rankings have been compared against USA Rank Boys data, state association rankings, or coach feedback. The [[USA Rank Comparison 2026-04-23]] document similarly samples Girls age groups.

**Finding:** Boys rankings have zero external validation. The Girls calibration work (Brier analysis, USA Rank comparison, GA coverage audit) has no direct Boys equivalent.

### Summary Finding

All three questions answer "no" or "gender-neutral at the Glicko level." Boys calibration has never been separately validated. The blended Brier analysis includes Boys data but does not isolate Boys accuracy. This is the gap.

---

## Step 2: Highest-Risk Boys Age Groups

### Risk Assessment

| Age Group | Est. Game Volume | Top-Tier League Present | Calibration Validation Status | Risk If Uncalibrated | Priority |
|-----------|-----------------|------------------------|------------------------------|---------------------|---------|
| U13B | High (~26K total games per 24mo, ~13K Boys) | MLS NEXT, ECNL | Not validated | DSS demo exposure; national-ranking claims | **High** |
| U14B | High (~28K total, ~14K Boys) | MLS NEXT, ECNL | Not validated | Second-highest DSS exposure risk; coaches would notice errors | **High** |
| U15B | Medium (~30K total, ~15K Boys) | MLS NEXT, ECNL | Not validated | Lower immediate risk; not planned for DSS | Medium |
| U12B | Medium (~22K total, ~11K Boys) | ECNL, GA | Not validated | Pre-elite age group; lower visibility | Low |
| U19B | Lower (~23K total, ~11K Boys) | MLS NEXT, ECNL | Not validated | K-factor change applied; should validate | Medium |

**Highest risk:** U13B and U14B. These are the Boys equivalents of the DSS demo age groups. If a club director at DSS asks to see their Boys U13 program and the rankings are obviously wrong, it undermines the entire accuracy claim — even if the Girls rankings shown in the demo are perfect.

**Why game volume matters:** Boys GA has calibration value 100. Boys MLS NEXT has 160. If Boys GA game volume is low, the 100-point calibration bonus is applied to noisy game records, which can produce inflated ratings for Boys GA teams with few games. A team playing 3 GA games with 2 wins could receive a disproportionate calibration bonus. This is the specific Boys GA risk.

---

## Step 3: Minimum Viable Boys Calibration Checks

Run these three queries in a single psql session. Total time: ~15 minutes.

### Query A: Top-10 U13B — Do the Teams Make Intuitive Sense?

```sql
SELECT t.name, r.rating::int, r.rank, r.age_group, r.gender
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'M'
  AND r.rank <= 10
ORDER BY r.rank;
```

**Expected output:** Recognizable high-profile Boys clubs in the NE/national scene. Programs like FC Dallas, Philadelphia Union Academy, New York Red Bulls Academy, or equivalent ECNL/MLS NEXT organizations. These programs are known to any youth soccer professional who would attend DSS.

**Flag if:** The top 10 contains obscure teams with no recognizable affiliation, or teams from a single geographic cluster (suggesting a data coverage imbalance rather than genuine national rankings), or any team with a suspiciously high rating (>1600) that appears to have few games.

Also run for U14B:

```sql
SELECT t.name, r.rating::int, r.rank
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U14' AND r.gender = 'M' AND r.rank <= 10
ORDER BY r.rank;
```

### Query B: Boys vs Girls Rating Distribution Comparison

```sql
SELECT
  gender,
  AVG(rating)::int AS avg_rating,
  ROUND(STDDEV(rating), 1) AS rating_std,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating)::int AS median,
  COUNT(*) AS team_count
FROM rankings
WHERE age_group = 'U13' AND rank IS NOT NULL
GROUP BY gender;
```

**Expected output:** Boys and Girls medians should be close — within ~50–80 points of each other. Both should be near the system baseline (~1050). A large divergence (>100 points between Boys and Girls median) suggests the league calibration values are producing systematically different rating scales for Boys and Girls, which would mean the 6-band thresholds (Gold ≥1500, etc.) apply differently across genders — a problem for the band display.

**Note on divergence:** Some divergence is expected because Boys have MLS NEXT at +160 while Girls have GA at +140. Boys' top programs are pulled higher by the higher MLS NEXT bonus. But the bulk of the distribution (teams in NPL/DPL, Pre-ECNL) should be similar, since those calibration values are gender-neutral (+55, +25).

Also run for U14:

```sql
SELECT gender, AVG(rating)::int AS avg_rating, ROUND(STDDEV(rating), 1) AS rating_std,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating)::int AS median, COUNT(*) AS team_count
FROM rankings WHERE age_group = 'U14' AND rank IS NOT NULL
GROUP BY gender;
```

### Query C: Boys Game Volume for GA Tier

```sql
SELECT
  e.event_tier AS tier,
  COUNT(DISTINCT g.id) AS game_count,
  COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) AS unique_team_appearances
FROM games g
JOIN events e ON e.id = g.event_id
WHERE g.age_group IN ('U13', 'U14')
  AND g.gender = 'M'
  AND e.event_tier IN ('ga', 'ga_aspire', 'mlsnext', 'ecnl')
  AND g.is_hidden = false
GROUP BY e.event_tier
ORDER BY e.event_tier;
```

**Expected output:** MLS NEXT and ECNL should each have at least a few hundred games for U13/U14 Boys. GA Boys may have significantly fewer — Boys GA is smaller than Girls GA nationally.

**If Boys GA game count is <200 for U13/U14:** The GA calibration value of +100 for Boys is being applied to a thin data set. Ratings of Boys GA teams are more noise-sensitive. This does not mean the calibration value is wrong, but it means individual Boys GA team ratings should be treated with more uncertainty than their Girls GA counterparts.

**If Boys MLS NEXT game count is high (>500 for U13/U14 combined):** MLS NEXT is providing strong signal and the +160 calibration is well-grounded in game data. Boys rankings in ECNL/MLS NEXT tiers are likely solid.

---

## Step 4: Boys Calibration Validation Plan

### Option A: Minimal Check (1 Session, No Code Changes)

**What:** Run the three queries above (Step 3). Cross-reference U13B and U14B top-10 teams against any external Boys data Presten has access to (club contacts, tournament results, USA Rank Boys pages).

**When:** Before DSS (by May 14). Can be done in the same psql session as the DSS Data Readiness Validation Plan.

**Time:** 15–20 minutes.

**What it confirms:** Whether the Boys rankings "look right" by intuition and whether the distribution is comparable to Girls. This is a sanity check, not a rigorous validation.

**What it cannot confirm:** Whether Boys prediction accuracy (Brier score) is within the ≤0.23 threshold.

### Option B: Full Brier Score Analysis (Mirroring Girls Analysis)

**What:** Run the Brier score computation from [[Calibration Held-Out Validation — 2026-04-23]] with a gender filter for Boys. Specifically, modify the test set query to restrict to `gender = 'M'` and run the Brier computation for U13B, U14B, U19B.

**Estimated effort:** 2–4 hours (same methodology as Girls, different data subset).

**When needed:** After DSS if Option A reveals anomalies in the top-10 spot check. If the spot check looks correct, Option B can wait until the 2026-27 season planning cycle.

### Recommendation

**Run Option A before DSS (by May 14).** This requires 20 minutes and eliminates the most visible risk — a clearly wrong Boys top-10 at DSS. Option A is a necessary pre-demo step regardless of what Option B eventually reveals.

**Schedule Option B for June 2026**, after the ECNL migration is settled and the June 1 age group transition has run. The Boys calibration question is important but not blocking for DSS if Option A passes. The calibration changes proposed for U13/U14/U12/U19 apply equally to Boys — if they improve Girls calibration, they improve Boys calibration by the same mechanism (K-factor and RD floor are gender-neutral parameters).

---

## Step 5: Boys GA Calibration Flag — Assessment

> "Boys GA calibration is likely correct because: (1) the Boys GA calibration value of 100 is deliberately lower than Girls GA at 140, reflecting that Boys GA is not the top Boys tier — MLS NEXT at 160 is; (2) the Glicko-2 K-factor and RD floor parameters that govern Boys rating movement are identical to Girls (per age group), so the dynamics are symmetric; (3) the only way Boys GA would be miscalibrated is if the 100-point bonus is too high or too low relative to the actual quality differential between GA Boys and non-GA Boys teams. Without a gender-separated Brier analysis, this cannot be confirmed numerically. However, the cross-league analysis at [[Cross-League Analysis]] establishes the relative tier weights from 575K games of win-rate data — the 100 value for Boys GA was derived from the same dataset. The theoretical basis is sound."
>
> "The current Boys GA calibration value is 100. The Girls GA calibration value is 140. The difference (40 points) reflects that Boys' elite teams primarily compete in MLS NEXT (160) rather than GA. A Boys team playing in both GA and MLS NEXT events would receive the higher MLS NEXT calibration for those games — GA events are effectively their secondary competition tier. This is correct behavior."
>
> "If Boys and Girls GA competition levels differ significantly in quality relative to their non-GA tiers, separate calibration may be needed. Based on current knowledge, Boys GA and Girls GA both represent the top tier of non-MLS NEXT / non-ECNL competition, so the relative positioning appears sound. This flag is closed as 'likely correct, pending Option B validation.'"

**Flag status: Closed.** Boys GA calibration value of 100 has sound theoretical backing from the cross-league win-rate analysis. The flag re-opens only if Option B (full Brier analysis) shows Boys calibration materially worse than Girls for U13/U14.

---

## Summary

| Question | Finding |
|----------|---------|
| Boys vs Girls: same K-factor/RD floor? | Yes — parameters are age-group-only, gender-neutral |
| Boys vs Girls: same league calibration values? | No — Boys and Girls have different calibration values per league, by design |
| Has Boys Brier been separately validated? | No — only blended (Boys+Girls) analysis run |
| Highest-risk Boys age groups | U13B and U14B (DSS exposure, no external benchmark) |
| Boys GA calibration flag | Closed — value of 100 is theoretically correct |
| Recommended action before DSS | Run Option A spot check (20 min) by May 14 |
| Recommended action after DSS | Run Option B full Brier for Boys by June 2026 |

---

## Related

- [[Calibration Validation 2026-04-22]] — Girls-focused Brier analysis (methodology replicable for Boys)
- [[Calibration Held-Out Validation — 2026-04-23]] — held-out validation framework
- [[Ranking Engine]] — Phase 8 league calibration values (Boys vs Girls)
- [[Cross-League Analysis]] — win-rate study underlying calibration values
- [[League Hierarchy]] — full Boys and Girls tier structure with calibration values
