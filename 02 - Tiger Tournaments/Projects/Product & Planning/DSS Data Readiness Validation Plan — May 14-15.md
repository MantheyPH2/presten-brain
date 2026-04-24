---
title: DSS Data Readiness Validation Plan — May 14-15
aliases: [DSS Validation Plan, May 14 Checklist]
tags: [dss, demo, validation, rankings, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: ELO Agent
status: final — ready to execute May 14
demo_date: 2026-05-16
validation_date: 2026-05-14
---

# DSS Data Readiness Validation Plan — May 14–15

> **Instructions:** Open this file at 9 AM on May 14. Run every query in psql against `youth_soccer` at localhost:5432. Run the queries top to bottom in sequence. At each branch point, follow the decision logic. All decisions should be made by 10:30 AM. DSS is May 16.
>
> This document is a companion to [[DSS Demo Spec — May 2026]]. The demo spec says what to show. This plan confirms it can be shown.

---

## Pre-Validation Setup

```bash
# Connect to the database
psql -U p -d youth_soccer

# Check the connection is live
SELECT NOW();
```

---

## Step 1: Rank Bands Validation

These queries confirm `rankings_with_bands` view exists and the primary demo team has the correct band.

---

### 1A: View exists and is populated

```sql
SELECT COUNT(*) AS u13g_band_count
FROM rankings_with_bands
WHERE age_group = 'U13' AND gender = 'F';
```

**Expected:** > 200

**If 0:** Rank Bands was NOT implemented. Switch to fallback demo immediately (see Section 5, Branch 1). Do not run further Rank Bands queries.

---

### 1B: Football Academy NJ — single record, correct band

```sql
SELECT
  t.team_name,
  rb.rating,
  rb.rank,
  rb.rank_band,
  rb.band_color
FROM rankings_with_bands rb
JOIN teams t ON t.team_id = rb.team_id
WHERE t.team_name ILIKE '%Football Academy%'
  AND rb.age_group = 'U13'
  AND rb.gender = 'F'
ORDER BY rb.rank;
```

**Expected:** Exactly ONE record, `rank_band` = 'Gold' or 'Silver', `rank` ≤ 50.

**If two records returned:** The Football Academy NJ / FA 12G Black merge from `Pre-DSS Team Merge Audit — U13G.md` was NOT executed. This is a blocker — see Section 5, Branch 2. Decision time limit: 30 minutes to fix or fall back.

**If rank > 50:** Not a blocker, but note the rank for the demo narrative. Adjust the demo claim to "we agree on who's elite" vs a specific rank comparison.

**If not found:** Run the broader search:
```sql
SELECT t.team_name, rb.rating, rb.rank, rb.rank_band
FROM rankings_with_bands rb
JOIN teams t ON t.team_id = rb.team_id
WHERE rb.age_group = 'U13' AND rb.gender = 'F'
  AND (t.team_name ILIKE '%Football%Academy%' OR t.team_name ILIKE '%FA 12G%')
ORDER BY rb.rank;
```
If still nothing: switch to the top-rated U13G team as the demo anchor (Query 1D below).

---

### 1C: Top-10 U13G all have non-null bands

```sql
SELECT t.team_name, rb.rating, rb.rank, rb.rank_band
FROM rankings_with_bands rb
JOIN teams t ON t.team_id = rb.team_id
WHERE rb.age_group = 'U13' AND rb.gender = 'F'
  AND rb.rank <= 10
ORDER BY rb.rank;
```

**Expected:** All 10 rows have non-null `rank_band`. Gold count ≥ 3 in top 10.

**If any null bands in top 10:** The view logic has a gap. Flag for Presten — this is cosmetic, not a blocker. The demo can proceed by manually noting the gap.

**If Gold count < 3 in top 10:** The thresholds may need to shift. Compare top-1 team's rating: if it's below 1500, the Gold threshold (≥1500) is too high for this data. Adjust the demo narrative — show Silver as the top tier for this age group, not Gold. Do not change the threshold on demo day.

---

### 1D: No null bands in top 50, and backup anchor team identification

```sql
-- Count null bands in top 50
SELECT COUNT(*) AS null_band_count
FROM rankings_with_bands rb
WHERE rb.age_group = 'U13' AND rb.gender = 'F'
  AND rb.rank <= 50 AND rb.rank_band IS NULL;

-- Top 5 U13G teams (for backup demo anchor if Football Academy NJ is not found)
SELECT t.team_name, rb.rating, rb.rank, rb.rank_band
FROM rankings_with_bands rb
JOIN teams t ON t.team_id = rb.team_id
WHERE rb.age_group = 'U13' AND rb.gender = 'F'
  AND rb.rank <= 5
ORDER BY rb.rank;
```

**Record top-5 results here for demo day:**
- Rank 1: _____________________ | Rating: _____ | Band: _____
- Rank 2: _____________________ | Rating: _____ | Band: _____
- Rank 3: _____________________ | Rating: _____ | Band: _____

If Football Academy NJ is missing, use Rank 1 team as the demo anchor. The demo claim shifts from "we agree with USA Rank on Football Academy NJ" to "here's our top-ranked U13G team and here's their full history."

---

## Step 2: Event Strength Validation

These queries confirm `event_strength` is populated and identify the specific High-strength event for the demo.

---

### 2A: Table exists and has current-season data

```sql
SELECT COUNT(*) AS event_count, MIN(season) AS min_season, MAX(season) AS max_season
FROM event_strength
WHERE season = '2025-2026';
```

**Expected:** > 50 events for 2025-2026.

**If 0:** Event Strength was NOT implemented. Cut Step 3 from the demo. See Section 5, Branch 3.

---

### 2B: Find the High-strength event for the demo

```sql
SELECT
  e.event_name,
  e.start_date,
  es.age_group,
  es.gender,
  es.team_count,
  ROUND(es.avg_team_rating::numeric, 0)         AS avg_rating,
  ROUND(es.rating_spread::numeric, 0)           AS spread_stddev,
  ROUND((es.upset_rate * 100)::numeric, 1)      AS upset_pct,
  es.strength_label,
  ROUND(es.strength_percentile::numeric, 0)     AS strength_pctile
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE es.season = '2025-2026'
  AND es.gender = 'F'
  AND es.age_group IN ('U13', 'U14')
  AND es.strength_label = 'High'
  AND es.team_count >= 8
  AND e.event_name NOT ILIKE '%friendly%'
  AND e.event_name NOT ILIKE '%training%'
ORDER BY es.avg_team_rating DESC
LIMIT 10;
```

**From this list, choose the event with the most recognizable name to a NJ/PA/NE audience.** Prefer:
1. Any ECNL Regional or National event name
2. Any Girls Academy National/Regional event name
3. Any named invitational with a geographic or sponsor name (avoid generic "Tournament 12345")

**Record the chosen event here:**
- Event name: _____________________________________
- Start date: _____________________
- Age group: _______ | Gender: ___
- Avg rating: _____ | Strength: _____ | Percentile: _____th | Spread StdDev: _____ | Upset %: _____%

---

### 2C: Verify the chosen event has complete data

```sql
-- Substitute the event name from 2B
SELECT
  e.event_name,
  e.start_date,
  es.team_count,
  es.avg_team_rating,
  es.rating_spread,
  es.upset_rate,
  es.strength_label,
  es.spread_label,
  es.upset_label,
  es.strength_percentile,
  es.computed_at
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE e.event_name = '[INSERT CHOSEN EVENT NAME FROM 2B]';
```

**Expected:** All metric columns non-null. `upset_rate` between 0.05 and 0.60. `rating_spread` > 0.

**If any metric is null:** This event had an edge case in computation. Pick the next event from Query 2B's result list.

---

## Step 3: Club Rankings Validation

These queries confirm `club_rankings` is populated and identify the primary demo club.

---

### 3A: Table has data for both genders

```sql
SELECT gender, COUNT(*) AS club_count
FROM club_rankings
GROUP BY gender;
```

**Expected:** > 50 clubs for 'F', > 50 clubs for 'M'.

**If 0 for both:** Club Rankings was NOT implemented. Cut Step 4 from the demo. See Section 5, Branch 4.

---

### 3B: PA Classics presence and rank check

```sql
SELECT
  c.name AS club_name,
  cr.club_rank,
  ROUND(cr.club_rating::numeric, 0) AS club_rating,
  cr.age_groups_counted,
  cr.computed_at
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE cr.gender = 'F'
  AND (
    c.name ILIKE '%PA Classics%'
    OR c.name ILIKE '%Pennsylvania Classics%'
  )
ORDER BY cr.club_rank;
```

**Expected:** PA Classics present, `club_rank` ≤ 30, `age_groups_counted` ≥ 5.

**If not present OR rank > 30:** Run the backup club check (3C).

---

### 3C: Backup club check

```sql
SELECT
  c.name AS club_name,
  cr.club_rank,
  ROUND(cr.club_rating::numeric, 0) AS club_rating,
  cr.age_groups_counted
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE cr.gender = 'F'
  AND (
    c.name ILIKE '%PDA%'
    OR c.name ILIKE '%NJ Wildcats%'
    OR c.name ILIKE '%Eclipse%'
    OR c.name ILIKE '%FC Dallas%'
    OR c.name ILIKE '%Bethesda%'
  )
ORDER BY cr.club_rank
LIMIT 10;
```

Pick the highest-ranked club the DSS audience (NJ/PA/NE corridor) is likely to recognize. Eclipse Select is nationally known but is Chicago-based — if the audience is primarily northeast, PDA or NJ Wildcats is a better choice.

**Record the chosen club:**
- Club name: _______________________
- Club rank: _____ | Rating: _____ | Age groups: _____

---

### 3D: Club profile depth check

```sql
-- Confirm the chosen club has a usable profile (top_teams JSONB populated)
SELECT
  c.name,
  cr.club_rank,
  cr.club_rating,
  cr.age_groups_counted,
  cr.top_teams  -- should be non-null JSONB with team breakdowns
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE c.name ILIKE '%[CHOSEN CLUB NAME]%'
  AND cr.gender = 'F';
```

**Expected:** `top_teams` is non-null JSON. If it contains team names and ratings, the club profile page will render. If `top_teams` is null, the profile page may be empty — this is a fallback-worthy issue for the demo (show the rankings table but skip the club profile click-through).

---

## Step 4: Coverage Claim Validation

These numbers go into the demo's closing statement. Run them now; write the exact claim language before demo day.

---

### 4A: How many U13G teams do we actively rank?

```sql
SELECT COUNT(*) AS active_ranked_u13g
FROM rankings r
JOIN teams t ON t.team_id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND r.last_game_date >= NOW() - INTERVAL '7 months';
```

**Record the number:** _______ actively ranked U13G teams.

---

### 4B: Total ranked vs active (for "we go deep" differentiation)

```sql
SELECT
  COUNT(*) AS total_ranked,
  COUNT(CASE WHEN r.last_game_date >= NOW() - INTERVAL '7 months' THEN 1 END) AS active_ranked,
  COUNT(CASE WHEN r.last_game_date < NOW() - INTERVAL '7 months' THEN 1 END) AS inactive_ranked
FROM rankings r
WHERE r.age_group = 'U13' AND r.gender = 'F';
```

Use **active_ranked** in the demo. The demo claim: "We actively rank **[4A count]** U13G teams." USA Rank is not publicly known to share their count, so the comparison is directional: "More than USA Rank's list of publicly named teams."

---

### 4C: USYS / state league teams presence (if FORGE completed org-ID expansion)

```sql
-- Check if state league teams appear in U13G rankings
-- (Adjust source pattern based on actual source values in the DB)
SELECT COUNT(*) AS state_league_count
FROM rankings r
JOIN teams t ON t.team_id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND r.last_game_date >= NOW() - INTERVAL '7 months'
  AND (
    t.team_name ILIKE '%State Cup%'
    OR t.team_name ILIKE '%State League%'
    OR EXISTS (
      SELECT 1 FROM games g
      WHERE (g.home_team_id = t.team_id OR g.away_team_id = t.team_id)
        AND g.source = 'sinc_sports'
        AND g.is_hidden = false
    )
  );
```

**If count = 0:** Do NOT claim state league coverage in the demo. Use only: "We rank more teams than USA Rank publicly names, covering all ECNL, GA, and MLS NEXT data."

**If count > 50:** The state league claim is valid. Adjust demo Step 5 to: "We rank [count] state league teams in U13G alone — USA Rank doesn't cover these."

---

### 4D: Write the exact coverage claim language

Fill this in now (not on demo day):

```
Demo Step 5 coverage claim (fill in from query results above):

"We actively rank [___] U13G teams. That includes every ECNL team,
every GA team, every MLS NEXT team. [IF state league count > 50:
And [___] state league teams USA Rank has no data on.]
Our accuracy audit shows our top-50 rankings closely match USA Rank's
for the teams we share — the gap is source coverage, not methodology."
```

---

## Step 5: Fallback Decision Tree

Run through this only if any validation step above fails its expected result.

### Branch 1 — Rank Bands missing (Query 1A returned 0)

Rankings page has no bands. Fall back to the Section 5 demo from the Demo Spec:
- Step 1: Show rankings page as-is, describe what bands WILL look like
- Step 2: Team detail page for Football Academy NJ (full game history is still the differentiator)
- Step 3: Skip event strength (requires bands context to land cleanly)
- Step 4: Skip club rankings
- Close with coverage claim only

**Decision time limit:** Do NOT attempt to implement Rank Bands on May 14. The 5.5-hour implementation cannot be completed in a morning with DSS 48 hours away. Fall back.

---

### Branch 2 — Football Academy NJ appears as 2 records (merge not executed)

This is a 20-minute fix if the merge SQL from `Pre-DSS Team Merge Audit — U13G.md` Section 3 is ready:

```bash
# Run from the merge audit document — this should be pre-staged
# 1. Execute the INSERT INTO team_merges statement
# 2. Run compute-rankings.js for U13G
node scripts/compute-rankings.js  # or scoped if supported
# 3. Re-run Query 1B to confirm single record
```

**Decision time limit:** If the merge completes and Query 1B returns a single record within 30 minutes: continue with the full demo. If the ranking recompute takes longer than 90 minutes or the merge introduces unexpected results: use the backup anchor team from Query 1D and continue. Do not block the full demo on this one team.

---

### Branch 3 — Event Strength missing (Query 2A returned 0)

Cut Step 3 from the demo entirely. The 3-step fallback (Rank Bands + Team Detail + Coverage Claim) is still compelling. Do not mention Event Strength.

**Revised 3-step demo:**
1. Rankings page → U13G top 10 with bands
2. Team detail → Football Academy NJ full game history
3. Coverage claim → "We rank [X] U13G teams"

---

### Branch 4 — Club Rankings missing (Query 3A returned 0)

Cut Step 4 from the demo. Remain on the 3-step fallback from Branch 3, or the 4-step flow if Event Strength is present:
1. Rankings + bands
2. Team detail
3. Event strength card (if present)
4. Coverage claim

---

### Branch 5 — All three features missing (Branches 1+3+4)

**Do not attempt to implement anything on May 14.** The demo is the fallback from Demo Spec Section 5:
1. Rankings page (unstyled, no bands) → "This is the engine. Bands, event strength, and club rankings are coming."
2. Team detail → full game history is still the accuracy story
3. Coverage comparison → "We rank more teams. Here's the proof."

This fallback is still compelling. The accuracy story — Glicko-2 vs last 20 games — does not depend on any feature implementation. The core claim holds with just the data.

---

## Step 6: Pre-Demo State Snapshot

Run this at the end of the May 14 validation session. Save the output. If anything breaks between May 14 and May 16, this is the baseline to compare against.

```sql
-- Save output: \o /tmp/dss-demo-snapshot-2026-05-14.txt
\o /tmp/dss-demo-snapshot-2026-05-14.txt
SELECT
  (SELECT COUNT(*) FROM rankings_with_bands WHERE age_group = 'U13' AND gender = 'F')
    AS u13g_band_count,
  (SELECT rb.rank_band FROM rankings_with_bands rb JOIN teams t ON t.team_id = rb.team_id
   WHERE rb.rank = 1 AND rb.age_group = 'U13' AND rb.gender = 'F' LIMIT 1)
    AS rank1_u13g_band,
  (SELECT COUNT(*) FROM event_strength WHERE season = '2025-2026')
    AS event_strength_count,
  (SELECT COUNT(*) FROM club_rankings WHERE gender = 'F')
    AS club_count_girls,
  (SELECT COUNT(*) FROM rankings WHERE age_group = 'U13' AND gender = 'F'
   AND last_game_date >= NOW() - INTERVAL '7 months')
    AS active_u13g_teams,
  NOW() AS snapshot_time;
\o
```

**Record key values here after running:**
- U13G band count: _______
- Rank 1 U13G band: _______
- Event strength count: _______
- Girls club count: _______
- Active U13G teams: _______
- Snapshot time: _______

If these numbers change unexpectedly between May 14 and May 16, investigate before demo.

---

## Session Summary

Total estimated time: 60–90 minutes in psql.

| Query Group | Time | Key Decision |
|---|---|---|
| Step 1 (Rank Bands) | ~20 min | Is Football Academy NJ single-record + banded? |
| Step 2 (Event Strength) | ~15 min | Which High-strength event to name? |
| Step 3 (Club Rankings) | ~15 min | PA Classics or backup club? |
| Step 4 (Coverage claim) | ~10 min | Write exact claim language |
| Step 5 (Fallback decisions) | ~5 min | Only if triggered by failed queries |
| Step 6 (Snapshot) | ~5 min | Save baseline |

If finished by 10:30 AM May 14: all demo decisions are made. May 14 afternoon is free for implementation fixes if Branch 2 (merge) was triggered.

---

## Related Notes

- [[DSS Demo Spec — May 2026]] — the demo this plan validates; Step/Section numbers in this document map to that spec
- [[Pre-DSS Team Merge Audit — U13G]] — pre-written merge SQL for Branch 2
- [[Rank Bands — Implementation Plan]] — source of truth for `rankings_with_bands` view structure
- [[Event Strength Ratings — Implementation Plan]] — source of truth for `event_strength` table structure
- [[Club Rankings — Implementation Plan]] — source of truth for `club_rankings` table structure
- [[USA Rank Ranking Accuracy Audit — April 2026]] — Football Academy NJ SQL context
