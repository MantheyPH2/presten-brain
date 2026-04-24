---
title: Boys Calibration Option A — Interpretation Guide
tags: [calibration, boys, validation, rankings, dss, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: ELO Agent
status: ready-to-use
related_task: task-2026-04-24-boys-calibration-option-a-interpretation
related_doc: "[[Boys Calibration Gap Analysis — April 2026]]"
---

# Boys Calibration Option A — Interpretation Guide

> Run the three queries from [[Boys Calibration Gap Analysis — April 2026]] Section 3 in a single ~15-minute psql session. This guide tells you what each result means, what thresholds to compare against, and how to reach a Boys DSS go/no-go in under 20 minutes.
>
> **Prerequisites:** Complete the DSS Data Readiness Validation Plan session (May 14). Add these three queries to that same session — they share the same DB and take ~5 minutes total.

---

## Section 1: Query A — Top-10 U13B Spot Check

**What to run:**

```sql
SELECT t.name, r.rating::int, r.rank, r.age_group, r.gender
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'M'
  AND r.rank <= 10
ORDER BY r.rank;
```

Also run for U14B:

```sql
SELECT t.name, r.rating::int, r.rank
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U14' AND r.gender = 'M' AND r.rank <= 10
ORDER BY r.rank;
```

---

**What indicates healthy calibration:**

- At least 5 of the 10 teams are recognizable Boys clubs from major national programs. Examples: FC Dallas Academy, Philadelphia Union Academy, New York Red Bulls Academy, LA Galaxy Academy, Chicago Fire Academy, Seattle Sounders Academy, Revolution (New England) Academy, LAFC Academy, NYC FC Academy, or top regional ECNL Boys clubs.
- All top-10 ratings are above **1,400**. Teams competing at MLS NEXT / ECNL level with sufficient game history should have accumulated calibration bonuses well above the 1,000 initialization floor.
- No single team has a rating above **1,700**. A rating that high is typically a data artifact — either a bad merge that inflated game history, or a team that played only in very high-calibration events with a small game count.

---

**Warning signs — flag to SENTINEL if any of these occur:**

| Warning | What it suggests | Immediate action |
|---------|-----------------|------------------|
| Majority of top-10 from a single region (e.g., all Northeast) | Source coverage imbalance — national rankings may reflect data availability, not true national quality | Check game volume per region using Query C |
| Any top-10 team with fewer than **10 games** | Rating is noise-dominated at low game counts; calibration bonus can distort a 3–5 game record | Run game count check: `SELECT COUNT(*) FROM games WHERE home_team_id = [id] OR away_team_id = [id]` |
| Any team name that looks regional or recreational (e.g., a town-name club, not a recognized Academy) in rank ≤5 | Possible bad merge or coverage artifact inflating an obscure team | Check for erroneous merges: `SELECT * FROM team_merges WHERE canonical_team_id = [id]` — if merged game count >> individual histories, investigate |
| A team with rating >1,600 that you don't recognize | Likely merge artifact or coverage outlier | Same merge check as above; flag the team name and rank to SENTINEL |

---

**If the top-10 looks obviously wrong:**

1. Run the game count check on any suspicious team (above).
2. Check for bad merges on the suspicious team (above).
3. Document the finding: team name, rank, rating, game count, merge status.
4. Flag to SENTINEL: "Top-10 U13B anomaly: [team name] at rank [X] with [Y] games. Possible merge artifact."
5. **Do not attempt to fix the merge during the May 14 validation session** — add to post-DSS triage queue.

---

## Section 2: Query B — Boys vs Girls Rating Distribution

**What to run:**

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

Also run for U14:

```sql
SELECT gender, AVG(rating)::int AS avg_rating, ROUND(STDDEV(rating), 1) AS rating_std,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating)::int AS median, COUNT(*) AS team_count
FROM rankings WHERE age_group = 'U14' AND rank IS NOT NULL
GROUP BY gender;
```

---

**What indicates healthy calibration:**

- Boys and Girls medians are within **75 points** of each other. Both should center near the system baseline (~1,050–1,150 for well-populated age groups).
- Standard deviations are within **75 points** of each other. A much wider Boys spread could indicate a small number of MLS NEXT teams with very high ratings pulling the distribution, which is acceptable if team_count is large enough.
- Boys `team_count` is at least **200**. Below this, the distribution statistics are unstable and cannot be used as calibration evidence.

---

**Warning thresholds:**

| Metric | Acceptable Range | Warning Threshold | Likely Cause If Triggered |
|--------|-----------------|-------------------|--------------------------|
| Boys avg_rating vs Girls avg_rating | Within 75 pts | > 100 pts divergence | Top-tier calibration asymmetry; Boys MLS NEXT (160) pulls Boys top teams higher than Girls GA (140) pulls Girls — some divergence is expected; >100pts is structural |
| Boys stddev vs Girls stddev | Within 75 pts | > 125 pts divergence | Boys distribution is bimodal (MLS NEXT cluster vs bottom of table); not necessarily miscalibration |
| Boys median vs Girls median | Within 75 pts | > 100 pts divergence | More diagnostic than avg; if median diverges >100pts, calibration is producing systematically different scales |
| Boys team_count | ≥ 200 | < 100 | Insufficient Boys data for this age group; Boys rankings are coverage-limited |

---

**If Boys average is >100 points below Girls:**

1. Pull Boys GA calibration value: `ga (Boys) = 100` (from [[League Hierarchy]]).
2. Pull Girls GA calibration value: `ga (Girls) = 140`.
3. Note: Boys GA (100) is lower than Girls GA (140) by design — Boys top-tier calibration runs through MLS NEXT (160). A lower Boys average is partly expected.
4. If the divergence exceeds 100 points AND Boys team_count is sufficient (>200), document: *"Boys U13 median is N pts below Girls. Some divergence is expected (Boys GA=100 vs Girls GA=140). Divergence at this level may indicate source coverage gap (fewer Boys games from high-cal tiers) rather than miscalibration. Recommend post-DSS Boys Brier analysis to confirm."*

**If Boys average is >100 points ABOVE Girls:**

This is unexpected. Boys MLS NEXT (160) is higher than Girls top tier (140), but both are bounded by the same K-factor and RD floor. An above-Girls average for Boys would suggest either (a) Boys have more MLS NEXT game coverage than Girls have GA/ECNL coverage, or (b) a calibration anomaly. Flag to SENTINEL.

---

## Section 3: Query C — Boys GA and MLS NEXT Game Volume

**What to run:**

```sql
SELECT
  e.event_tier AS tier,
  COUNT(DISTINCT g.id) AS game_count,
  COUNT(DISTINCT g.home_team_id) + COUNT(DISTINCT g.away_team_id) AS unique_team_appearances
FROM games g
JOIN events e ON e.id = g.event_id
WHERE g.age_group IN ('U13', 'U14')
  AND g.gender = 'M'
  AND e.event_tier IN ('ga', 'ga_aspire', 'mlsnext', 'ecnl', 'mlsnext_academy', 'mlsnext_homegrown')
  AND g.is_hidden = false
GROUP BY e.event_tier
ORDER BY game_count DESC;
```

---

**Minimum meaningful volume thresholds:**

| Tier | Minimum for Calibration to be Meaningful | Below This: |
|------|------------------------------------------|-------------|
| mlsnext (or mlsnext_academy + mlsnext_homegrown combined) | ≥ 500 games (U13+U14 combined) | Boys MLS NEXT calibration is noise-sensitive |
| ecnl | ≥ 300 games | ECNL Boys baseline (120) is speculative |
| ga (Boys) | ≥ 200 games | Boys GA calibration (100) is applied to thin data |
| ga_aspire (Boys, if tier exists) | ≥ 100 games | Separate Boys GA ASPIRE calibration is not yet warranted |

---

**Interpretation by scenario:**

**Boys MLS NEXT game count ≥ 500:**
Strong signal. The +160 / +135 calibration values are backed by substantial data. Boys U15–U17 rankings for MLS NEXT teams are well-grounded. Boys U13B and U14B have lower MLS NEXT density (teams qualify to MLS NEXT at U13 but peak enrollment is U15–U17), so counts around 300–500 are normal for those age groups.

**Boys MLS NEXT game count < 300 for U13B+U14B combined:**
Low signal for these age groups specifically. MLS NEXT calibration is still being applied but with fewer samples to validate it. Boys U13 and U14 rankings will trend accurate once more game data accumulates. Do not claim "comprehensive national Boys rankings" for these age groups at DSS unless coverage is notably strong.

**Boys GA game count < 200:**
The GA calibration value (100) is being applied to a thin Boys dataset. Individual Boys GA team ratings are more volatile than their Girls GA counterparts. Do not show Boys GA-heavy teams in the DSS demo — their ratings carry higher uncertainty.

**Boys GA game count is < 20% of Girls GA game count:**
Source coverage for Boys GA is significantly imbalanced. Boys rankings in the GA tier reflect fewer games and may not represent the full Boys GA competitive picture. Add a coverage caveat if Boys rankings are shown at DSS.

---

**Immediate action if Boys MLS NEXT volume is < 300:**

Do not propose Boys-specific MLS NEXT calibration changes before DSS. The data is insufficient to validate a split value for Boys-only analysis at U13/U14.

Add to coverage gap notes: *"Boys MLS NEXT U13+U14 game coverage: [N] games — below 300 threshold. Boys rankings in MLS NEXT tier reflect limited sample. Academy/Homegrown split improvement is projected but not Boys-Brier-validated at this age group."*

---

## Section 4: Go/No-Go Summary for Boys at DSS

After running all three queries, complete this status block. Add it to the bottom of [[Boys Calibration Gap Analysis — April 2026]].

```
Boys Calibration DSS Status — [date run]

Query A (Top-10 U13B): [PASS / WARN / FAIL]
  Finding: [list top-3 team names and whether they're recognizable]
  Rating range (top 10): [min]–[max]

Query B (Distribution U13B): [PASS / WARN / FAIL]
  Boys avg: [X] | Girls avg: [Y] | Divergence: [Z pts]
  Boys median: [X] | Girls median: [Y]
  Boys team_count: [N]

Query C (Game volume): [PASS / WARN / FAIL]
  MLS NEXT (all Boys U13+U14): [N] games
  GA Boys (U13+U14): [N] games
  ECNL Boys (U13+U14): [N] games

Overall: [SAFE to show Boys at DSS / CAUTION — note below / DO NOT show Boys rankings at DSS]

Notes: [any warnings or specific findings]
```

**Decision rules:**

| Result combination | Decision |
|---|---|
| All three PASS | **SAFE** — Boys rankings can be shown at DSS if a club director asks |
| Query A WARN, B and C PASS | **CAUTION** — investigate the anomalous top-10 team before DSS; if it's a known artifact, proceed |
| Query B WARN only (distribution divergence) | **CAUTION** — note the expected divergence rationale (Boys MLS NEXT vs Girls GA); show Boys rankings but avoid direct Boys/Girls comparison claims |
| Query C WARN (low volume) | **CAUTION** — do not anchor Boys demo on MLS NEXT or GA specific claims; stick to general ranking narrative |
| Query A FAIL (top-10 looks wrong) | **DO NOT show Boys rankings at DSS** — visible calibration error in the most DSS-relevant age group |
| Multiple WARNs or any FAIL | **Escalate to SENTINEL** before May 14 |

---

## References

- [[Boys Calibration Gap Analysis — April 2026]] — the three queries (source); add Section 4 status block here after running
- [[Ranking Engine]] — Boys Phase 8 calibration values per tier
- [[League Hierarchy]] — Boys and Girls calibration values with status
- [[DSS Demo Validation Criteria — May 2026]] — Girls U13G pass/fail structure (parallel document for Girls)
