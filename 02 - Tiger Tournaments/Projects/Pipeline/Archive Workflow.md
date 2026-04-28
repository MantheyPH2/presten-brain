---
title: Archive Workflow
aliases: [archive-inactive-teams.js, Team Archival, archived_at]
tags: [pipeline, archival, schema, data-hygiene, evo-draw]
created: 2026-04-22
updated: 2026-04-27
author: FORGE
status: migration executed 2026-04-22 — archive script pending first production run — GROUP BY bug fixed in spec 2026-04-27
---

# Seasonal Team Archive Workflow

> [!success] Migration Executed
> The `archived_at` schema migration ran on production on 2026-04-22 (approved by Presten via CHIEF). All existing teams now have `archived_at = NULL`. The archive script (`archive-inactive-teams.js`) has not yet run its first production pass — the 180 de-prioritized team archival is the next step.

---

## Problem Statement

Every tournament season that concludes without an archival pass leaves orphaned `active` team records in the `teams` table. These ghost-active teams:

1. **Load into the pipeline unnecessarily.** Steps that filter on `active = true` (or `status = 'active'`) include these teams in processing passes where they add noise but contribute no signal.

2. **Distort Glicko-2 rating deviation.** The [[Ranking Engine]] interprets inactivity as "fewer matches played" rather than "data collection ended." For teams that are fully settled (no longer competing), their RD (rating deviation) increases inappropriately — indicating uncertainty in a rating that should be considered final for that team's competitive career on this platform.

3. **Inflate active team counts.** The 144,676 `teams` table entries include an unknown number of teams from concluded seasons. The true "actively competing" team count is lower, and the discrepancy grows with each season.

The immediate trigger: the 2026-04-21 stale team audit identified ~180 teams from concluded tournaments still marked `active`. These are the first confirmed cohort, but the pattern recurs every season.

---

## Part 1: Schema Change — `archived_at` Column

### Migration SQL

```sql
-- Add archived_at column to teams table
ALTER TABLE teams ADD COLUMN archived_at TIMESTAMP NULL DEFAULT NULL;

-- Index for performance (most queries filter on archived_at IS NULL)
CREATE INDEX idx_teams_archived_at ON teams (archived_at);
```

### Why a timestamp, not a boolean?

A nullable `archived_at` timestamp is strictly better than an `is_archived` boolean because:
- It records **when** the team was archived, enabling audit trails and rollback identification
- `WHERE archived_at IS NULL` is logically equivalent to `WHERE is_archived = false` but carries more information
- Partial archival passes can be queried by date range: `WHERE archived_at BETWEEN '2026-04-01' AND '2026-04-30'`

### New Standard "Active Only" Filter

All pipeline steps and queries that currently use `WHERE active = true` or `WHERE status = 'active'` must add:

```sql
WHERE archived_at IS NULL
```

This becomes the canonical "team is active" filter. The `active` column should be considered secondary once `archived_at` is deployed.

### Pipeline Steps Requiring the New Filter

| Step | File | Current Filter | Filter to Add |
|---|---|---|---|
| Step 6 | `dedup-and-link-teams.js` | `active = true` | `AND archived_at IS NULL` |
| Step 7 | `match-teams-cross-platform.js` | `active = true` | `AND archived_at IS NULL` |
| Step 8 | `compute-rankings.js` (Phase 1) | `active = true` in team merges loader | `AND t.archived_at IS NULL` |
| Scraper | `crawl-gotsport-api-parallel.js` | team queue population | `AND archived_at IS NULL` |
| Audit | `audit-data.js` | team count reporting | Add separate `archived_count` metric |

> [!tip] Backfill Safety
> After the migration runs, all existing teams will have `archived_at = NULL` (the default). No teams are archived by the migration itself — it is purely additive. The archival script runs separately.

---

## Part 2: Archival Logic Specification

### Script: `archive-inactive-teams.js`

#### Archival Trigger Criteria

A team is a candidate for archival if ALL three conditions are true:

1. **Most recent game is more than 180 days ago**
   ```sql
   MAX(g.match_date) < NOW() - INTERVAL '180 days'
   ```

2. **The event that most recent game belongs to ended more than 90 days ago**
   — Uses `event_end_date` (if available in `games` or a joined `events` table)
   — If `event_end_date` is not populated, fall back to: `MAX(g.match_date) < NOW() - INTERVAL '90 days'` as a proxy for event conclusion

3. **Team has not appeared in any sync window in the last 30 days**
   — Verified via: `teams.last_synced_at < NOW() - INTERVAL '30 days'`
   — This confirms GotSport is not actively returning data for this team even when queried

#### Safety Guard: Do Not Archive Cross-Active Teams

Before archiving any candidate, check whether the team has played in an active event in the last 90 days:

```sql
SELECT 1 FROM games g2
WHERE (g2.home_team_id = t.team_id OR g2.away_team_id = t.team_id)
  AND g2.match_date >= NOW() - INTERVAL '90 days'
LIMIT 1
```

If this returns any result, skip archival — the team is still competing in an ongoing event even if one of their past events has concluded. This is the critical guard for teams registered in both a concluded tournament and an active league.

#### Full Candidate Query

> [!warning] Bug Fixed 2026-04-27
> Prior version included `g.event_name` in both `SELECT` and `GROUP BY`. This caused a team with games in multiple events to produce multiple rows — one per event — instead of one row per team. During a dry-run this inflated the candidate count and surfaced duplicate team_ids. Fixed by removing `g.event_name` from the GROUP BY and replacing it with a subquery scalar so one row per team is guaranteed. SENTINEL flagged this bug during the 2026-04-22 review. The fix in this spec must also be applied to the `archive-inactive-teams.js` script before the first dry-run runs.

```sql
SELECT
  t.team_id,
  t.team_name,
  t.active,
  t.last_synced_at,
  MAX(g.match_date) AS last_game_date,
  COUNT(g.id) AS total_games,
  (
    SELECT g2.event_name
    FROM games g2
    WHERE (g2.home_team_id = t.team_id OR g2.away_team_id = t.team_id)
    ORDER BY g2.match_date DESC
    LIMIT 1
  ) AS last_event_name
FROM teams t
JOIN games g ON (g.home_team_id = t.team_id OR g.away_team_id = t.team_id)
  AND g.is_hidden = false
WHERE
  t.archived_at IS NULL              -- not already archived
  AND t.active = true                -- currently considered active
  AND t.last_synced_at < NOW() - INTERVAL '30 days'  -- not recently synced
GROUP BY t.team_id, t.team_name, t.active, t.last_synced_at
HAVING MAX(g.match_date) < NOW() - INTERVAL '180 days'
  AND NOT EXISTS (
    SELECT 1 FROM games g2
    WHERE (g2.home_team_id = t.team_id OR g2.away_team_id = t.team_id)
      AND g2.match_date >= NOW() - INTERVAL '90 days'
  );
```

#### Script Logic (Pseudocode)

```javascript
async function archiveInactiveTeams() {
  const runId = `archive-${Date.now()}`;
  logger.info({ runId, event: 'ARCHIVE_RUN_START' });

  // Step 1: Identify candidates
  const candidates = await db.query(CANDIDATE_QUERY);
  logger.info({ runId, event: 'CANDIDATES_IDENTIFIED', count: candidates.rowCount });

  // Step 2: Log candidate list — NEVER archive without a log
  const candidateLog = candidates.rows.map(r => ({
    team_id: r.team_id,
    team_name: r.team_name,
    last_game_date: r.last_game_date,
    last_event_name: r.last_event_name,
    total_games: r.total_games,
    last_synced_at: r.last_synced_at
  }));
  await fs.writeFile(
    `Logs/archive-runs/${runId}-candidates.json`,
    JSON.stringify(candidateLog, null, 2)
  );

  // Step 3: Execute archival
  if (candidates.rowCount === 0) {
    logger.info({ runId, event: 'ARCHIVE_RUN_COMPLETE', archived: 0 });
    return;
  }

  const teamIds = candidates.rows.map(r => r.team_id);
  const result = await db.query(
    `UPDATE teams SET archived_at = NOW() WHERE team_id = ANY($1) AND archived_at IS NULL`,
    [teamIds]
  );

  // Step 4: Output summary
  const summary = {
    run_id: runId,
    candidates_identified: candidates.rowCount,
    teams_archived: result.rowCount,
    teams_skipped: candidates.rowCount - result.rowCount
  };
  logger.info({ runId, event: 'ARCHIVE_RUN_COMPLETE', ...summary });
  await fs.writeFile(
    `Logs/archive-runs/${runId}-summary.json`,
    JSON.stringify(summary, null, 2)
  );
}
```

---

## Part 3: Pipeline Integration Point

### Where in `run-pipeline.sh`

The archive step runs **between Step 5 and Step 6** — specifically, immediately before `dedup-and-link-teams.js`:

```bash
# run-pipeline.sh (proposed sequence with archive step)
bash scripts/cleanup-and-normalize.js        # Step 1
bash scripts/backfill-age-gender.js          # Step 2
bash scripts/final-backfill.js               # Step 3
bash scripts/backfill-all-games.js           # Step 4
bash scripts/fix-api-gender.js               # Step 5

# --- NEW: Archive step ---
node scripts/archive-inactive-teams.js       # Step 5.5 — archive before dedup

bash scripts/dedup-and-link-teams.js         # Step 6 (now excludes archived teams)
bash scripts/match-teams-cross-platform.js   # Step 7
bash scripts/compute-rankings.js             # Step 8
bash scripts/audit-data.js                   # Step 9
```

### Why Before Step 6

- **Dedup (Step 6)** processes the active team pool to resolve cross-source duplicates. Archived teams should not participate in this pass — they would generate spurious team merge candidates and slow the dedup query.
- **Team merges loader (Phase 1 of the ranking engine, Step 8)** reads the `team_merges` table. Archived teams that were previously merged with active teams should have their merge records excluded. By the time Step 8 runs, archived teams are already filtered via `archived_at IS NULL`, so Phase 1 cleanly excludes them.
- Running the archive step **after** Step 5 means the normalization passes (Steps 1–5) have already cleaned and standardized all game data. This gives `archive-inactive-teams.js` the cleanest possible view of each team's game history for the candidacy check.

### Frequency

The archive step does not need to run on every pipeline execution. Recommended schedule:
- **Weekly** during active tournament season (Sep–Jun)
- **Monthly** during off-season

A flag in `run-pipeline.sh` controls this:
```bash
ARCHIVE_STEP=${ARCHIVE_STEP:-false}   # Default off; set to true for weekly runs
if [ "$ARCHIVE_STEP" = "true" ]; then
  node scripts/archive-inactive-teams.js
fi
```

---

## Part 4: Immediate Action — The 180 De-Prioritized Teams

### Background

The 2026-04-21 stale team audit identified ~180 teams from concluded tournaments that remain `active` in the `teams` table. These are teams from events that the sync scheduler de-prioritized (events ended >60 days ago) but never formally archived. They are not receiving fresh data from GotSport because GotSport itself shows no new games — these teams are genuinely done competing in those events.

### Pre-Archival Verification Required

Before the `archive-inactive-teams.js` script can be run on these 180 teams, two checks must be completed:

**Check 1: Cross-active tournament participation**
Run the safety guard query against all 180 team IDs to identify any team that has also competed in an active event in the last 90 days. These teams must be excluded from the initial archival pass.

**Check 2: Event end date documentation**
Confirm which tournaments these 180 teams belong to and verify that those tournaments have genuinely concluded (not just de-prioritized mid-tournament). Pull `event_name` from the `games` table for these team IDs and cross-reference against GotSport event records.

### Recommended Execution Sequence (Once Schema Migration is Approved)

1. Run migration: `ALTER TABLE teams ADD COLUMN archived_at TIMESTAMP NULL DEFAULT NULL;`
2. Create index: `CREATE INDEX idx_teams_archived_at ON teams (archived_at);`
3. Update all pipeline step queries to add `AND archived_at IS NULL`
4. Run `archive-inactive-teams.js` in `--dry-run` mode first (outputs candidate list without executing `UPDATE`)
5. Review candidate list with Presten
6. Execute archival for confirmed candidates

**Estimated safe archival count:** ~155–165 of the 180 (after excluding cross-active teams)

---

## Impact on the Ranking Engine

Glicko-2 RD (rating deviation) increases for teams that are not playing, which models uncertainty correctly for active teams between matches. But for teams that have permanently stopped competing, a steadily increasing RD is misleading — it signals "we're increasingly uncertain about this team" rather than "this team has graduated from the system."

By archiving concluded-season teams:
- They are excluded from Phase 1 team merges loading
- Their RD does not continue inflating in the rankings table
- The rankings output reflects only teams currently in competition, which is the semantically correct population for the ranking system

---

## Related Notes
- [[Data Pipeline]] — pipeline step sequence
- [[Database Schema]] — `teams` table structure
- [[Ranking Engine]] — Phase 1 team merges loading
- [[Dedup Strategy]] — how team merges interact with active teams
- `Reports/stale-teams-2026-04-21.csv` — source of the 180 de-prioritized team list
- `Reports/stale-teams-unexplained-2026-04-22.md` — the 127 unexplained stale teams investigation
