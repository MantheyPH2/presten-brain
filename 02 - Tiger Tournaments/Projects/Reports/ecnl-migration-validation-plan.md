---
title: ECNL Migration Validation Plan
tags: [report, ecnl, migration, validation, tgs, gotsport, pipeline, evo-draw]
created: 2026-04-23
updated: 2026-04-23
author: FORGE
status: active — pre-migration baseline captured
priority: high
deadline: before June 2026
---

# ECNL Migration Validation Plan

**Analyst:** FORGE  
**Date:** 2026-04-23  
**Deadline:** Must be fully operational before June 2026 ECNL platform migration  
**Stakes:** 91,401 ECNL games shifting from source priority 3 (TGS) to source priority 1 (GotSport API). Without this plan, the migration runs blind.

> [!warning] Time-Sensitive
> The pre-migration baseline snapshot must exist BEFORE the ECNL migration occurs. It cannot be retroactively created. As of 2026-04-23, the snapshot has been captured. **Do not delete or overwrite `Logs/ecnl-migration/pre-migration-snapshot-2026-04-23.json`.**

---

## Why This Validation Is Necessary

ECNL is Evo Draw's entire TGS data footprint — 91,401 of 91,401 TGS games are ECNL games. When ECNL migrates to GotSport in June 2026, three things happen simultaneously:

1. **Source priority shift:** ECNL games move from source priority 3 (TGS) to priority 1 (GotSport API). The GotSport versions become canonical; TGS versions get marked `is_hidden = true` by the dedup logic.
2. **Source ID format change:** TGS records use `tgs_{eventId}_{gameId}`. GotSport records use `gs_api_{eventId}_{matchId}`. These are different identifiers for the same games — the dedup logic must bridge them.
3. **Data quality upgrade:** GotSport provides structured team IDs, stable match identifiers, and better gender metadata. But only if the dedup correctly links historical TGS records to their GotSport counterparts.

If the dedup fails silently, ECNL teams will appear to have two separate rating histories — one from TGS (now hidden), one from GotSport (new) — and their Glicko-2 ratings will reset as if they were new teams. This is the failure mode the validation script is designed to catch.

---

## The Validation Script: `scripts/validate-ecnl-migration.js`

### Overview

The script runs in two modes, separated in time by the migration event:

```
BEFORE MIGRATION                    AFTER MIGRATION
     │                                     │
     ▼                                     ▼
--mode=pre                           --mode=post
Captures baseline snapshot           Compares current state
of all TGS ECNL teams                against the baseline
     │                                     │
     ▼                                     ▼
Logs/ecnl-migration/                 Logs/ecnl-migration/
pre-migration-snapshot-              post-migration-validation-
2026-04-23.json          ────────►   {date}.json
                          compare
```

---

### Pre-Migration Mode (`--mode=pre`)

**Run date:** 2026-04-23 (completed — see snapshot file)

**What it captures for each ECNL team:**

```javascript
// For each team where >50% of rated games come from TGS:
{
  team_id: integer,                    // Internal team ID
  team_name: string,                   // Display name
  tgs_game_count: integer,             // Total games in TGS source
  most_recent_tgs_game_date: date,     // Last TGS game date
  glicko2_rating: float,               // Current rating from rankings table
  glicko2_rd: float,                   // Rating deviation
  gotsport_team_id: integer | null,    // GotSport ID if linked via team_merges
  team_merge_status: string            // 'verified' | 'unverified' | 'none'
}
```

**Key query — identifying TGS-primary ECNL teams:**

```sql
SELECT 
  t.team_id,
  t.team_name,
  COUNT(CASE WHEN g.source IN ('tgs', 'tgs_athleteone') THEN 1 END) AS tgs_game_count,
  MAX(CASE WHEN g.source IN ('tgs', 'tgs_athleteone') THEN g.game_date END) AS most_recent_tgs_game_date,
  r.glicko2_rating,
  r.glicko2_rd,
  tm.gotsport_team_id,
  tm.verified AS team_merge_status
FROM teams t
JOIN games g ON (g.home_team_id = t.team_id OR g.away_team_id = t.team_id)
LEFT JOIN rankings r ON r.team_id = t.team_id
LEFT JOIN team_merges tm ON tm.tgs_team_id = t.team_id
WHERE g.is_hidden = false
GROUP BY t.team_id, t.team_name, r.glicko2_rating, r.glicko2_rd, tm.gotsport_team_id, tm.verified
HAVING 
  COUNT(CASE WHEN g.source IN ('tgs', 'tgs_athleteone') THEN 1 END)::float 
  / COUNT(*) > 0.5;
```

**Output:** `Logs/ecnl-migration/pre-migration-snapshot-2026-04-23.json`

---

### Post-Migration Mode (`--mode=post`)

**Run date:** Within 7 days of ECNL migration going live on GotSport (target: first week of June 2026)

**Four checks, in order:**

#### Check 1 — Team Continuity

For each team in the pre-migration snapshot, verify there is now a GotSport record with a confirmed `team_merges` link.

**Output buckets:**
- `confirmed_link`: TGS team → GotSport team, merge verified ✓
- `no_gotsport_match`: TGS team exists but no GotSport counterpart found — **these teams will lose their rating history**
- `ambiguous_merge`: Multiple GotSport candidates; merge could not be auto-resolved — **requires manual review**

**Threshold for intervention:** If `no_gotsport_match` exceeds 5% of snapshot teams, halt the migration dedup pass and escalate to Presten. Rating continuity is broken at scale.

#### Check 2 — Game Record Continuity

For each TGS ECNL game in the snapshot, verify there is a corresponding GotSport record.

**Match key:** same home team + away team + game date (±1 day tolerance for timezone edge cases) + similar score.

**Output buckets:**
- `matched`: TGS game has a GotSport counterpart — dedup will hide the TGS version correctly
- `orphaned_tgs`: TGS game has no GotSport counterpart — this game will be hidden by dedup but has no replacement. **Historical data is lost from rankings.**
- `new_gotsport_only`: GotSport has games with no TGS history — expected for games added to GotSport retroactively

**Threshold for intervention:** If `orphaned_tgs` exceeds 3% of TGS game volume (~2,742 games), the dedup reconciliation pass needs manual review before it runs.

#### Check 3 — Rating Continuity

For each team that successfully linked (confirmed in Check 1), compare the Glicko-2 rating before and after migration.

**Logic:**
```javascript
const ratingShift = Math.abs(postRating.glicko2_rating - snapshotTeam.glicko2_rating);
if (ratingShift > 25) {
  // Flag: a >25-point shift from source migration alone indicates a data problem
  // Legitimate competitive improvement/decline should not spike at migration time
  flaggedTeams.push({
    team_id: team.team_id,
    pre_rating: snapshotTeam.glicko2_rating,
    post_rating: postRating.glicko2_rating,
    shift: ratingShift,
    flag: 'RATING_CONTINUITY_VIOLATION'
  });
}
```

**Why 25 points:** The Glicko-2 RD for an active ECNL team should be ~60–80 points. A source change — not competitive results — should not move the rating by more than ~15–20 points. 25 points is a conservative threshold; violations above it indicate the new GotSport record and the old TGS record are not actually matching the same team's games.

#### Check 4 — Summary Report

Write both machine-readable and human-readable output:

**Machine:** `Logs/ecnl-migration/post-migration-validation-{date}.json`
```json
{
  "run_date": "2026-06-XX",
  "snapshot_date": "2026-04-23",
  "teams_in_snapshot": N,
  "check1_confirmed_link": N,
  "check1_no_gotsport_match": N,
  "check1_ambiguous_merge": N,
  "check2_games_matched": N,
  "check2_orphaned_tgs": N,
  "check2_new_gotsport_only": N,
  "check3_rating_violations": N,
  "intervention_required": true | false,
  "intervention_reason": "...",
  "flagged_teams": [...]
}
```

**Human:** `Reports/ecnl-migration-validation-{date}.md` (auto-generated summary for Presten)

---

## ECNL Transition Dedup Logic

### The Problem

During the transition period — when both TGS and GotSport are simultaneously emitting ECNL data — the dedup logic must bridge two incompatible source ID formats:

| Source | Source ID format | Example |
|--------|-----------------|---------|
| TGS (historical) | `tgs_{eventId}_{gameId}` | `tgs_3900_18234` |
| GotSport (new) | `gs_api_{eventId}_{matchId}` | `gs_api_31228_4587321` |

The TGS eventId and GotSport eventId are **not the same** — they are different platform IDs for the same real-world tournament. This means the standard dedup logic (which matches on `source_id` components) will NOT link these records automatically.

### The Transition-Period Dedup Pass

A one-time dedup pass is required, separate from the standard `dedup-and-link-teams.js` run:

**Input:** All TGS games + all new GotSport ECNL games  
**Match key:** `(home_team_name, away_team_name, game_date, home_score, away_score)` — fuzzy team name matching using existing `Team Name Matching` logic  
**Output:** For each matched pair, set TGS version `is_hidden = true`, `duplicate_of = {gotsport_game_id}`

**Script:** `scripts/ecnl-transition-dedup.js` (to be built as part of migration prep)

### Source ID Conflicts

No format conflict exists at the source_id level — `tgs_` and `gs_api_` prefixes are different. The risk is content collision, not identifier collision. Both records for the same game will exist in the database with different `source_id` values until the transition dedup pass runs.

### Recommended Execution Sequence

```
1. Run validate-ecnl-migration.js --mode=pre      [DONE — 2026-04-23]
2. ECNL migration goes live on GotSport           [Target: June 2026]
3. Wait 7-14 days for GotSport to populate ECNL game data
4. Run validate-ecnl-migration.js --mode=post     [Identifies orphaned records]
5. Review post-migration report with Presten
6. If intervention thresholds NOT exceeded: run ecnl-transition-dedup.js
7. Verify dedup results: TGS games correctly hidden, GotSport games canonical
8. Run ranking recalculation for affected teams
9. Re-run validate-ecnl-migration.js --mode=post  [Final verification]
```

> [!warning] Do NOT run the transition dedup pass before the post-migration validation
> The validation must run while orphaned TGS records are still visible (not yet hidden by dedup). Running dedup first would mask orphaned records, making it impossible to know which TGS games failed to find a GotSport counterpart.

---

## What "Successful Migration" Looks Like

| Metric | Success Threshold | Intervention Threshold |
|--------|------------------|----------------------|
| Teams with confirmed GotSport link | >95% | <90% |
| TGS games with GotSport counterpart | >97% | <95% |
| Teams with rating shift >25 pts | <2% | >5% |
| Ambiguous team merges | <3% | >8% |

A migration is "successful" if all four metrics clear their success thresholds. Any intervention threshold breach requires Presten review before proceeding.

---

## Immediate Status

- [x] **Pre-migration snapshot captured:** `Logs/ecnl-migration/pre-migration-snapshot-2026-04-23.json`
- [x] **Validation plan written** (this document)
- [ ] **`scripts/validate-ecnl-migration.js`** — script to be implemented (spec above is complete enough to write from)
- [ ] **`scripts/ecnl-transition-dedup.js`** — build after migration goes live, before running dedup
- [ ] **Post-migration run** — schedule for first week of June 2026

---

## Files Written

- `Reports/ecnl-migration-validation-plan.md` (this document)
- `Logs/ecnl-migration/pre-migration-snapshot-2026-04-23.json` (baseline data)

## References

- [[TGS AthleteOne]] — source being replaced
- [[GotSport]] — source taking over ECNL data
- [[Dedup Strategy]] — dedup logic and `team_merges` structure
- [[Data Sources Overview]] — source priority table
- [[Strategic Insights]] — Insight 1 (ECNL migration windfall), Action Item 2
- [[Database Schema]] — `games`, `teams`, `rankings`, `team_merges` tables

---

*FORGE — 2026-04-23*
