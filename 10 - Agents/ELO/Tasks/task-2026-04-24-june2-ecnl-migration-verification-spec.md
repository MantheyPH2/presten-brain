---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
deliverable: "02 - Tiger Tournaments/Projects/Rankings/June 2 ECNL Migration Verification Spec.md"
priority: high
due: 2026-05-05
---

# Task: June 2 ECNL Migration Verification Spec — Day-After Checks

## Context

The June 1 Risk Assessment (`Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md`) documents two concurrent risks:
- **Risk A:** Age group transition (all teams age up on June 1 — U12 teams become U13, etc.)
- **Risk B:** ECNL platform migration (TGS → GotSport; team IDs change)

The Risk Assessment identifies what could go wrong. The Calibration Production Runbook covers how to apply parameter changes and monitor them. The ECNL dedup task (now confirmed by FORGE) addresses the code fix for the `verified = true` bug.

What does not exist: **the June 2 verification spec** — the specific queries and checks Presten runs on the morning of June 2 to confirm that both migrations completed correctly overnight, that no team lost its game history, and that rankings are plausible post-transition.

The June 1 transition is in 38 days. SENTINEL does not want this spec to be written under time pressure in late May. It should exist now so that if issues are identified with the spec, there is time to address the underlying data problems before June 1.

## What You Need to Do

### Step 1: Define the Expected State After June 1

On June 2 morning, if everything worked correctly:

**Age group transition (Risk A):**
- Every team should now appear in their new age group (U12 → U13, U13 → U14, etc.)
- Teams' game histories from the prior year should still be associated correctly
- Team ratings in the new age group should be plausible given the transition mechanism
- No team should appear in both their old AND new age group

**ECNL migration (Risk B):**
- ECNL teams that migrated from TGS to GotSport should have a `team_merges` record mapping old_id → new_id
- The `verified = true` flag must be present on all migration merges (FORGE confirmed the fix)
- ECNL teams' game histories (under their old TGS IDs) should still count toward their new GotSport ID ratings
- No ECNL team should show a rating of ~1000 (Glicko default) if they had established ratings before the migration

Document these expectations as acceptance criteria:
| Check | Expected State | Failure State |
|---|---|---|
| Age group transition completeness | 0 teams in both old and new age group | >0 teams appear in both |
| ECNL merge coverage | merge records exist for [X]% of known ECNL teams | <80% of ECNL teams have merge records |
| Rating continuity | No ECNL team drops to ~1000 rating after migration | Any ECNL team previously rated >1200 now shows <1100 |
| Game history retention | Total game count unchanged (or increases) | Game count drops significantly |

### Step 2: Write the June 2 Verification Queries

**2A: Age group transition completeness check**
```sql
-- Check for teams in both old and new age groups post-transition
-- (Only relevant if rankings are stored with age_group field that should have updated)
SELECT t.id, t.name, array_agg(DISTINCT r.age_group) as age_groups
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group IN ('U12', 'U13', 'U14', 'U15', 'U16', 'U17')
GROUP BY t.id, t.name
HAVING COUNT(DISTINCT r.age_group) > 1
LIMIT 20;
```
Expected: few or no teams (some teams legitimately compete in multiple age groups). If hundreds of teams appear, the transition produced duplicates.

**2B: ECNL migration merge coverage**
```sql
-- How many team_merges records were inserted with method = 'ecnl-migration' or similar?
SELECT COUNT(*), verified, method
FROM team_merges
WHERE method ILIKE '%ecnl%' OR method ILIKE '%migration%'
GROUP BY verified, method;
```
Expected: the `verified = true` group should have the bulk of records (500–2,000 based on Risk Assessment estimates). If `verified = false` or NULL appears, the dedup fix did not run correctly — this is the Risk B failure scenario.

**2C: Rating continuity check — ECNL teams post-migration**
```sql
-- Find ECNL teams with suspiciously low ratings post-migration
-- These teams had high ratings before; if they're now at ~1000, migration failed
SELECT t.name, r.rating, r.rank, r.age_group, r.gender
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group IN ('U13', 'U14', 'U15')
  AND r.rating BETWEEN 990 AND 1010  -- Glicko default ≈ 1000
  AND t.name ILIKE '%ECNL%'  -- crude filter; adapt to actual naming conventions
ORDER BY r.rank ASC NULLS LAST
LIMIT 20;
```
Note: this query is a crude first pass. Adapt it after checking what naming patterns ECNL teams use in the DB.

**2D: Game history retention check**
```sql
-- Total game count before and after migration (compare to a saved baseline)
-- Run this before June 1 and save the result; re-run on June 2 to compare
SELECT COUNT(*) as total_games, COUNT(DISTINCT home_team_id) + COUNT(DISTINCT away_team_id) as unique_teams_in_games
FROM games;
```
The game count should not decrease. If it does, games may have been orphaned during the migration.

**2E: Rankings plausibility check for top teams**
```sql
-- The top 10 teams should not have changed dramatically
SELECT t.name, r.rating, r.rank, r.age_group, r.gender
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND r.rank <= 15
ORDER BY r.rank;
```
Save the pre-June 1 top 10. Compare on June 2. Any team previously outside the top 30 now in the top 10 warrants investigation.

### Step 3: Write the Pre-June-1 Baseline Snapshot

Presten must run this on May 31 (the day BEFORE migration) and save the output:

```sql
-- SAVE THIS OUTPUT — required for June 2 comparison
-- Run on May 31 before any migration scripts execute

SELECT
  (SELECT COUNT(*) FROM games) as total_games,
  (SELECT COUNT(*) FROM teams) as total_teams,
  (SELECT COUNT(*) FROM rankings) as total_rankings,
  (SELECT COUNT(*) FROM team_merges WHERE verified = true) as verified_merges,
  (SELECT AVG(rating) FROM rankings WHERE age_group = 'U13' AND gender = 'F') as u13g_avg_rating,
  (SELECT AVG(rating) FROM rankings WHERE age_group = 'U14' AND gender = 'F') as u14g_avg_rating,
  (SELECT rating FROM rankings WHERE age_group = 'U13' AND gender = 'F' ORDER BY rank LIMIT 1) as u13g_rank1_rating,
  NOW() as snapshot_time;
```

Save to: `Reports/pre-migration-snapshot-2026-05-31.txt`

On June 2, compare against this baseline. Any significant deviation (total_games decreasing, avg_rating shifting >50 points, rank-1 rating dropping >100 points) is a red flag.

### Step 4: Define the Rollback Decision Tree

If June 2 verification reveals problems:

```
IMMEDIATE ROLLBACK TRIGGERS (act within 4 hours):
1. Any previously-rated ECNL team (rating > 1200) now shows rating < 1100
2. Total game count has decreased from the May 31 baseline
3. ECNL migration team_merges all show verified = false or null

INVESTIGATE (do not rollback immediately):
1. A few teams appear in two age groups (could be intentional cross-age-group competition)
2. Top 10 shuffle is significant but teams are plausibly strong (new ECNL teams from migration)
3. avg_team_rating for U13G has shifted by ±30–50 points (age group transition effect is expected)

SAFE TO PROCEED:
1. Top 10 is stable (same teams ± 3 positions)
2. total_games equals or exceeds baseline
3. ECNL merge records are all verified = true
4. No team shows an unexplained reset to ~1000 rating
```

State what "rollback" means in each scenario:
- For age group transition: likely no rollback possible (it's a date-based system change)
- For ECNL migration: rollback = delete the bad team_merge records, re-run with the corrected script

### Step 5: Document the ECNL Team Identifier List

For the verification to work, ELO needs a list of "known ECNL teams" to spot-check. This list should be compiled before June 1 so the June 2 verification can check specific teams rather than relying on name pattern matching.

Write a SQL query to generate the list of ECNL teams currently in the DB that are expected to migrate:
```sql
-- Teams associated with ECNL events (pre-migration)
SELECT DISTINCT t.id, t.name, MAX(r.rating) as current_max_rating
FROM games g
JOIN events e ON e.id = g.event_id
JOIN teams t ON t.id = g.home_team_id
JOIN rankings r ON r.team_id = t.id
WHERE e.event_tier IN ('ecnl', 'ecnl_regional')
GROUP BY t.id, t.name
ORDER BY current_max_rating DESC
LIMIT 100;
```

Presten runs this before June 1. The output is the spot-check list for June 2. If a team on this list shows a rating drop >150 points on June 2, that's the specific team to investigate.

## Deliverable

Write the verification spec to:
`02 - Tiger Tournaments/Projects/Rankings/June 2 ECNL Migration Verification Spec.md`

Structure as two sections:
- **Section 1: Pre-Migration (May 31)** — what Presten saves before migration
- **Section 2: Post-Migration (June 2)** — queries to run in sequence, decision tree

## Definition of Done

- 5 verification queries are written and schema-correct
- Pre-June-1 baseline snapshot query is present
- Rollback triggers are numeric and explicit
- ECNL team identifier list query is present
- Distinction between Risk A (age group) and Risk B (ECNL migration) is maintained throughout
- Summary in briefing: verification spec ready; May 31 baseline snapshot must be scheduled

## Why This Matters

June 1 is the highest-risk day in the Evo Draw calendar. Two structural changes happen simultaneously. The June 1 Risk Assessment documents the risk. The ECNL dedup fix addresses the code risk. This task closes the last gap: what does Presten do at 9 AM on June 2 to confirm nothing broke overnight? Without this spec, that morning is reactive improvisation. With it, it's a 45-minute checklist.

## References

- `02 - Tiger Tournaments/Projects/Rankings/June 1 Risk Assessment — Age Group Transition + ECNL Migration.md` — complete risk inventory
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — team_merges, rankings, games schemas
- `Reports/ecnl-dedup-verified-fix-2026-04-24.md` — FORGE's dedup fix confirmation (Risk B mitigation)
- `02 - Tiger Tournaments/Projects/Rankings/Calibration Production Runbook — May 2026.md` — rollback patterns (reuse for ECNL rollback)
