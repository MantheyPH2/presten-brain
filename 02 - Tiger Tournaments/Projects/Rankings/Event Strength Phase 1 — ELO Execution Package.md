---
title: Event Strength Phase 1 — ELO Execution Package
type: validation-package
author: ELO
date: 2026-04-27
status: ready
related: "[[Event Strength Phase 1 — Full Execution Package]]", "[[Event Strength — League Tier Coverage Audit]]", "[[Post-Fix Calibration Stability Monitoring Spec]]"
tags: [event-strength, phase-1, validation, calibration, elo, evo-draw]
---

# Event Strength Phase 1 — ELO Execution Package

> **Purpose:** ELO's structured calibration validation role in Event Strength Phase 1. FORGE executes the Phase 1 assignment (computation INSERT + event_tier remediation). ELO validates that the results are consistent with calibration assumptions. This package is ELO's checklist — run it within one session after FORGE signals Phase 1 complete.
>
> **Authorization dependency:** This package runs only after SENTINEL authorizes Phase 1 (requires G1–G4 all pass on April 29 and clean April 28 Execution Log). Do not begin until SENTINEL authorization is confirmed.

---

## Section 1: Phase 1 Scope Summary

Event Strength Phase 1 computes the `event_strength` table — a new table assigning each event an `avg_team_rating`, `strength_class` (High / Medium / Low), `strength_percentile`, `competitive_spread`, `upset_rate`, and `team_count`. This is not a change to the calibration values in the Ranking Engine — it is a new analytical layer that classifies events by the quality of teams that competed in them.

**What Phase 1 changes:**
- Creates (or repopulates) the `event_strength` table with one row per (event_id, age_group) pair
- Runs event_tier remediation on 6 unconfirmed leagues (USL Academy, EDP, Elite 64, NAL, USYS State Leagues, ODP) — assigns them structural event_tier values if null
- Assigns strength_class to all events with ≥ 4 teams, using PERCENT_RANK within (age_group, season)

**Leagues / event tiers covered:**

| Event Tier | League | Strength classification computed? |
|------------|--------|------------------------------------|
| `mlsnext_homegrown` | MLS NEXT Homegrown | Yes |
| `mlsnext_academy` | MLS NEXT Academy | Yes |
| `ecnl` | ECNL | Yes |
| `ga` | Girls Academy (non-ASPIRE) | Yes |
| `ga_aspire` | GA ASPIRE | Yes — post-April-28 fix |
| `ecnl_rl` | ECNL Regional Leagues | Yes |
| `npl` | NPL | Yes |
| `dpl` | DPL | Yes |
| `pre_ecnl` | Pre-ECNL | Yes |
| `usl_academy` | USL Academy | Yes — post-remediation |
| `edp` | EDP | Yes — post-remediation |
| `elite_64` | Elite 64 | Yes — post-remediation |
| `nal` | NAL | Yes — post-remediation |
| `state` | USYS State Cup / State Leagues | Yes — post-remediation |
| `local` | Remaining / unidentified | Yes — fallback assignment |

**Gender scope:** Both Boys and Girls. Event Strength does not have a gender split — it reflects the rating distribution of the teams (Boys or Girls) that competed at a given event. An event hosting a Boys U14 bracket and a Girls U14 bracket will produce separate rows per (event_id, age_group).

**Expected event count (2025-2026 season):** At minimum 50 events per the Phase 1 Full Execution Package V1 validation. ELO expects 100–300 events when remediation SQL brings all circuits in scope.

**Games affected:** Phase 1 does not modify any game records. It reads games and computes event-level summaries. No ratings are changed by Phase 1.

---

## Section 2: ELO Calibration Assumptions for Phase 1

For each confirmed event tier, ELO's expected strength_class skew and rating impact. "Expected direction" refers to where the tier's events should cluster in the percentile distribution — not a change in team ratings.

| League / Event Tier | Cal Value (Ranking Engine) | Expected Strength Class Skew | Expected avg_team_rating Range | Notes |
|---------------------|---------------------------|------------------------------|-------------------------------|-------|
| `mlsnext_homegrown` | 160 (Boys) | **High** (≥ 75th pct.) | 1,600–1,900+ | Top Boys pathway — should dominate High classification |
| `mlsnext_academy` | 135 (Boys) | **High** to Medium | 1,500–1,800 | Strong; slightly below Homegrown |
| `ecnl` | 120 Boys / 130 Girls | **High** | 1,500–1,750 | Top Girls pathway and Boys Tier 1 |
| `ga` | 140 Girls / 100 Boys | **High** to Medium (Girls), Medium (Boys) | Girls: 1,400–1,700; Boys: 1,200–1,500 | Girls GA cal=140 → historically strong ratings; Boys GA cal=100 → mid-tier expected |
| `ga_aspire` | 100 Girls / 100 Boys | **Medium** | 1,200–1,500 | Post-April-28 fix corrects to cal=100 — should sit below GA in strength percentile |
| `ecnl_rl` | 55 | **Medium** | 1,100–1,400 | Tier 2 — expect medium concentration |
| `npl` | 55 | **Medium** | 1,100–1,350 | Comparable to ECNL RL |
| `dpl` | 55 | **Medium** to Low | 1,000–1,300 | Regional variation; some DPL events will be Low |
| `pre_ecnl` | 25 | **Low** | 900–1,200 | Tier 3 — expect Low classification |
| `usl_academy` | 55 (structural estimate) | **Medium** | 1,100–1,400 | Estimated; not Brier-validated |
| `edp` | 55 (structural estimate) | **Medium** | 1,100–1,350 | Regional Tier 2 |
| `state` | 35 (structural estimate) | **Low** to Medium | 1,000–1,250 | Between Pre-ECNL and NPL |
| `local` | — (fallback) | **Low** | < 1,100 | Catch-all; expect Low |

**Key calibration assertion:** After Phase 1, the strength_class distribution should be monotonically consistent with league tier. That is: `mlsnext_homegrown` and `ecnl` events should have higher avg strength_percentile than `ecnl_rl`, which should be higher than `state`, which should be higher than `local`. Any inversion in this ordering is a calibration signal ELO must investigate.

**GA ASPIRE calibration note:** After the April 28 fix, GA ASPIRE events now use cal=100 (same as Boys GA and ECNL RL). Their event_strength class should sit at Medium, NOT High. If GA ASPIRE events cluster in High after Phase 1, the fix may not have been applied before Phase 1 ran — check Gate A from the Full Execution Package.

---

## Section 3: Validation Query Pack

Three queries ELO runs after FORGE signals Phase 1 complete. All three must pass before ELO files clearance.

**Query 1 — Event Strength Assignment Confirmation**

Confirm all expected event tiers have event_strength rows, and no Phase 1 tier shows NULL strength_class.

```sql
SELECT
  e.event_tier,
  COUNT(es.id) AS event_strength_rows,
  COUNT(*) FILTER (WHERE es.strength_class IS NULL) AS null_strength_rows,
  ROUND(AVG(es.avg_team_rating), 1) AS avg_team_rating,
  ROUND(AVG(es.strength_percentile), 1) AS avg_percentile
FROM events e
LEFT JOIN event_strength es ON es.event_id = e.id
WHERE e.season >= '2025-2026'
GROUP BY e.event_tier
ORDER BY avg_team_rating DESC NULLS LAST;
```

**Expected:** Every core tier (ecnl, ga, ga_aspire, mlsnext_*, ecnl_rl, npl, dpl, pre_ecnl) shows event_strength_rows > 0 and null_strength_rows = 0. Tiers showing 0 rows mean either those events don't exist in the 2025-2026 season (acceptable) or the computation skipped them (investigate). Any row with null_strength_rows > 0 is a failure — those events have incomplete data.

**Pass criterion:** null_strength_rows = 0 for all tiers. All Tier 1 tiers (mlsnext, ecnl, ga) show event_strength_rows > 0.

---

**Query 2 — Rating Shift After Phase 1 Recompute**

Event Strength Phase 1 does NOT modify team ratings — it only reads rankings to compute event metrics. After Phase 1, team ratings should be unchanged from the April 29 G4 snapshot.

This query checks for unexpected rating drift (any session-wide drift between G4 snapshot values and current values would indicate a recompute ran inadvertently during the Phase 1 session).

```sql
-- Compare current Girls GA ASPIRE avg ratings against April 29 baseline
-- ELO fills baseline_avg from April 29 Gate Results Structured Log (G4 column)
WITH current_state AS (
  SELECT
    r.age_group,
    LEFT(r.age_group, 1) AS gender_flag,
    e.event_tier,
    ROUND(AVG(r.rating), 1) AS current_avg_rating,
    COUNT(DISTINCT r.team_id) AS team_count
  FROM rankings r
  JOIN team_events te ON te.team_id = r.team_id
  JOIN events e ON e.id = te.event_id
  WHERE e.event_tier IN ('ga_aspire', 'ga', 'ecnl')
    AND e.season >= '2025-2026'
  GROUP BY r.age_group, LEFT(r.age_group, 1), e.event_tier
)
SELECT * FROM current_state
ORDER BY age_group, event_tier;
```

**Expected:** avg_team_rating values match (within ±2 pts floating-point variance) the values recorded in the April 29 Gate Results log. A shift > 5 pts for any age group / event_tier combination indicates an unintended recompute — flag to SENTINEL immediately before proceeding.

**Pass criterion:** All age group / event_tier avg ratings within ±5 pts of April 29 G4 baseline. No drift > 5 pts.

---

**Query 3 — Distribution Integrity Check**

Confirms the Phase 1 strength_class distribution is internally consistent and matches Section 2 calibration expectations.

```sql
-- Per-tier strength class distribution
SELECT
  e.event_tier,
  es.strength_class,
  COUNT(*) AS event_count,
  ROUND(AVG(es.avg_team_rating), 1) AS avg_rating,
  ROUND(AVG(es.strength_percentile), 1) AS avg_percentile
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE e.season >= '2025-2026'
GROUP BY e.event_tier, es.strength_class
ORDER BY e.event_tier, es.strength_class;
```

**Expected by tier (from Section 2):**
- `ecnl`, `mlsnext_*`: overwhelming majority High (> 60% of their events)
- `ga`: majority High (Girls), majority Medium (Boys) — if the query is not gender-split, expect a mixed distribution skewing High
- `ga_aspire`: majority Medium — NOT High. If ga_aspire skews High, Gate A from the Full Execution Package may have failed (ASPIRE fix didn't run before Phase 1)
- `ecnl_rl`, `npl`, `dpl`: majority Medium
- `pre_ecnl`, `state`, `local`: majority Low

**Alert thresholds:**
- If `ga_aspire` events show > 40% High: STOP. Investigate whether April 28 GA ASPIRE fix was applied before Phase 1.
- If any tier that Section 2 expects as "Medium" skews > 70% High: investigate percentile partitioning — may indicate the partition window is too narrow.
- If overall distribution is extreme (> 60% High or < 15% High across all events): check that PERCENT_RANK is partitioning by (age_group, season) and not globally.

**Pass criterion:** ga_aspire events are < 40% High. Overall distribution is within 15%–40% High, 30%–50% Medium, 20%–40% Low. No tier inversion in avg_percentile vs. Section 2 ordering.

---

## Section 4: Pass/Fail Criteria

| Check | Pass Criterion | Fail Criterion | Action if Fail |
|-------|---------------|----------------|----------------|
| Q1: All Phase 1 tiers assigned | All core tiers show event_strength rows > 0; null_strength_rows = 0 for all | Any null_strength_rows > 0 | Flag to FORGE: computation produced incomplete rows — truncate and re-run Section 3 INSERT |
| Q2: No unintended rating drift | All age group avg ratings within ±5 pts of April 29 G4 baseline | Any avg shift > 5 pts vs. baseline | STOP. Investigate source of drift before proceeding — Phase 1 should not modify ratings |
| Q3: Distribution integrity | ga_aspire < 40% High; overall distribution within expected bands; no tier inversion | ga_aspire ≥ 40% High OR overall > 60% High OR tier ordering inverted | If ga_aspire ≥ 40% High: check Gate A (ASPIRE fix prerequisite). If other inversion: check PERCENT_RANK partition |

**All 3 PASS:** Phase 1 validation complete. ELO files clearance in next briefing (use Section 5 format exactly).

**Any FAIL:** ELO files failure in briefing. SENTINEL holds Phase 2 authorization until ELO clears the flag and re-runs the failing query.

---

## Section 5: ELO Clearance Filing

When all 3 queries pass, ELO files this exact block in the next briefing:

```
Event Strength Phase 1 — ELO Validation Clearance
Date: [date of validation session]
Q1: PASS — [N] event tiers assigned with event_strength. No null strength_class rows.
Q2: PASS — No unexpected rating drift. Largest delta vs. April 29 baseline: [N] pts in [age_group / event_tier]. Within ±5 pt threshold.
Q3: PASS — GA ASPIRE: [X]% High (threshold: < 40%). Overall distribution: [X]% High / [X]% Medium / [X]% Low. Tier ordering consistent with Section 2 calibration expectations.

Event Strength Phase 1 validated. ELO recommends SENTINEL authorize Event Strength Phase 2 scoping session.
```

SENTINEL will look for this exact format. Do not summarize differently.

---

## Section 6: Dependency Chain

```
April 28: GA ASPIRE UPDATE + ranking recompute
  ↓
April 29: G1–G4 all pass
  ↓
SENTINEL authorizes Event Strength Phase 1
  ↓
FORGE executes Phase 1 (Full Execution Package: Gates → Remediation → Schema → INSERT → Validation)
  ↓
ELO runs this package (3 queries: Assignment Confirmation, Rating Drift, Distribution Integrity)
  ↓
ELO files Section 5 clearance in briefing
  ↓
SENTINEL authorizes Event Strength Phase 2 scoping
```

**Hard blocks:**
- If April 29 G2 fails (rating distribution unexpected after fix), Phase 1 authorization is held regardless of this package.
- If FORGE's Phase 1 Gate A fails (ASPIRE events still tagged `ga`), Phase 1 cannot proceed — the Phase 1 recompute will assign incorrect strength ratings to ASPIRE events, requiring a full Phase 1 re-run.
- ELO Q3 failure on ga_aspire clustering High should be treated as a potential Gate A failure downstream — investigate before clearing.

---

## References

- `Rankings/Event Strength Phase 1 — Full Execution Package.md` — FORGE-side execution; scope this package validates
- `Rankings/Event Strength — League Tier Coverage Audit.md` — event tier / cal value source of truth for Section 2
- `Rankings/April 29 Gate Results — Structured Log.md` — G4 baseline values for Q2 comparison
- `Rankings/Post-Fix Calibration Stability Monitoring Spec.md` — ongoing monitoring this package feeds into
- `Product & Planning/DSS Feature Readiness Tracker — May 2026.md` — Event Strength Phase 1 readiness cell

*ELO — 2026-04-27*
