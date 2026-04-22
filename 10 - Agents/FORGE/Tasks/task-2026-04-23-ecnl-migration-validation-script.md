---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-23
status: completed
priority: high
---

# Task: Build the ECNL Migration Validation Script

## Context

ECNL migrates from TGS AthleteOne to GotSport in June 2026 — approximately 6 weeks from now. Strategic Insights Action Item 2 identifies this as a required pre-migration build. When ECNL data shifts from source priority 3 (TGS) to source priority 1 (GotSport API), approximately 91,401 ECNL games currently in the database will have a new source representation. Without a validation script, we will not know:

- Which historical TGS ECNL teams have been correctly linked to their GotSport team IDs
- Which TGS ECNL game records have a matching GotSport record (successful dedup) vs. which are orphaned
- Whether ECNL team ratings will carry forward correctly into the post-migration state

This script must be built BEFORE the ECNL migration happens, not after. If we build it after, we will be debugging blind.

## What to Build

### Script: `scripts/validate-ecnl-migration.js`

A validation script that can be run in two modes:
- `--mode=pre` — Runs against current state (TGS as primary ECNL source). Captures a baseline snapshot.
- `--mode=post` — Runs after GotSport picks up ECNL data. Compares against the baseline.

#### Pre-Migration Mode (`--mode=pre`)

Captures a snapshot of the current TGS ECNL dataset:

```javascript
// For each ECNL team currently sourced primarily from TGS:
// 1. Team ID, team name, total TGS game count
// 2. Most recent TGS game date
// 3. Current Glicko-2 rating and RD from rankings table
// 4. Any existing GotSport team_id link via team_merges table
// Write to: Logs/ecnl-migration/pre-migration-snapshot-{date}.json
```

Key query: identify all teams where >50% of rated games come from TGS source (`source IN ('tgs', 'tgs_athleteone')`).

#### Post-Migration Mode (`--mode=post`)

Compares current state against the pre-migration snapshot:

1. **Team continuity check:** For each team in the pre-migration snapshot, is there now a GotSport record with a verified team_merge link? Report: teams with confirmed link, teams without a GotSport match, teams with multiple potential GotSport candidates (ambiguous merge).

2. **Game record continuity check:** For each TGS ECNL game in the snapshot, is there a corresponding GotSport game record (same match, same teams, same date) that has superseded it via the dedup logic? Report: matched games, unmatched TGS games (orphaned — no GotSport counterpart), unmatched GotSport games (new games with no TGS history).

3. **Rating continuity check:** For teams that carried forward, did their Glicko-2 rating shift by more than 25 points after the source change? If so, flag them — a >25-point shift from source migration alone indicates a data quality problem, not legitimate competitive change.

4. **Output:** `Logs/ecnl-migration/post-migration-validation-{date}.json` with a summary section and per-team detail section. Also write a human-readable summary to `Reports/ecnl-migration-validation-{date}.md`.

### Supporting Work: Expand the Dedup Strategy for ECNL Transition

Read [[Dedup Strategy]] in the vault. The current dedup key for TGS games is birth-year-based division format. GotSport uses `gs_api_{eventId}_{matchId}`. When ECNL moves to GotSport, a new dedup pass is needed to link historical TGS records to their incoming GotSport counterparts.

Document in `Reports/ecnl-migration-validation-{date}.md`:
- The specific dedup logic that will apply during the ECNL transition period (when both TGS and GotSport emit ECNL data simultaneously)
- Any `source_id` format conflicts that will need resolution
- The recommended sequence: run the migration validation BEFORE running dedup, so orphaned TGS records are visible before they get hidden by the dedup logic

## What to Read

- `/Users/p/Documents/Obsidian Vault/02 - Tiger Tournaments/Projects/Knowledge/Data Sources Overview.md` — TGS game counts, source priority table
- `/Users/p/Documents/Obsidian Vault/02 - Tiger Tournaments/Projects/Strategic Insights.md` — Insight 1 (ECNL migration windfall) and Action Item 2
- `[[TGS AthleteOne]]` and `[[ECNL]]` vault notes for source-specific context
- `[[Dedup Strategy]]` for current dedup key formats and the `team_merges` table structure

## What to Produce

1. `scripts/validate-ecnl-migration.js` — working script with both modes
2. `Reports/ecnl-migration-validation-plan.md` — written explanation of what the script checks and why, for Presten to review before June 2026
3. Run the script in `--mode=pre` immediately to capture the pre-migration baseline snapshot. Store the output in `Logs/ecnl-migration/pre-migration-snapshot-2026-04-23.json`. This file is time-sensitive — it must exist before the migration happens.

## Definition of Done

- Script runs in both modes without errors on the production database
- Pre-migration snapshot file exists at `Logs/ecnl-migration/pre-migration-snapshot-2026-04-23.json`
- The validation plan document explains what constitutes a "successful" migration vs. one that requires intervention

## Priority Note

This is HIGH priority because the June 2026 deadline is fixed by external ECNL organizational decisions — it cannot be moved. The pre-migration snapshot cannot be retroactively created. If we miss capturing the baseline before ECNL migrates, the post-migration comparison becomes impossible.
