---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
priority: medium
due: 2026-04-28
deliverable: "02 - Tiger Tournaments/Projects/Rankings/Boys Calibration Option A — Interpretation Guide.md"
---

# Task: Boys Calibration Option A — Query Interpretation Guide

## Context

ELO completed the Boys Calibration Gap Analysis (`Rankings/Boys Calibration Gap Analysis — April 2026.md`) and wrote three minimum-viable validation queries (Option A). These queries are ready for Presten to run in a ~15-minute psql session.

What does not exist: a guide for what the results mean. Presten will run the queries, get numbers back, and have no calibrated sense of whether those numbers indicate healthy rankings or a serious problem. Without interpretation criteria, Presten will either:
- Assume the results are fine and move on (potentially missing a calibration problem)
- Flag everything as suspicious and create unnecessary remediation work

This task: write the interpretation guide so that each query result is immediately actionable. Due April 28 — the same day SENTINEL has set as the GA ASPIRE escalation deadline. Both Boys validation and Girls GA fix should be resolved on the same session.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Rankings/Boys Calibration Option A — Interpretation Guide.md`

---

### Section 1: Query A — Top-10 U13B Spot Check

**What Presten will see:** A table of 10 team names, ratings, and ranks.

**What indicates healthy calibration:**
- At least 5 of the 10 teams are recognizable competitive Boys clubs from major programs (FC Dallas, Chicago Fire Academy, Revolution Academy, LAFC, NYC FC Academy, etc.)
- Ratings are above 1400 for all 10 (consistent with the Gold/high-Silver band that top national programs should occupy)
- No single team has a disproportionately high rating (e.g., >1700) suggesting a data artifact or merge error

**Warning signs requiring ELO review:**
- Majority of top-10 teams are from a single geographic region (e.g., all Northeast) — may indicate source coverage bias, not true national quality signal
- Any team with fewer than 10 games in the top 10 — rating is noise-dominated at that game count
- Any team name that appears regional or low-tier (e.g., a rec league or local club) at rank ≤5

**Immediate action if result looks wrong:**
1. Run the game count check: `SELECT COUNT(*) FROM games WHERE (home_team_id = [id] OR away_team_id = [id])` for any suspicious top-10 team
2. Check for bad merges: `SELECT * FROM team_merges WHERE canonical_team_id = [id]` — a high rating from a merged team with inflated game count is a common artifact
3. Flag to SENTINEL: team name, rank, rating, game count, merge status

---

### Section 2: Query B — Boys vs Girls Rating Distribution

**What Presten will see:** Two rows — one for gender='M', one for gender='F' — showing avg_rating, stddev, and team_count for U13.

**What indicates healthy calibration:**
- Average ratings within 75 points of each other (both should center around similar values since Glicko-2 initializes all teams at 1000 and the calibration multiplier is the only systematic offset)
- Standard deviations within 50 points of each other — similar spread indicates similar calibration spread
- Team counts: Boys count expected to be somewhat lower than Girls (fewer Boys circuits in current sources)

**Warning thresholds (flag to SENTINEL if any of these trigger):**

| Metric | Acceptable Range | Warning Threshold |
|--------|-----------------|-------------------|
| Boys avg_rating vs Girls avg_rating | Within 75 pts | > 100 pts divergence |
| Boys stddev vs Girls stddev | Within 75 pts | > 125 pts divergence |
| Boys team_count | ≥ 200 | < 100 (insufficient data for calibration) |

**Why these thresholds:** Boys and Girls use the same Glicko-2 initialization (1000) and the same K-factor. The calibration multipliers differ by tier but should produce similar overall distributions if the tier compositions are comparable. A >100pt average divergence most likely indicates one gender's top tier is significantly over- or under-calibrated.

**Immediate action if Boys average is >100pts below Girls:**
- Pull Boys GA calibration value from Ranking Engine or League Hierarchy
- Compare to Girls GA calibration value
- If values are the same AND Boys avg is lower, the issue is likely source coverage (fewer Boys games → fewer opportunities to accumulate rating points), not miscalibration
- Document the finding: "Boys average is N pts below Girls. Likely source gap, not calibration error. Recommend post-DSS Boys Brier analysis."

---

### Section 3: Query C — Boys GA Game Volume

**What Presten will see:** Rows for 'ga', 'ga_aspire', 'mlsnext' tiers showing game_count and unique_teams for U13B.

**What indicates healthy calibration:**

| Tier | Minimum Meaningful Volume | Below This: |
|------|--------------------------|-------------|
| ga | ≥ 500 games | Calibration is noise-dominated |
| ga_aspire | ≥ 200 games | Insufficient to validate the tier's calibration value |
| mlsnext | ≥ 300 games | Calibration value is speculative |

**Warning signs:**
- Any Boys tier showing < 100 games: the calibration value for that tier is essentially arbitrary — either skip calibrating it or acknowledge it's guesswork
- Boys GA game count is less than 20% of Girls GA game count: source coverage is highly imbalanced; Boys rankings in this tier should carry a coverage caveat at DSS

**Immediate action if Boys GA is <500 games:**
- Do not propose Boys-specific calibration changes before DSS
- Add to coverage gaps table: "Boys GA game coverage insufficient for independent calibration — combined Boys/Girls calibration value in use"
- Flag to SENTINEL if you believe this gap materially affects Boys rankings accuracy

---

### Section 4: Go/No-Go Summary for Boys at DSS

Based on the three query results, complete this summary and add it to the bottom of the Boys Calibration Gap Analysis document:

```
Boys Calibration DSS Status — [date run]

Query A (Top-10 U13B): [PASS / WARN / FAIL] — [one sentence on what was found]
Query B (Distribution): [PASS / WARN / FAIL] — [divergence: X pts avg, Y pts stddev]
Query C (Game volume): [PASS / WARN / FAIL] — [GA: N games, GA Aspire: N games, MLSNEXT: N games]

Overall: [SAFE to show Boys at DSS / CAUTION — note the following / DO NOT show Boys rankings at DSS]
```

This summary is what SENTINEL needs to close the Boys calibration validation flag.

---

## Definition of Done

- Interpretation guide written at the specified path
- Each query section has: what to look for, specific numeric thresholds, warning signs, and immediate action steps
- Section 4 DSS summary template is ready to fill in
- Document is self-contained (Presten can run queries, read this guide, and reach a go/no-go without ELO follow-up)
- Summary in briefing: "Boys calibration Option A interpretation guide written. Ready for Presten to run."

## Why This Matters

The Boys Calibration Gap Analysis wrote the queries. This document completes the loop — it tells Presten what to do with the results. Without interpretation criteria, Option A queries return data with no meaning. With this guide, a 15-minute psql session produces a definitive Boys DSS go/no-go.

## References

- `02 - Tiger Tournaments/Projects/Rankings/Boys Calibration Gap Analysis — April 2026.md` — the three queries to interpret (written here)
- `02 - Tiger Tournaments/Projects/Rankings/Ranking Engine.md` — Boys calibration values per tier (needed for Section 4 divergence reasoning)
- `02 - Tiger Tournaments/Projects/Rankings/DSS Demo Validation Criteria — May 2026.md` — Girls U13G pass/fail structure (mirror this format for Boys)
