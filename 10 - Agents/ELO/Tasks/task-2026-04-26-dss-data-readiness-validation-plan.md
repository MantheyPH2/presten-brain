---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-26
status: completed
completed: 2026-04-23
priority: high
due: 2026-04-30
deliverable: "02 - Tiger Tournaments/Projects/Product & Planning/DSS Data Readiness Validation Plan — May 14-15.md"
---

# Task: DSS Data Readiness Validation Plan — May 14–15

## Context

You wrote the DSS Demo Spec (`02 - Tiger Tournaments/Projects/Product & Planning/DSS Demo Spec — May 2026.md`). It names specific assets: Football Academy NJ as the U13G anchor team, PA Classics as the primary club, a High-strength event to be identified by SQL before May 14, and a 5-step demo flow. The spec also has a Data Readiness Checklist (Section 4).

What the spec does not have: the validation queries Presten runs on May 14–15 to confirm all of these are actually correct and clean in the database before walking into DSS on May 16.

This task: write that validation plan. Presten should be able to open this document on May 14 at 9 AM, run every query in sequence, and know within 90 minutes whether the demo is clean, what (if anything) needs fixing, and whether to fall back to the fallback demo.

The validation plan is ELO's responsibility because ELO knows what correct looks like. Presten knows how to run psql. You know what the data should show.

## What You Need to Do

### Step 1: Rank Bands Validation Queries

These queries confirm the `rankings_with_bands` view is correct and the demo teams have appropriate bands.

**1A: View exists and is populated**
```sql
SELECT COUNT(*) FROM rankings_with_bands WHERE age_group = 'U13G';
```
Expected: >200 rows. If 0, Rank Bands was not implemented. Fall back to Rank Bands fallback demo immediately.

**1B: Football Academy NJ band assignment**
```sql
SELECT t.name, r.rating, r.rank, rb.band, rb.within_band_rank
FROM rankings_with_bands rb
JOIN teams t ON t.id = rb.team_id
WHERE t.name ILIKE '%football academy%' AND rb.age_group = 'U13G'
ORDER BY rb.rank;
```
Expected: Football Academy NJ appears as a single record, rated ≥1350 (Silver minimum), ideally ≥1500 (Gold). If it appears as two records, the merge task (task-2026-04-25-team-merge-pre-dss-audit) was not executed — this is a blocker.

**1C: Top-10 U13G all have bands assigned**
```sql
SELECT t.name, r.rating, rb.band
FROM rankings_with_bands rb
JOIN teams t ON t.id = rb.team_id
WHERE rb.age_group = 'U13G' AND rb.rank <= 10
ORDER BY rb.rank;
```
Expected: all 10 have a non-null band. Gold count in top 10 ≥ 3 (if fewer than 3, thresholds may need adjustment — flag for Presten to decide, do not auto-adjust).

**1D: No null bands in top 50**
```sql
SELECT COUNT(*) FROM rankings_with_bands WHERE age_group = 'U13G' AND rank <= 50 AND band IS NULL;
```
Expected: 0. If >0, the view logic has a gap.

### Step 2: Event Strength Validation Queries

These confirm the `event_strength` table is populated for 2025–2026 events.

**2A: Table exists and has current-season data**
```sql
SELECT COUNT(*), MIN(season), MAX(season) FROM event_strength WHERE season = '2025-2026';
```
Expected: >50 events for the current season. If 0, Event Strength was not implemented. Fall back to Event Strength being cut from the demo.

**2B: Find the High-strength event to feature**

This query identifies the strongest tournament in the DB for U13G — the one to use in the demo:
```sql
SELECT e.event_name, es.avg_team_rating, es.strength_class, es.strength_percentile,
       es.competitive_spread, es.upset_rate, es.team_count
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE es.age_group = 'U13G'
  AND es.strength_class = 'High'
  AND es.team_count >= 8
  AND e.event_name NOT ILIKE '%friendly%'
ORDER BY es.strength_percentile DESC
LIMIT 10;
```
Output: this is your named High-strength event for the demo. Pick the one with the most recognizable name (prefer ECNL, GA, or named invitationals over generic tournament names). Record the exact name here for Section 2 of the demo spec.

**2C: Verify the chosen event has complete data**
```sql
SELECT es.*, e.event_name, e.start_date
FROM event_strength es
JOIN events e ON e.id = es.event_id
WHERE e.event_name = '[INSERT CHOSEN EVENT NAME]';
```
All four metrics should be non-null. Upset rate and competitive spread should be within expected ranges (upset_rate 10–60%, spread >0).

### Step 3: Club Rankings Validation Queries

These confirm the `club_rankings` table is populated and PA Classics (primary demo club) is present and correct.

**3A: Club rankings table has data**
```sql
SELECT COUNT(*), gender FROM club_rankings GROUP BY gender;
```
Expected: >50 clubs for girls, >50 for boys. If 0, Club Rankings was not implemented. Cut from demo.

**3B: PA Classics is present and plausible**
```sql
SELECT c.name, cr.club_rank, cr.club_rating, cr.age_groups_counted, cr.computed_at
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE c.name ILIKE '%PA Classics%' OR c.name ILIKE '%Pennsylvania Classics%'
  AND cr.gender = 'F';
```
Expected: PA Classics appears, rank in top 30, age_groups_counted ≥ 5. If not present, check NJ Wildcats and PDA as backup clubs.

**3C: Backup club check (if PA Classics is missing or rank > 30)**
```sql
SELECT c.name, cr.club_rank, cr.club_rating, cr.age_groups_counted
FROM club_rankings cr
JOIN clubs c ON c.id = cr.club_id
WHERE (c.name ILIKE '%PDA%' OR c.name ILIKE '%Wildcats%' OR c.name ILIKE '%Eclipse%')
  AND cr.gender = 'F'
ORDER BY cr.club_rank;
```
Pick the highest-ranked recognizable club from this list as the demo club if PA Classics is not usable.

### Step 4: Coverage Claim Validation

The demo claims Evo Draw covers more teams than USA Rank in U13G. Validate this claim is accurate before stating it.

**4A: How many U13G teams do we rank?**
```sql
SELECT COUNT(*) FROM rankings WHERE age_group = 'U13G' AND rank IS NOT NULL;
```

**4B: How many have played in last 7 months (active teams)?**
```sql
SELECT COUNT(*) FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13G'
  AND t.last_game_date >= NOW() - INTERVAL '7 months';
```

The coverage claim in the demo should use the active team count (4B), not the total. Write the exact claim language here once you have the numbers — e.g., "We actively rank [X] U13G teams. USA Rank lists approximately 1,200." Do not leave Presten to improvise this number live.

**4C: Verify USYS state league teams are represented**

If FORGE completed the GotSport org-ID expansion before May 14, validate that at least some state league teams appear in U13G rankings:
```sql
SELECT COUNT(*) FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13G'
  AND t.source ILIKE '%state%' OR t.source_id ILIKE '%usys%';
```
If 0, do not claim state league coverage in the demo.

### Step 5: Fallback Decision Tree

If any validation query returns an unexpected result, what does Presten do?

Write explicit decision logic:

```
IF rankings_with_bands empty → use fallback demo (Rank Bands only, no actual bands)
IF Football Academy NJ appears as 2 records → run merge SQL from task-2026-04-25 IMMEDIATELY; re-run 1B
IF event_strength empty → cut Event Strength card from demo flow
IF club_rankings empty OR PA Classics missing → cut Club Rankings OR use backup club
IF all three features missing → deliver the fallback demo from Demo Spec Section 5
```

Each branch must have a decision time limit. If Presten is running this on May 14 at 9 AM, decisions should be made by noon. If a fix requires more than 2 hours, fall back rather than fix — demo day is May 16.

### Step 6: Pre-Demo State Snapshot

Write a single query Presten runs at the end of the May 14 validation session and saves to a file. This is the known-good snapshot:

```sql
-- Save this output as: reports/dss-demo-snapshot-2026-05-14.txt
SELECT
  (SELECT COUNT(*) FROM rankings_with_bands WHERE age_group = 'U13G') as u13g_band_count,
  (SELECT band FROM rankings_with_bands WHERE rank = 1 AND age_group = 'U13G') as rank1_band,
  (SELECT COUNT(*) FROM event_strength WHERE season = '2025-2026') as event_strength_count,
  (SELECT COUNT(*) FROM club_rankings WHERE gender = 'F') as club_count_girls,
  NOW() as snapshot_time;
```

If anything breaks between May 14 and May 16, Presten has a baseline to compare against.

## What Done Looks Like

Deliverable: `02 - Tiger Tournaments/Projects/Product & Planning/DSS Data Readiness Validation Plan — May 14-15.md`

The document must:
- Have exactly the queries in Steps 1–4, pre-written with no placeholders except `[INSERT CHOSEN EVENT NAME]` in 2C (which gets filled after 2B is run)
- Include the fallback decision tree in Step 5
- Include the snapshot query in Step 6
- Read as a linear checklist — Presten runs queries top-to-bottom, makes decisions at each branch

Total estimated time to run: 60–90 minutes in psql. If Presten starts May 14 at 9 AM, all decisions are made by 10:30 AM.

## Success Criteria

- All six steps are present with no placeholder queries
- Football Academy NJ investigation query is pre-written and specific
- High-strength event identification query (2B) is complete and ready to run
- Fallback decision tree covers all three features failing independently and all three failing together
- The snapshot query is present for the May 14 end-of-session record
- Document requires zero additional research by Presten to execute

## Why This Is ELO's Task

The demo spec is ELO's document. The validation plan must be consistent with it — using the same named assets, the same claim thresholds, the same fallback logic. ELO wrote the spec; ELO closes the loop by writing the validation that confirms the spec is achievable.
