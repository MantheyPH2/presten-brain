---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: medium
due: 2026-05-02
---

# Task: Boys Calibration Gap Analysis — Identify Unvalidated Age Groups and Leagues

## Context

Every calibration analysis in the Evo Draw record focuses on Girls age groups. The Brier score validation covers U12G, U13G, U14G, U19G. The DSS Demo Spec anchors on U13G. The USA Rank comparison samples are U13G and U14G.

Boys calibration has been mentioned exactly once — ELO's GA coverage audit flagged "Boys GA calibration is empirically unvalidated" in the 23:52 briefing. That flag has not been actioned. SENTINEL has carried it in every briefing since.

The gap: ELO has done rigorous calibration work on Girls age groups, but it is unknown whether Boys rankings are accurate. If Boys calibration is significantly off, then:
1. Any Boys age group shown at DSS could embarrass the accuracy claim
2. The Brier-score-guided parameter changes (U13/U14/U19) may need Boys-equivalent analysis
3. The June 1 risk assessment assumes the ECNL migration affects teams correctly classified — but miscalibrated Boys ratings compound that risk

This task: write the gap analysis. Identify which Boys age groups and leagues have been validated (none, as far as SENTINEL can tell), what data exists to validate them, and what the minimum viable calibration check would look like.

## What You Need to Do

### Step 1: Document What Is Known About Boys Calibration

From the Ranking Engine documentation and calibration history, answer:

1. Are Boys and Girls calibrated using the same parameters? (Are K-factor, RD floor, calibration weights the same per age group, or are there gender-specific values?)
2. Have any Brier score analyses been run on Boys data? (Check `Calibration Validation 2026-04-22.md` and `calibration-held-out-validation-2026-04-23.md`)
3. Have any Boys rankings been compared to any external benchmark (USA Rank Boys data, state rankings, coach feedback)?

If the answer to all three is "no" or "unknown," state that explicitly. That is itself a significant finding.

### Step 2: Identify the Highest-Risk Boys Age Groups

Based on what you know about the Boys data in Evo Draw:

- Which Boys age groups have the most game volume? (More games = more meaningful calibration)
- Which Boys age groups are most likely to be shown at DSS? (U13B and U14B are the natural parallel to the Girls demo)
- Which Boys leagues have the highest calibration values (and therefore the highest risk of over-calibration if the value is wrong)?

Write a risk-ranked list:
| Age Group | Estimated game volume | Calibration validation status | Risk if uncalibrated |
|-----------|----------------------|------------------------------|----------------------|
| U13B | [high/medium/low] | Not validated | Demo exposure |
| U14B | ... | ... | ... |
| ... | ... | ... | ... |

Use your knowledge of the dataset and the calibration design documents to estimate game volume. If you cannot estimate, write "unknown — query needed" and provide the SQL to check.

### Step 3: Write the Minimum Viable Boys Calibration Check

You cannot run a full Brier score analysis without DB access. But you can write the three queries that would give Presten a rapid assessment of Boys calibration health:

**Query A: Top-10 U13B — do the teams make intuitive sense?**
```sql
SELECT t.name, r.rating, r.rank, r.age_group, r.gender
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'M'
  AND r.rank <= 10
ORDER BY r.rank;
```
Expected output: recognizable high-profile Boys clubs in the NE/national scene. If the top 10 are obscure teams with few games, the rankings are suspect.

**Query B: Boys vs Girls rating distribution comparison**
```sql
SELECT
  gender,
  AVG(rating) AS avg_rating,
  STDDEV(rating) AS rating_std,
  COUNT(*) AS team_count
FROM rankings
WHERE age_group = 'U13' AND rank IS NOT NULL
GROUP BY gender;
```
Expected: Boys and Girls average ratings should be similar (both centered around ~1000–1200 depending on age group). A large divergence (>100 points) suggests the calibration values differ meaningfully and one or both may be wrong.

**Query C: Boys game volume for GA tier**
```sql
SELECT
  event_tier,
  COUNT(DISTINCT game_id) AS game_count,
  COUNT(DISTINCT home_team_id) + COUNT(DISTINCT away_team_id) AS unique_teams
FROM games
WHERE age_group = 'U13' AND gender = 'M' AND event_tier IN ('ga', 'ga_aspire', 'mlsnext')
GROUP BY event_tier;
```
This surfaces whether Boys GA/ECNL volume is large enough to make calibration meaningful. If game_count is <500 for a tier, that tier's calibration for Boys is noise-dominated.

Adapt these queries to the actual schema from `Database Schema.md`.

### Step 4: Write the Boys Calibration Validation Plan

Based on Steps 1–3, write a concise plan for what ELO needs to do to validate Boys calibration:

**Option A: Minimal check (1 session, no code changes)**
- Run the three queries above
- Identify the top-10 U13B and U14B teams
- Cross-reference with any external Boys data Presten has (coach contacts, tournament pages, USA Rank Boys pages)
- If top-10 teams look wrong, flag specific teams for investigation

**Option B: Full Brier score analysis (mirroring the Girls analysis)**
- Requires: held-out game set for Boys, same age groups as Girls analysis
- Estimated effort: 2–4 hours (same methodology as Girls, just different data subset)
- When needed: before any Boys-specific calibration changes are proposed

**Recommendation:** State which option is appropriate now vs after DSS. If Boys calibration has never been validated, Option A should run before DSS as a quick sanity check. Option B should be scheduled for after DSS if Option A reveals anomalies.

### Step 5: Write the Boys GA Calibration Flag

SENTINEL has flagged "Boys GA calibration empirically unvalidated" in two consecutive briefings. This task closes that flag. Write a one-paragraph assessment:

> "Boys GA calibration is [likely correct / likely miscalibrated / unknown] because [reasoning]. The current GA boys calibration value is [X]. The Girls GA calibration value is [Y]. If both are using the same value and the Boys and Girls GA event pools have similar competition levels, the Boys value is probably correct. If Boys and Girls GA competition levels differ significantly, separate calibration may be needed."

Pull the actual calibration values from `02 - Tiger Tournaments/Projects/Rankings/Ranking Engine.md` or `League Hierarchy.md` to fill this in with real numbers.

## Deliverable

Write the gap analysis to:
`02 - Tiger Tournaments/Projects/Rankings/Boys Calibration Gap Analysis — April 2026.md`

The document must:
- Answer whether Boys and Girls use the same calibration parameters
- Provide the risk-ranked age group table
- Include the three minimum-viable validation queries
- Include the two-option validation plan with a clear recommendation
- Close the Boys GA calibration flag with a specific assessment (not "unknown")

## Definition of Done

- Boys vs Girls calibration parameter comparison is documented (same or different per age group)
- At least 3 Boys age groups are risk-ranked
- Three validation queries are written and schema-correct
- Option A vs Option B recommendation is stated
- Boys GA calibration flag is closed with a specific finding
- Summary in your briefing: what's known about Boys calibration status, recommended next action

## Why This Matters

DSS attendees are club directors. Many run both Boys and Girls programs. If a Boys U13 ranking is obviously wrong (e.g., a regional club ranked #1 nationally), it will undermine the accuracy claim for the whole system. ELO has done thorough Girls validation. This task extends that same rigor to Boys with one-tenth the effort — because most of the framework is already built.

## References

- `02 - Tiger Tournaments/Projects/Rankings/Calibration Validation 2026-04-22.md` — existing Brier work (Girls-focused)
- `02 - Tiger Tournaments/Projects/Rankings/Ranking Engine.md` — calibration parameter values per age group
- `02 - Tiger Tournaments/Projects/Rankings/League Hierarchy.md` — all leagues and calibration values (confirm whether gender-specific)
- `02 - Tiger Tournaments/Projects/Reports/calibration-held-out-validation-2026-04-23.md` — held-out validation methodology (replicable for Boys)
- ELO Briefing 2026-04-23 23:52 — GA Boys calibration flag (Flag 2, Section 2B)
