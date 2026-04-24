---
type: agent-task
assigned_to: FORGE
assigned_by: SENTINEL
date: 2026-04-24
status: completed
priority: high
due: 2026-04-27
completed: 2026-04-24
deliverable: Rankings/MLS NEXT Event Classifier — Queries 2026-04-24.md
---

# Task: MLS NEXT Event Name Classifier — Distinguish `mlsnext_academy` vs `mlsnext_homegrown`

## Context

ELO's MLS NEXT tier split analysis (`02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split Analysis 2026-04-22.md`) proposes splitting the `mlsnext` calibration tier into two: `mlsnext_academy` (cal=160, unchanged) and `mlsnext_homegrown` (cal=135, reduced). The rationale: MLS Next Homegrown leagues are softer competition than MLS Next Academy leagues, and lumping them together over-calibrates Homegrown teams.

ELO is blocked on this split because the split requires knowing which events in the `events` table are Academy events vs Homegrown events. The `event_tier` column currently marks all of them as `mlsnext`. ELO does not have DB access to run the classification query. That query is FORGE's job.

**This is a vault-level task.** You cannot run the query yourself, but you can write it precisely enough that when Presten runs it, the output immediately tells ELO whether the data supports the split.

## What You Need to Do

### Step 1: Document MLS NEXT Event Name Patterns

From what you know about the GotSport data and the USYS/MLS ecosystem:

MLS NEXT Academy events typically include phrases like:
- "MLS NEXT Academy"
- "MLS Next Academy"
- "MLSNEXT Academy"

MLS NEXT Homegrown events typically include phrases like:
- "MLS NEXT Homegrown"
- "MLS Next Homegrown"
- "Homegrown"

Document these patterns. Add any additional name variants you have encountered in the research docs:
- `02 - Tiger Tournaments/Projects/Data Sources/USA Rank Benchmark Scraper Design.md`
- `02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split Analysis 2026-04-22.md`

### Step 2: Write the Classification Query

Write the SQL query Presten will run to classify all `mlsnext`-tiered events:

```sql
-- Step A: Count events by name pattern to confirm classification quality
SELECT
  CASE
    WHEN event_name ILIKE '%academy%' THEN 'mlsnext_academy'
    WHEN event_name ILIKE '%homegrown%' THEN 'mlsnext_homegrown'
    ELSE 'mlsnext_unclassified'
  END AS proposed_tier,
  COUNT(*) AS event_count,
  SUM(game_count) AS total_games  -- or equivalent column
FROM events
WHERE event_tier = 'mlsnext'
GROUP BY proposed_tier
ORDER BY total_games DESC;
```

Adapt this query to match the actual `events` table schema from `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md`. In particular:
- Confirm the column name is `event_name` (not `name` or `event_title`)
- Confirm the column name is `event_tier` (not `tier`)
- Add a game count if the `events` table has a `game_count` column or if you need a JOIN to `games`

### Step 3: Write the Sample Events Query

Write a second query to surface sample event names for each proposed tier so ELO can sanity-check the classification:

```sql
-- Step B: Sample 5 event names per proposed tier
SELECT
  CASE
    WHEN event_name ILIKE '%academy%' THEN 'mlsnext_academy'
    WHEN event_name ILIKE '%homegrown%' THEN 'mlsnext_homegrown'
    ELSE 'mlsnext_unclassified'
  END AS proposed_tier,
  event_name,
  event_tier
FROM events
WHERE event_tier = 'mlsnext'
ORDER BY proposed_tier, event_name
LIMIT 30;
```

### Step 4: Write the Unclassified Events Query

Write a third query to isolate any `mlsnext` events that don't match either pattern — these will need manual review:

```sql
-- Step C: List all unclassified mlsnext events
SELECT event_name, COUNT(*) as occurrences
FROM events
WHERE event_tier = 'mlsnext'
  AND event_name NOT ILIKE '%academy%'
  AND event_name NOT ILIKE '%homegrown%'
GROUP BY event_name
ORDER BY occurrences DESC
LIMIT 50;
```

### Step 5: Write the UPDATE SQL (Conditional on Step A Results)

If Step A shows that the Academy/Homegrown split captures ≥90% of mlsnext events (i.e., `mlsnext_unclassified` is <10% of total games), write the UPDATE statement for Presten to apply:

```sql
-- Apply classification (only run after confirming Step A results are acceptable)
UPDATE event_tiers
SET event_tier = CASE
  WHEN event_name ILIKE '%academy%' THEN 'mlsnext_academy'
  WHEN event_name ILIKE '%homegrown%' THEN 'mlsnext_homegrown'
  ELSE event_tier  -- leave unclassified ones unchanged
END
WHERE event_tier = 'mlsnext';
```

Adapt to the actual table structure — the UPDATE may be on the `events` table directly or on a separate `event_tiers` table.

**Note the decision rule explicitly:** "Run the UPDATE only if Step A shows ≥90% of mlsnext game volume classifies cleanly (Academy + Homegrown combined). If unclassified is ≥10%, do not apply — flag to ELO for manual review."

### Step 6: Write the Deliverable

Write all queries and the decision rule to:
`02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Event Classifier — Queries 2026-04-24.md`

Structure the document as a linear checklist:
1. Run Step A (count by proposed tier)
2. Interpret: is unclassified <10%? If yes, proceed. If no, stop and flag to ELO.
3. Run Step B (sample events to sanity-check names)
4. Run Step C (list unclassified events for manual review)
5. If Step A passed: run the UPDATE
6. Re-run Step A to confirm 0 unclassified remain

## Definition of Done

- Queries in Steps A–C are written and schema-correct (no column name guesses)
- Decision rule is explicit: what threshold determines whether the UPDATE should run
- UPDATE SQL is present and conditional
- Deliverable document filed at `Rankings/MLS NEXT Event Classifier — Queries 2026-04-24.md`
- Summary in your briefing: queries are ready for Presten to run; classification quality unknown until execution

## Why This Matters

ELO's calibration split (mlsnext_academy 160 / mlsnext_homegrown 135) has been on hold since April 22 specifically because the data query to validate it doesn't exist. The split could improve calibration accuracy for ~40+ events across Boys and Girls age groups. This is a 30-minute vault task for FORGE that unblocks two weeks of ELO analysis.

## References

- `02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split Analysis 2026-04-22.md` — ELO's split proposal and expected calibration values
- `02 - Tiger Tournaments/Projects/Infrastructure/Database Schema.md` — `events` table schema
- ELO Briefing 2026-04-23 23:52 — "Autonomous" section, MLS NEXT classifier dependency noted
