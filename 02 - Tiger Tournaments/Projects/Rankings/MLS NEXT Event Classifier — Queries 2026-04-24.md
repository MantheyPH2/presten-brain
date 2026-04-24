---
title: MLS NEXT Event Classifier — Queries 2026-04-24
tags: [mlsnext, calibration, event-classification, sql, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: FORGE
status: ready-for-execution
task: task-2026-04-24-mlsnext-event-classifier
for: ELO — Presten executes
---

# MLS NEXT Event Classifier — Execution Queries

**Purpose:** Classify all `mlsnext`-tiered events into `mlsnext_academy` vs `mlsnext_homegrown` using event name patterns. Required before ELO can apply the calibration split (Homegrown 160 / Academy 135) from [[MLS NEXT Tier Split Analysis 2026-04-22]].

**Who runs this:** Presten, against `youth_soccer` at localhost:5432.
**Decision target:** Apply the classification UPDATE only if Step A shows ≤10% of mlsnext game volume is unclassified.

---

## Schema Note

The `event_tiers` table maps event names to tier classifications. The `games` table has an `event_tier` column but event-level reclassification operates on `event_tiers`. If no `event_tiers` table exists, adapt all queries to target `games.event_tier` directly — see the fallback variant at the bottom of each step.

---

## Step 1: Run Step A — Count Events by Proposed Tier

This tells you whether the classification quality is good enough to proceed.

```sql
-- Step A: Count mlsnext events and games by proposed tier
SELECT
  CASE
    WHEN et.event_name ILIKE '%homegrown%'                         THEN 'mlsnext_homegrown'
    WHEN et.event_name ILIKE '%fall premier%'                      THEN 'mlsnext_homegrown'
    WHEN et.event_name ILIKE '%spring premier%'                    THEN 'mlsnext_homegrown'
    WHEN et.event_name ILIKE '%mls next fest%'                     THEN 'mlsnext_homegrown'
    WHEN et.event_name ILIKE '%mls next showcase%'                 THEN 'mlsnext_homegrown'
    WHEN et.event_name ILIKE '%national academy championship%'     THEN 'mlsnext_homegrown'
    WHEN et.event_name ILIKE '%mls next national%'                 THEN 'mlsnext_homegrown'
    WHEN et.event_name ILIKE '%premier league%'                    THEN 'mlsnext_homegrown'
    WHEN et.event_name ILIKE '%premier division%'                  THEN 'mlsnext_homegrown'
    WHEN et.event_name ILIKE '%academy division%'                  THEN 'mlsnext_academy'
    WHEN et.event_name ~ '(?i)division\s+\d'                       THEN 'mlsnext_academy'
    WHEN et.event_name ILIKE '%regional%'                          THEN 'mlsnext_academy'
    WHEN et.event_name ~ '(?i)(southwest|southeast|northeast|northwest|midwest|mountain|great lakes|mid.?atlantic|far west|south central|plains)\s+\d'
                                                                   THEN 'mlsnext_academy'
    WHEN et.event_name ~ '(?i)(south|north|east|west|central)\s+\d' THEN 'mlsnext_academy'
    ELSE 'mlsnext_unclassified'
  END AS proposed_tier,
  COUNT(DISTINCT et.event_name) AS event_count,
  SUM(game_counts.game_count)   AS total_games
FROM event_tiers et
JOIN (
  SELECT event_name, COUNT(*) AS game_count
  FROM games
  WHERE is_hidden = false
  GROUP BY event_name
) game_counts ON game_counts.event_name = et.event_name
WHERE et.event_tier = 'mlsnext'
GROUP BY proposed_tier
ORDER BY total_games DESC;
```

**Interpret the result:**

| Outcome | Action |
|---|---|
| `mlsnext_unclassified` total_games < 10% of grand total | **Proceed to Step 2** |
| `mlsnext_unclassified` total_games ≥ 10% of grand total | **STOP — flag to ELO for manual review before applying UPDATE** |

Expected result based on prior analysis: ~43% Homegrown, ~45% Academy, ~4–6% unclassified.

---

## Step 2: Interpret — Go / No-Go

Calculate: `unclassified_games / total_games × 100`

- **If < 10%:** Continue to Steps 3, 4, 5.
- **If ≥ 10%:** Stop. Post findings to ELO. The pattern set needs expansion before UPDATE should run. Do not proceed to Step 5.

---

## Step 3: Run Step B — Sample Events Per Tier (Sanity Check)

Confirm the classification is labeling real events correctly before committing the UPDATE.

```sql
-- Step B: Sample 5 events per proposed tier for sanity check
SELECT
  CASE
    WHEN event_name ILIKE '%homegrown%'                         THEN 'mlsnext_homegrown'
    WHEN event_name ILIKE '%fall premier%'                      THEN 'mlsnext_homegrown'
    WHEN event_name ILIKE '%spring premier%'                    THEN 'mlsnext_homegrown'
    WHEN event_name ILIKE '%mls next fest%'                     THEN 'mlsnext_homegrown'
    WHEN event_name ILIKE '%mls next showcase%'                 THEN 'mlsnext_homegrown'
    WHEN event_name ILIKE '%national academy championship%'     THEN 'mlsnext_homegrown'
    WHEN event_name ILIKE '%mls next national%'                 THEN 'mlsnext_homegrown'
    WHEN event_name ILIKE '%premier league%'                    THEN 'mlsnext_homegrown'
    WHEN event_name ILIKE '%premier division%'                  THEN 'mlsnext_homegrown'
    WHEN event_name ILIKE '%academy division%'                  THEN 'mlsnext_academy'
    WHEN event_name ~ '(?i)division\s+\d'                       THEN 'mlsnext_academy'
    WHEN event_name ILIKE '%regional%'                          THEN 'mlsnext_academy'
    WHEN event_name ~ '(?i)(southwest|southeast|northeast|northwest|midwest|mountain|great lakes|mid.?atlantic|far west|south central|plains)\s+\d'
                                                                THEN 'mlsnext_academy'
    WHEN event_name ~ '(?i)(south|north|east|west|central)\s+\d' THEN 'mlsnext_academy'
    ELSE 'mlsnext_unclassified'
  END AS proposed_tier,
  event_name,
  event_tier AS current_tier
FROM event_tiers
WHERE event_tier = 'mlsnext'
ORDER BY proposed_tier, event_name
LIMIT 45;
```

**What to look for:** Are the Homegrown rows real top-tier events? Are the Academy rows real regional division games? Flag anything that looks misclassified before running the UPDATE.

---

## Step 4: Run Step C — List Unclassified Events

Review every mlsnext event that doesn't match either pattern. These need manual assignment or will carry the 147 midpoint calibration.

```sql
-- Step C: All unclassified mlsnext events (manual review candidates)
SELECT
  et.event_name,
  COUNT(g.id) AS game_count
FROM event_tiers et
JOIN games g ON g.event_name = et.event_name AND g.is_hidden = false
WHERE et.event_tier = 'mlsnext'
  AND et.event_name NOT ILIKE '%homegrown%'
  AND et.event_name NOT ILIKE '%fall premier%'
  AND et.event_name NOT ILIKE '%spring premier%'
  AND et.event_name NOT ILIKE '%mls next fest%'
  AND et.event_name NOT ILIKE '%mls next showcase%'
  AND et.event_name NOT ILIKE '%national academy championship%'
  AND et.event_name NOT ILIKE '%mls next national%'
  AND et.event_name NOT ILIKE '%premier league%'
  AND et.event_name NOT ILIKE '%premier division%'
  AND et.event_name NOT ILIKE '%academy division%'
  AND et.event_name NOT ~ '(?i)division\s+\d'
  AND et.event_name NOT ILIKE '%regional%'
  AND et.event_name NOT ~ '(?i)(southwest|southeast|northeast|northwest|midwest|mountain|great lakes|mid.?atlantic|far west|south central|plains)\s+\d'
  AND et.event_name NOT ~ '(?i)(south|north|east|west|central)\s+\d'
GROUP BY et.event_name
ORDER BY game_count DESC
LIMIT 50;
```

Share this output with ELO. They may recognize patterns that can extend the classifier.

---

## Step 5: Apply Classification UPDATE (Only if Step A passed ≥90% threshold)

Run only after confirming Step A shows < 10% unclassified game volume.

```sql
-- Step 5A: Update Homegrown events
UPDATE event_tiers
SET event_tier = 'mlsnext_homegrown'
WHERE event_tier = 'mlsnext'
  AND (
    event_name ILIKE '%homegrown%'
    OR event_name ILIKE '%fall premier%'
    OR event_name ILIKE '%spring premier%'
    OR event_name ILIKE '%mls next fest%'
    OR event_name ILIKE '%mls next showcase%'
    OR event_name ILIKE '%national academy championship%'
    OR event_name ILIKE '%mls next national%'
    OR event_name ILIKE '%premier league%'
    OR event_name ILIKE '%premier division%'
  );

-- Step 5B: Update Academy events
UPDATE event_tiers
SET event_tier = 'mlsnext_academy'
WHERE event_tier = 'mlsnext'
  AND (
    event_name ILIKE '%academy division%'
    OR event_name ~ '(?i)division\s+\d'
    OR event_name ILIKE '%regional%'
    OR event_name ~ '(?i)(southwest|southeast|northeast|northwest|midwest|mountain|great lakes|mid.?atlantic|far west|south central|plains)\s+\d'
    OR event_name ~ '(?i)(south|north|east|west|central)\s+\d'
  );

-- Step 5C: Update remaining mlsnext events to mlsnext_unclassified
UPDATE event_tiers
SET event_tier = 'mlsnext_unclassified'
WHERE event_tier = 'mlsnext';
-- Note: after 5A and 5B, only the genuinely ambiguous events remain with 'mlsnext'
-- This gives them the explicit 'mlsnext_unclassified' label (calibration 147)
```

**Run 5A first, then 5B, then 5C.** Order matters — the first two UPDATEs will leave only the ambiguous events for 5C.

---

## Step 6: Confirm — Re-run Step A

After applying the UPDATE, re-run Step A. Expected result:

| proposed_tier | event_count | total_games |
|---|---|---|
| mlsnext_homegrown | ~320 | ~11,900 |
| mlsnext_academy | ~330 | ~12,400 |
| mlsnext_unclassified | ~35 | ~1,100 |
| mlsnext | 0 | 0 |

The last row must be 0. If any events still have `event_tier = 'mlsnext'` after Step 5C, they were not caught by the WHERE clause — investigate.

---

## Calibration Map Update (for Presten / FORGE)

After the UPDATE runs, update Phase 8 of `compute-rankings.js`:

```javascript
const CALIBRATION = {
  mlsnext_homegrown:     160,  // unchanged — confirmed by win-rate analysis
  mlsnext_academy:       135,  // reduced from 160 — 58.2% win rate vs ECNL implies ~15-20pt gap
  mlsnext_unclassified:  147,  // weighted midpoint (Academy 60% + Homegrown 40%)
  mlsnext:               160,  // legacy fallback — should have 0 events after reclassification
  // other tiers unchanged
};
```

**Apply this calibration change in the May 18–20 window** alongside the Brier score fixes. Do NOT apply before DSS (May 16) — rating shifts during an active managed event window are disruptive.

---

## Fallback: If No `event_tiers` Table

If only `games.event_tier` exists (no separate `event_tiers` table), adapt Step A as:

```sql
SELECT
  CASE
    WHEN event_name ILIKE '%homegrown%' THEN 'mlsnext_homegrown'
    WHEN event_name ILIKE '%academy division%' OR event_name ~ '(?i)division\s+\d' THEN 'mlsnext_academy'
    ELSE 'mlsnext_unclassified'
  END AS proposed_tier,
  COUNT(*) AS game_count
FROM games
WHERE event_tier = 'mlsnext'
  AND is_hidden = false
GROUP BY proposed_tier
ORDER BY game_count DESC;
```

And adapt the UPDATE to target `games.event_tier` directly.

---

## References

- [[MLS NEXT Tier Split Analysis 2026-04-22]] — win-rate analysis supporting the 160/135 split
- `Reports/mlsnext-event-classification-2026-04-23.md` — full classifier script (ELO, April 23)
- [[Database Schema]] — `event_tiers` and `games` table structure
- [[Ranking Engine]] — Phase 8 calibration map
