---
title: DSS Demo Validation Criteria — May 2026
aliases: [DSS Validation Criteria, May 14 Validation]
tags: [rankings, dss, validation, demo, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: ELO Agent
status: ready
task: task-2026-04-24-dss-demo-validation-criteria
run_on: 2026-05-14
---

# DSS Demo Validation Criteria — May 2026

> Explicit pass/fail criteria for the May 14 data validation run. Run all queries in sequence. The go/no-go decision for the DSS demo is determined by these criteria — not by judgment. Total time: ~45 minutes in psql.
>
> **DSS date: May 16, 2026. Final validation: May 14.**

---

## Section 1: Anchor Team Requirements (MUST-PASS)

Thresholds derived from: USA Rank reference positions (Section 2 of `USA Rank Comparison 2026-04-23.md`), Evo Draw band thresholds from `Rank Bands Threshold Validation — 2026-04-24.md`, and the ±3x rank alignment standard documented in the USA Rank Comparison.

| Team | Age Group | Must Find | Max Acceptable Rank | Min Acceptable Rating | Band Requirement |
|------|-----------|-----------|--------------------|-----------------------|-----------------|
| Football Academy NJ (any variation) | U13G | **REQUIRED** | ≤ 25 | ≥ 1500 (Gold) | Gold |
| Wall Soccer Club Wall Elite Porto | U13G | **REQUIRED** | ≤ 50 | ≥ 1450 | Gold or Silver |
| FC Stars G2012 ECNL White | U13G | Preferred | ≤ 300 | ≥ 1350 (Silver) | Silver or better |
| NE Surf G2012 State Navy | U13G | Preferred | ≤ 600 | ≥ 900 | NOT Gold |

**Derivation rationale:**
- Football Academy NJ: USA Rank #4. Using ±3x alignment standard → Evo Draw rank ≤12 is excellent, ≤25 is acceptable. Gold threshold ≥1500 is required — a #4 national ECNL club must be in the top band.
- Wall Soccer Club: USA Rank #12. ±3x alignment → rank ≤36 is good, ≤50 acceptable. ECNL club competing at national level → Gold or high Silver expected.
- FC Stars G2012 ECNL White: USA Rank #121. ±3x → rank ≤363 acceptable. ECNL club → Silver band expected.
- NE Surf G2012 State Navy: USA Rank #198, state-level team. Expected Red or Bronze band (900–1349). If this team appears as Gold, something is wrong with the calibration.

**Lookup queries:**

```sql
-- Football Academy NJ — search all name variants
SELECT t.name, r.rating::int, r.rank
FROM teams t
JOIN rankings r ON r.team_id = t.id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND (t.name ILIKE '%football academy%'
    OR t.name ILIKE '%FA NJ%'
    OR t.name ILIKE '%Football Academy NJ%'
    OR t.name ILIKE '%FA 12G%')
ORDER BY r.rank;
```

```sql
-- Wall Soccer Club
SELECT t.name, r.rating::int, r.rank
FROM teams t
JOIN rankings r ON r.team_id = t.id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND (t.name ILIKE '%wall soccer%'
    OR t.name ILIKE '%wall elite%')
ORDER BY r.rank;
```

```sql
-- FC Stars and NE Surf
SELECT t.name, r.rating::int, r.rank,
  CASE
    WHEN r.rating >= 1500 THEN 'Gold'
    WHEN r.rating >= 1350 THEN 'Silver'
    WHEN r.rating >= 1200 THEN 'Bronze'
    WHEN r.rating >= 1050 THEN 'Red'
    WHEN r.rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS band
FROM teams t
JOIN rankings r ON r.team_id = t.id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND (t.name ILIKE '%FC Stars%G2012%'
    OR t.name ILIKE '%FC Stars%2012%'
    OR t.name ILIKE '%NE Surf%G2012%'
    OR t.name ILIKE '%NE Surf%2012%')
ORDER BY r.rank;
```

**Also check team_merges for name variants:**

```sql
-- Football Academy NJ may be merged under a different canonical name
SELECT t1.name AS canonical_name, t2.name AS merged_name, r.rating::int, r.rank
FROM team_merges tm
JOIN teams t1 ON tm.canonical_team_id = t1.id
JOIN teams t2 ON tm.merged_team_id = t2.id
JOIN rankings r ON r.team_id = tm.canonical_team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND (t2.name ILIKE '%football academy%'
    OR t2.name ILIKE '%FA NJ%'
    OR t2.name ILIKE '%FA 12G%'
    OR t2.name ILIKE '%wall soccer%')
ORDER BY r.rank;
```

**FAILURE CRITERION (demo-blocking):**
- Football Academy NJ is not found in any name variation in U13G rankings → **demo NOT safe for rankings feature. SENTINEL escalation immediately.**
- Football Academy NJ found but rank > 50 → flag to SENTINEL. Demo weakened but may proceed with caveat.
- Football Academy NJ found but not Gold (rating < 1500) → **Rank Bands feature is miscalibrated. Demo NOT safe for band display.**

---

## Section 2: Rank Band Assignment Requirements

Thresholds from `Rank Bands Threshold Validation — 2026-04-24.md`:
- Gold: ≥ 1500
- Silver: 1350–1499
- Bronze: 1200–1349
- Red: 1050–1199
- Blue: 900–1049
- Green: < 900

| Team | Expected Band | Failure Assignment | Reasoning |
|------|--------------|-------------------|-----------|
| Football Academy NJ | Gold | Silver, Bronze, or lower | Demo anchor — a top-5 national ECNL team must be Gold |
| Wall Soccer Club | Gold or Silver | Bronze or lower | Top-12 national team; should be in the top 6% of ranked teams |
| FC Stars G2012 ECNL White | Silver | Gold (over-calibrated) or Bronze/lower (under) | ECNL club ranked #121 nationally — mid-Silver expected |
| NE Surf G2012 State Navy | Bronze, Red, or Blue | Gold | State-level sub-team; Gold would indicate calibration failure |

**Band verification query with actual thresholds:**

```sql
SELECT t.name, r.rating::int, r.rank,
  CASE
    WHEN r.rating >= 1500 THEN 'Gold'
    WHEN r.rating >= 1350 THEN 'Silver'
    WHEN r.rating >= 1200 THEN 'Bronze'
    WHEN r.rating >= 1050 THEN 'Red'
    WHEN r.rating >= 900  THEN 'Blue'
    ELSE 'Green'
  END AS band
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND (t.name ILIKE '%football academy%'
    OR t.name ILIKE '%wall soccer%'
    OR t.name ILIKE '%wall elite%'
    OR t.name ILIKE '%FC Stars%2012%'
    OR t.name ILIKE '%NE Surf%2012%')
ORDER BY r.rank;
```

**FAILURE CRITERION (demo-blocking for Rank Bands feature):**
- Football Academy NJ is not Gold (rating < 1500) → Rank Bands miscalibrated. Do not demo the band feature.
- Wall Soccer Club is Bronze or lower (rating < 1200) → Investigate merge integrity and game coverage before proceeding.

---

## Section 3: Data Coverage Requirements

### 3.1 Total ranked teams in U13G

```sql
SELECT COUNT(*) AS total_u13g_ranked
FROM rankings
WHERE age_group = 'U13' AND gender = 'F' AND rank IS NOT NULL;
```

**Pass threshold: ≥ 800 teams**

**Reasoning:** USA Rank shows rankings for roughly 3,000+ U13G teams nationally. Evo Draw covers 7 data sources vs USA Rank's 400+. A credible "national rankings" claim requires covering at least the ECNL, GA, MLS NEXT, and NPL circuits — these circuits together represent approximately 800–1,500 nationally competitive U13G teams. Below 500 is indefensible at DSS; 800 is the floor for a credible coverage claim when speaking to ECNL club directors.

**Failure threshold: < 500 teams** — if below 500, the coverage claim cannot be made at DSS without significant qualification.

### 3.2 Teams with ≥ 5 games (reliable ratings)

```sql
SELECT COUNT(DISTINCT r.team_id) AS teams_with_5plus_games
FROM rankings r
WHERE r.age_group = 'U13' AND r.gender = 'F' AND r.rank IS NOT NULL
  AND (
    SELECT COUNT(*)
    FROM games g
    WHERE (g.home_team_id = r.team_id OR g.away_team_id = r.team_id)
      AND g.is_hidden = false
  ) >= 5;
```

**Pass threshold: ≥ 60% of total ranked U13G teams have ≥ 5 games**

**Reasoning:** Glicko-2 ratings with fewer than 5 games carry high uncertainty (RD near the floor value). If fewer than 60% of ranked teams have reliable ratings, the rankings list contains too much noise for a credible accuracy claim. 60% is the minimum for a defensible demo; 75%+ is preferred.

### 3.3 Total U13G games in the last 12 months

```sql
SELECT COUNT(*) AS u13g_games_last_12mo
FROM games
WHERE age_group = 'U13' AND gender = 'F'
  AND match_date >= NOW() - INTERVAL '12 months'
  AND is_hidden = false;
```

**Pass threshold: ≥ 10,000 games**

**Reasoning:** A national ranking system for U13G claiming comprehensive coverage should see at minimum 10,000 games per year to represent the major circuits (ECNL alone runs ~2,000+ U13G games per season; GA and MLS NEXT add comparable volume). Below 5,000 is a red flag; below 10,000 weakens the coverage claim but is not demo-blocking.

**Failure threshold: < 5,000 games** — if below 5,000, do not claim comprehensive national coverage at DSS.

---

## Section 4: Event Strength Requirements (Conditional)

### If Event Strength IS implemented before May 14

```sql
-- Confirm event_strength table is populated and ECNL nationals are High
SELECT e.event_name, es.strength_class, es.avg_team_rating::int,
  es.team_count
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE e.event_name ILIKE '%ECNL%'
  AND es.age_group = 'U13' AND es.gender = 'F'
ORDER BY es.avg_team_rating DESC
LIMIT 10;
```

**Pass criteria:**
1. At least one ECNL event has `strength_class = 'High'`
2. No ECNL National event has `strength_class = 'Low'`
3. The demo event selected in `Event Strength Post-Compute Validation — 2026-04-24.md` appears with `strength_class = 'High'`

**Failure criterion:** Any ECNL National event labeled 'Low' → Event Strength is miscalibrated. Do not demo the feature.

### If Event Strength is NOT implemented before May 14

Note as scope reduction: "Event Strength labeling is deferred. Demo will show rankings and Rank Bands only."

**Expected DSS impact:** Moderate reduction in demo depth. Rankings and Rank Bands are the core value proposition. Event Strength adds credibility to the algorithm explanation but is not demo-blocking to skip it.

---

## Section 5: May 14 Go/No-Go Decision Tree

```
MUST-PASS (any failure = demo not ready for that feature):
□ Football Academy NJ found in U13G rankings, rank ≤ 50
□ Football Academy NJ rating ≥ 1500 (Gold band) [REQUIRED for Rank Bands demo]
□ Total U13G ranked teams ≥ 500 (floor for coverage claim)

SHOULD-PASS (failure = flag to SENTINEL, but demo can proceed with caveats):
□ Football Academy NJ rank ≤ 25 (ideal; rank 25–50 is acceptable with talking point)
□ Wall Soccer Club found in U13G, rank ≤ 50
□ Wall Soccer Club rated Gold or Silver (≥ 1350)
□ FC Stars G2012 ECNL White found, band = Silver or better
□ NE Surf G2012 State Navy found, band ≠ Gold
□ Teams with ≥ 5 games count ≥ 60% of ranked teams
□ ECNL National events labeled High (if Event Strength implemented)
□ Total U13G ranked teams ≥ 800 (preferred threshold)

ACCEPTABLE GAPS (do not block demo):
□ Boys calibration not formally validated (Boys not in primary demo)
□ SnapSoccer ingestion not running (documented source gap)
□ Club Rankings not implemented
□ USA Rank comparison table columns still showing [QUERY] placeholders
□ NE Surf or FC Stars not found in Evo Draw (source coverage gap — expected)
□ Event Strength not implemented (scope reduction, not failure)

ESCALATION PROTOCOL:
- Any MUST-PASS failure → SENTINEL immediately, do not proceed with demo prep
- Multiple SHOULD-PASS failures (≥ 3) → SENTINEL, consider scoping demo to rankings only (no Rank Bands)
- Single SHOULD-PASS failure → note it, prepare fallback talking point from Section 6
```

---

## Section 6: Fallback Talking Points

For SHOULD-PASS failures that do not block the demo:

**Football Academy NJ found but rank 26–50:**
> "Rankings reflect the full history of game results — Football Academy's current position reflects their 2025-26 ECNL season. They've played against some of the strongest competition in the Northeast, and their rating of [X] puts them solidly in the Gold tier. Absolute rank positions shift as more game data is processed each week."

**Wall Soccer Club not in top 50:**
> "Wall's ranking reflects performance across their full competitive schedule this season. Their [Gold/Silver] band designation means they're ranked in the top [X]% of all U13 Girls teams nationally — which is an accurate reflection of their ECNL circuit standing."

**Total U13G ranked teams between 500–799:**
> "We're actively adding game sources — our current 7 pipeline integrations capture all ECNL, GA, MLS NEXT, and NPL circuit games. We have [X] U13 Girls teams ranked today and that number grows each week as we add sources. The teams that matter most to the clubs in this room are already in the system."

**Event Strength not implemented:**
> "Event Strength labeling is coming this summer — it's the next layer on top of what you're seeing. Today I'll walk you through the ranking engine itself and how a team's Glicko rating reflects the quality of competition they've faced."

**NE Surf appears as Gold (calibration anomaly):**
> [Do not demo Rank Bands — this is a SHOULD-PASS failure, escalate to SENTINEL, prepare to demo rankings without the band feature. Do not explain band assignments live if they're wrong.]

---

## References

- [[Rank Bands Threshold Validation — 2026-04-24]] — Gold/Silver/Bronze threshold values used in Section 2
- [[USA Rank Comparison 2026-04-23]] — reference positions for anchor teams (Section 1)
- [[Event Strength Post-Compute Validation — 2026-04-24]] — demo event selection (Section 4)
- `02 - Tiger Tournaments/Projects/Product & Planning/DSS Demo Spec — May 2026.md` — demo structure and anchor team list
- [[Calibration Validation 2026-04-22]] — for coverage count estimates in Section 3
