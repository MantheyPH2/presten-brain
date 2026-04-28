---
type: elo-deliverable
agent: ELO
date: 2026-04-27
status: ready
related_task: task-2026-04-27-tier2-calibration-post-implementation-monitoring-plan
execution_window: post-implementation (expected May 17–30 sprint)
---

# Tier 2 Calibration — Post-Implementation Monitoring Plan

**Filed by:** ELO — 2026-04-27  
**Use:** Run immediately after Tier 2 calibration SQL executes (expected May 17–30 sprint window).  
**Companion document:** `Rankings/Tier 2 Undefined Leagues — Calibration Implementation SQL.md`

---

## Section 1: What Was Implemented (Reference Summary)

Four Tier 2 leagues currently have no `event_tier` calibration value in the League Hierarchy. The implementation assigns:

| League | Event Tier String | Old Cal Value | New Cal Value |
|--------|------------------|--------------|--------------|
| USL Academy | `usl_academy` | NULL | 55 |
| EDP Soccer | `edp` | NULL | 55 |
| Elite 64 | `elite_64` | NULL | 55 |
| NAL (National Academy League) | `nal` | NULL | 40 |

**Note:** NAL is conditional — only apply if April 29 volume check shows ≥ 500 NAL games. If NAL was deferred, adjust this document to cover only the 3 leagues that were implemented.

**Expected direction:** Teams in USL Academy, EDP, and Elite 64 (cal=55) should see ratings shift toward the mid-Tier-2 range — above fully uncalibrated teams (who have been effectively treated as cal=0 or default), but below Tier-1 ECNL/MLS NEXT. NAL teams (cal=40) should shift slightly lower than the other three. Magnitude estimate: teams with ≥10 games in these leagues should see average rating changes of 5–25 points; teams with ≥30 games may see up to 40-point shifts.

---

## Section 2: Immediate Post-Implementation Queries

Run these queries within 24 hours of the SQL executing and before the next pipeline run completes. Estimated runtime: 15 minutes.

---

### Query 1 — Affected League Count

**Purpose:** Confirm the UPDATE applied to the correct leagues and no others.

```sql
-- Confirm calibration values updated for the 4 Tier 2 leagues
SELECT
    event_tier,
    calibration_value,
    updated_at
FROM league_hierarchy
WHERE event_tier IN ('usl_academy', 'edp', 'elite_64', 'nal')
ORDER BY event_tier;
```

**Expected output:**
| event_tier | calibration_value | updated_at |
|------------|------------------|------------|
| edp | 55 | [implementation date] |
| elite_64 | 55 | [implementation date] |
| nal | 40 | [implementation date, or absent if deferred] |
| usl_academy | 55 | [implementation date] |

**Green:** All implemented leagues show the correct cal value with today's `updated_at`.  
**Red:** Any row shows NULL, 0, or a value other than 55/40. UPDATE did not apply correctly — do NOT proceed to recompute. Diagnose before continuing.

---

### Query 2 — Affected Team Count

**Purpose:** Understand the scope of the rating impact. How many teams have games in these leagues?

```sql
-- Teams with games in the recalibrated leagues (current season + prior season)
SELECT
    event_tier,
    gender,
    age_group,
    COUNT(DISTINCT team_id) AS team_count,
    COUNT(*) AS game_count
FROM games
WHERE event_tier IN ('usl_academy', 'edp', 'elite_64', 'nal')
  AND game_date >= '2024-01-01'
GROUP BY event_tier, gender, age_group
ORDER BY event_tier, gender, age_group;
```

**Expected output:** Non-zero team counts for Boys and Girls across multiple age groups. USL Academy and NAL are expected Boys-heavy; EDP and Elite 64 are mixed.  
**Flag:** If a league shows 0 teams after the implementation, the `event_tier` string used in the UPDATE may not match the string in the `games` table. Escalate to SENTINEL before proceeding.

---

### Query 3 — Rating Shift Distribution (Post-Recompute)

**Purpose:** Confirm the calibration change produced expected rating movements. Run after the first compute-rankings.js execution post-implementation.

```sql
-- Rating shift for teams with ≥5 games in Tier 2 leagues
-- Requires a pre-implementation snapshot table; adjust [snapshot_table] to actual name
SELECT
    g.event_tier,
    t.gender,
    t.age_group,
    COUNT(DISTINCT t.team_id) AS teams_moved,
    ROUND(AVG(tr.rating - snap.pre_rating), 1) AS avg_delta,
    ROUND(MIN(tr.rating - snap.pre_rating), 1) AS min_delta,
    ROUND(MAX(tr.rating - snap.pre_rating), 1) AS max_delta,
    SUM(CASE WHEN ABS(tr.rating - snap.pre_rating) > 40 THEN 1 ELSE 0 END) AS outlier_count
FROM team_ratings tr
JOIN [snapshot_table] snap ON tr.team_id = snap.team_id
JOIN teams t ON tr.team_id = t.team_id
JOIN (
    SELECT DISTINCT team_id, event_tier
    FROM games
    WHERE event_tier IN ('usl_academy', 'edp', 'elite_64', 'nal')
      AND game_date >= '2024-01-01'
    GROUP BY team_id, event_tier
    HAVING COUNT(*) >= 5
) g ON tr.team_id = g.team_id
GROUP BY g.event_tier, t.gender, t.age_group
ORDER BY g.event_tier, t.gender, t.age_group;
```

**Expected result:** avg_delta between +5 and +25 pts for most groups. Teams that were previously unrated (rating = 0 or default) may show larger shifts.  
**Yellow:** avg_delta outside +2 to +30 pts — investigate before filing clearance.  
**Red:** avg_delta > 50 pts for any group, or outlier_count > 10 in any group. Escalate to SENTINEL.

---

### Query 4 — Tier 2 vs Adjacent Tier Boundary Check

**Purpose:** Confirm that Tier 2 teams now fall on the correct side of the Tier 1 / Tier 2 boundary and Tier 2 / Tier 3 boundary.

```sql
-- Boundary check: sample Tier 2 teams vs Tier 1 (ECNL/MLS NEXT) and Tier 3 (state-level) teams
-- Compare average ratings across tiers for same age group (e.g., U15B)
SELECT
    CASE
        WHEN event_tier IN ('ecnl_boys', 'ecnl', 'mlsnext') THEN 'Tier 1'
        WHEN event_tier IN ('usl_academy', 'edp', 'elite_64', 'nal', 'npl', 'dpl') THEN 'Tier 2'
        ELSE 'Tier 3 / Other'
    END AS tier_group,
    gender,
    age_group,
    COUNT(DISTINCT tr.team_id) AS team_count,
    ROUND(AVG(tr.rating), 1) AS avg_rating
FROM team_ratings tr
JOIN games g ON tr.team_id = g.home_team_id OR tr.team_id = g.away_team_id
WHERE g.age_group = 'U15'  -- adjust as needed
  AND g.game_date >= '2024-01-01'
GROUP BY tier_group, gender, age_group
ORDER BY avg_rating DESC;
```

**Expected result:** Tier 1 avg_rating > Tier 2 avg_rating > Tier 3 avg_rating in each age group. The newly calibrated leagues (USL Academy, EDP, Elite 64) should land within the Tier 2 band, not above Tier 1.  
**Flag:** If any Tier 2 group avg_rating exceeds Tier 1 avg_rating, the calibration value may be too high. Escalate.

---

### Query 5 — No Unintended Scope

**Purpose:** Confirm that teams with zero games in the recalibrated leagues have ratings unchanged (delta = 0 after implementation).

```sql
-- Teams that should NOT have changed: zero games in Tier 2 leagues
-- Requires snapshot table [snapshot_table]
SELECT
    COUNT(*) AS teams_with_unexpected_change,
    ROUND(AVG(ABS(tr.rating - snap.pre_rating)), 1) AS avg_unexpected_delta
FROM team_ratings tr
JOIN [snapshot_table] snap ON tr.team_id = snap.team_id
WHERE tr.team_id NOT IN (
    SELECT DISTINCT team_id
    FROM games
    WHERE event_tier IN ('usl_academy', 'edp', 'elite_64', 'nal')
      AND game_date >= '2024-01-01'
)
AND ABS(tr.rating - snap.pre_rating) > 2;  -- allow ±2 pts for floating-point noise
```

**Expected result:** 0 teams with unexpected change. Any non-zero result means the recompute affected teams outside the implementation scope — this is a **Red** finding requiring investigation before filing clearance.

---

### Query 6 — Top-100 Stability Check

**Purpose:** How many top-100 teams (in age groups most affected by Tier 2 calibration) changed rank by ≥5 positions?

```sql
-- Rank shift check for U15B and U16B (expected most affected by USL Academy / EDP)
-- Requires snapshot table with pre-implementation ranks [snapshot_table]
SELECT
    t.age_group,
    t.gender,
    COUNT(*) AS teams_rank_shifted_5_plus,
    MAX(ABS(tr_rank.rank - snap.pre_rank)) AS max_rank_shift
FROM team_ratings tr
JOIN [snapshot_table] snap ON tr.team_id = snap.team_id
JOIN (
    SELECT team_id, RANK() OVER (PARTITION BY age_group, gender ORDER BY rating DESC) AS rank
    FROM team_ratings
) tr_rank ON tr.team_id = tr_rank.team_id
JOIN teams t ON tr.team_id = t.team_id
WHERE snap.pre_rank <= 100
  AND ABS(tr_rank.rank - snap.pre_rank) >= 5
GROUP BY t.age_group, t.gender
ORDER BY t.age_group, t.gender;
```

**Expected result:** Some reshuffling is expected — teams that were previously unrated or mis-rated will move. Moderate reshuffling (10–30 teams per age/gender group shifting ≥5 positions) is acceptable.  
**Yellow:** 30–50 teams per group shifting ≥5 positions — review which leagues are driving it.  
**Red:** > 50 teams per group shifting ≥5 positions, or any team shifting > 20 positions — escalate to SENTINEL.

---

## Section 3: Thresholds and Flags Summary

| Query | Green | Yellow | Red |
|-------|-------|--------|-----|
| Q1 — League count | All 3–4 leagues show correct cal value | N/A | Any league shows wrong or missing value |
| Q2 — Team count | Non-zero for expected leagues | One league shows 0 teams | Multiple leagues show 0 teams |
| Q3 — Rating shift | avg_delta 2–30 pts, outlier_count ≤ 10 | avg_delta 30–50 pts | avg_delta > 50 pts OR outlier_count > 10 OR delta is negative (ratings went down) |
| Q4 — Tier boundary | Tier 1 > Tier 2 > Tier 3 in avg_rating | Tier 2 within 5 pts of Tier 1 | Tier 2 avg exceeds Tier 1 avg |
| Q5 — No unintended scope | 0 teams with unexpected change | 1–5 teams (investigate) | > 5 teams — scope error in SQL |
| Q6 — Top-100 stability | < 30 teams per group shift ≥5 places | 30–50 teams | > 50 teams OR any team shifts > 20 places |

---

## Section 4: Rollback Criteria

ELO recommends rollback to SENTINEL (SENTINEL decides; ELO does not self-execute rollback) if ANY of the following are true:

1. **Query 5 shows > 0 teams with unexpected rating changes** — scope error in the UPDATE SQL
2. **Query 3 shows avg_delta > 50 pts** for any age/gender group — calibration values too large
3. **Query 3 shows avg_delta is negative** (teams rated DOWN after implementing non-zero cal) — implementation error
4. **Query 4 shows any Tier 2 group averaging HIGHER than Tier 1** — calibration values inconsistent with league hierarchy
5. **Query 6 shows > 50 teams per group shifting ≥ 5 rank positions** — systemic reshuffling at scale

If rollback is recommended:
- File SENTINEL flag in next briefing: "Tier 2 calibration implementation flag — rollback recommended: [specific query failure]"
- Reference rollback SQL in `Rankings/Tier 2 Undefined Leagues — Calibration Implementation SQL.md` Section 7
- Do not re-run implementation until SENTINEL and Presten review the failure

---

## Section 5: One-Week Follow-Up Check

After the first pipeline run following implementation, confirm the Tier 2 calibration values are being applied to newly ingested games (not just retroactively):

```sql
-- Games ingested after implementation date in Tier 2 leagues
-- Replace [implementation_date] with actual date
SELECT
    event_tier,
    COUNT(*) AS new_games,
    MIN(game_date) AS earliest_new_game,
    MAX(game_date) AS latest_new_game
FROM games
WHERE event_tier IN ('usl_academy', 'edp', 'elite_64', 'nal')
  AND ingestion_date >= '[implementation_date]'
GROUP BY event_tier;
```

**Expected result:** Non-zero new games for the Tier 2 leagues that are actively in-season. If a league shows 0 new games after one pipeline week, either (a) the league's season is over, or (b) the `event_tier` string in new games doesn't match the League Hierarchy entry. Investigate option (b) if the league should still be active.

File the one-week follow-up result in the next ELO briefing after it runs. One sentence: "Tier 2 leagues one-week pipeline check: [N] new games ingested across [N] leagues. Calibration applying correctly / [flag if not]."

---

## Filing Protocol

When monitoring queries are run:
1. Record Q1–Q6 results in `Rankings/Tier 2 Calibration — Implementation Results — [date].md`
2. State overall verdict: **IMPLEMENTATION CONFIRMED** / **YELLOW (investigate)** / **ROLLBACK RECOMMENDED**
3. File in ELO briefing: "Tier 2 calibration post-implementation monitoring complete. Verdict: [result]. Avg rating shift: [X] pts. Teams affected: [N]. Any flags: [none / describe]."
4. If IMPLEMENTATION CONFIRMED: update `Rankings/Tier 2 Undefined Leagues — Calibration Recommendation.md` status to `implemented: [date]`
