---
title: USA Rank Delta Interpretation Spec — April 2026
type: interpretation-spec
author: ELO
date: 2026-04-24
status: ready
related: "[[USA Rank Ranking Accuracy Audit — April 2026]]", "[[League Hierarchy]]"
tags: [usa-rank, calibration, accuracy-audit, evo-draw]
---

# USA Rank Delta Interpretation Spec — April 2026

> When Presten runs FORGE's 16 audit queries, the output will be raw rating deltas between Evo Draw and USA Rank for overlapping teams. This document tells ELO what each delta range means and what action to take.

---

## Section 1: Methodology Differences (Expected Structural Deltas)

These are known differences between Evo Draw and USA Rank that will produce systematic rating gaps. These are NOT calibration errors.

### 1.1 Goal Differential Inclusion

- **What:** USA Rank likely incorporates goal differential into ratings. Evo Draw uses result-only (W/D/L) — a 1-0 win and a 5-0 win produce the same rating change.
- **Direction:** Teams that frequently win by large margins will be rated higher by USA Rank than by Evo Draw. Teams that narrowly win close games repeatedly will be rated similarly or higher by Evo Draw.
- **Magnitude estimate:** ±15–30 points for teams with consistently large or small goal margins. Near-zero for balanced teams.

### 1.2 Game Coverage Gaps

- **What:** Evo Draw has known coverage gaps documented in [[Source Gap Inventory — April 2026]]. Teams in uncovered leagues (certain state associations, some regional circuits) have fewer games in Evo Draw than in USA Rank.
- **Direction:** Under-covered teams will have ratings based on a smaller game sample — higher uncertainty (RD), slower convergence, more noise. Their Evo Draw rating may differ in either direction from USA Rank depending on which games are present.
- **Magnitude estimate:** Up to ±50–80 points for teams where coverage gaps are severe (>30% of games missing from Evo Draw sources).
- **How to identify:** Cross-reference with [[Source Gap Inventory — April 2026]]. If a team's delta is large AND the team is in a league with a known coverage gap, classify as coverage-driven, not calibration-driven.

### 1.3 Age Group Mapping Differences

- **What:** USA Rank may use birth year cutoffs that differ from Evo Draw's school-year-based age group boundaries. A team classified as U13G by Evo Draw may appear as U12 or U14 in USA Rank's system.
- **Direction:** Affects which games count toward a team's rating in cross-age-group competition.
- **Magnitude estimate:** ±10–20 points. Teams near age group boundaries are most affected.
- **How to identify:** If a large delta correlates with teams born near the August 1 cutoff, this is likely the cause.

### 1.4 Recency Weighting

- **What:** USA Rank's recency weighting methodology is unknown. Evo Draw applies Glicko-2 temporal decay — older games have lower RD and contribute less to current ratings.
- **Direction:** Teams that peaked 2+ seasons ago may rate lower in Evo Draw than USA Rank. Teams that are improving fast (recent hot streak) may rate higher in Evo Draw if USA Rank applies a longer historical window.
- **Magnitude estimate:** ±20–35 points for teams with non-uniform performance histories.

---

## Section 2: Delta Thresholds

These thresholds apply after accounting for expected structural differences (Section 1). If a delta is explained by Section 1 factors, classify as Expected — do not escalate.

| Delta Category | Range       | Interpretation                                               | Action                              |
|----------------|-------------|--------------------------------------------------------------|-------------------------------------|
| Expected       | 0–35 pts    | Within normal methodology variance                           | No action                          |
| Mild discrepancy | 36–60 pts | Worth noting; check game coverage first                      | Flag for source review              |
| Concerning     | 61–100 pts  | Likely calibration error if not coverage-driven              | ELO investigation required          |
| Critical       | > 100 pts   | Strong signal of systematic miscalibration or merge error    | Escalate to Presten before DSS      |

**Reference point:** The Girls GA ASPIRE overcalibration was ~40 points — enough to cause visible rank distortion and require a fix. A delta of 40 pts already falls in the Mild Discrepancy category; 60+ pts should be treated as Concerning.

**Important:** These thresholds apply to individual team deltas. Systematic patterns (e.g., all U13G teams rated lower than USA Rank by ~30 points) indicate a global calibration issue, even if individual deltas are small.

---

## Section 3: Investigation Protocol

When a team is flagged as Concerning or Critical:

**Step 1: Coverage check first.**
Run FORGE's Source Gap Inventory cross-reference. Is this team in a league with a known coverage gap? If yes: delta is likely coverage-driven. Classify as coverage gap, not calibration error. File in the source gap list for post-DSS resolution.

**Step 2: Merge check.**
Is this team present in `team_merges` as a merged record? A team incorrectly split across two records will have a suppressed rating in Evo Draw (each fragment holds only part of the game history). Check:
```sql
SELECT tm.*, t1.name, t2.name
FROM team_merges tm
JOIN teams t1 ON t1.id = tm.canonical_team_id
JOIN teams t2 ON t2.id = tm.merged_team_id
WHERE t1.name ILIKE '%[team name]%' OR t2.name ILIKE '%[team name]%';
```
If the team is fragmented: resolve merge before concluding calibration is wrong.

**Step 3: Opponent quality check.**
Pull the team's top 5 highest-rated opponents in Evo Draw. Do those opponent ratings look reasonable compared to USA Rank's assessment of the same opponents? If Evo Draw's opponent ratings are systematically lower, the delta may be a cascading effect of opponent miscalibration rather than this team's calibration specifically.

**Step 4: Calibration value check.**
Cross-reference the team's primary league in [[League Hierarchy]]. Is the calibration value validated or assumed? GA Boys (100) is theoretically sound but not Brier-validated. MLS NEXT (160) is validated. If the team primarily plays in an unvalidated-calibration league and has a large delta, flag for the Boys Brier analysis post-DSS.

**Step 5: Resolution classification.**
After Steps 1–4, classify the delta as one of:
- **Coverage-driven:** file in source gap list, no calibration action
- **Merge-driven:** fix the team_merges record, no calibration action
- **Calibration-driven:** propose specific calibration value change, flag to SENTINEL
- **Methodology difference:** document the structural reason, no action (expected)

---

## Section 4: Comparison Scope Limitations

**USA Rank coverage:**
- Girls: focuses heavily on ECNL and GA. Coverage of NPL and state leagues is variable.
- Boys: MLS NEXT and ECNL are primary. Regional coverage is thinner than for Girls.
- **USA Rank is NOT ground truth.** It is a peer comparison. Evo Draw's methodology is more transparent and auditable — we know our calibration weights and have Brier-score validation. USA Rank's methodology is a black box.

**Where Evo Draw is expected to be more accurate:**
- State league teams: Evo Draw sources USYS state leagues more comprehensively than USA Rank.
- Teams with long multi-season histories: Glicko-2 temporal decay rewards consistent multi-year performance; USA Rank's weighting of historical games is unknown.
- Cross-league comparability: Evo Draw has explicit calibration weights per tier validated by 575K game win-rate analysis. USA Rank has no published cross-league normalization method.

**Where USA Rank may outperform:**
- Recent form sensitivity: if USA Rank upweights last 90 days more aggressively than Glicko-2 decay, it may be more responsive to hot streaks.
- Goal differential signal: teams that dominate in score margin may be better captured by USA Rank.

**DSS framing:** At DSS, Evo Draw's advantage is depth of coverage and methodological transparency, not raw rank agreement with USA Rank. Where we agree on the top teams, that validates methodology. Where we disagree, we have a specific, defensible reason — calibration weights, game coverage, or merge correctness — not a black box.

---

## Section 5: Output Format for Post-Run Review

After Presten runs FORGE's 16 queries:

1. Paste raw output to: `02 - Tiger Tournaments/Projects/Reports/usarank-accuracy-audit-results-[YYYY-MM-DD].md`
2. File with frontmatter: `type: audit-results`, `date: [date]`, `status: pending-elo-review`
3. ELO will apply this spec to classify each delta within one working session.
4. Results will be summarized in ELO's next briefing with:
   - Count of teams in each delta category (Expected / Mild / Concerning / Critical)
   - Any Concerning or Critical teams identified by name with investigation findings
   - DSS recommendation: "Data is clean" / "Fix required for [X] teams" / "Coverage gap noted for [Y] teams"

---

## Related

- [[USA Rank Ranking Accuracy Audit — April 2026]] — the 16 queries this spec interprets
- [[League Hierarchy]] — calibration values by circuit
- [[Source Gap Inventory — April 2026]] — coverage gaps that explain some deltas
- [[Boys Calibration Gap Analysis — April 2026]] — Boys-specific calibration validation context
- [[Competitors]] — USA Rank scope and methodology description
