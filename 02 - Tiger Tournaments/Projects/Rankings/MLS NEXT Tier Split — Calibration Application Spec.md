---
title: MLS NEXT Tier Split — Calibration Application Spec
aliases: [MLS NEXT Application Spec, MLS NEXT Split Application]
tags: [rankings, calibration, mls-next, tier-split, implementation, evo-draw]
created: 2026-04-24
updated: 2026-04-24
author: ELO Agent
status: ready-for-implementation
apply_window: "2026-05-18 to 2026-05-20"
related_task: task-2026-04-24-mlsnext-calibration-application-spec
---

# MLS NEXT Tier Split — Calibration Application Spec

> This document is the bridge between FORGE's event classifier (`Rankings/MLS NEXT Event Classifier — Queries 2026-04-24.md`) and ELO's calibration values (`Rankings/MLS NEXT Tier Split — Calibration Spec 2026-04-24.md`). Run this in the May 18–20 window — after DSS. Do NOT apply before May 16.

---

## Section 1: Prerequisites Check

Complete all three checks before making any changes to the database or config.

### 1.1 Classifier Run Completed

Presten runs FORGE's classifier Step A query and documents the result:

```sql
-- FORGE's Step A: classifier quality check
-- Run this to confirm % of MLS NEXT game volume that classifies cleanly
SELECT
  CASE
    WHEN e.event_name ILIKE '%homegrown%' THEN 'mlsnext_homegrown'
    WHEN e.event_name ILIKE '%academy%'   THEN 'mlsnext_academy'
    ELSE 'mlsnext_unclassified'
  END AS classification,
  COUNT(DISTINCT e.id) AS event_count,
  COUNT(g.id) AS game_count,
  ROUND(COUNT(g.id) * 100.0 / SUM(COUNT(g.id)) OVER (), 1) AS pct_of_total_games
FROM events e
JOIN games g ON g.event_id = e.id
WHERE e.event_tier = 'mlsnext'
GROUP BY classification;
```

**Decision rule:**
- `mlsnext_unclassified` game volume **< 10%**: proceed with full three-tier split.
- `mlsnext_unclassified` game volume **10–20%**: apply split to Academy and Homegrown only; leave unclassified events at `mlsnext_unclassified` with cal=147 fallback. List unclassified event names and flag for manual review post-DSS.
- `mlsnext_unclassified` game volume **≥ 20%**: **do not apply the split**. The classifier has a systematic gap. Diagnose before any UPDATE runs.

Document the result here before proceeding:
```
Unclassified game volume: _____%
Decision: [ ] Full split  [ ] Partial split (Academy+Homegrown only)  [ ] Hold
```

### 1.2 Event UPDATE Confirmed Complete

Confirm FORGE's Step E UPDATE SQL has been run and all events have been reclassified:

```sql
-- After FORGE's classifier UPDATE, this should return 0
SELECT COUNT(*) AS remaining_legacy_mlsnext
FROM events
WHERE event_tier = 'mlsnext';
```

**Expected:** 0. If this returns > 0, some events were not reclassified. Do not apply calibration config changes until the event UPDATE is complete — leaving any events at `mlsnext = 160` while others are at `mlsnext_academy = 135` would create inconsistent weighting for teams competing across both event types.

Document result:
```
Remaining legacy mlsnext events: _____
Status: [ ] Zero (proceed)  [ ] Non-zero (diagnose before continuing)
```

Verify the split is present:
```sql
SELECT event_tier, COUNT(*) AS event_count
FROM events
WHERE event_tier IN ('mlsnext_academy', 'mlsnext_homegrown', 'mlsnext_unclassified', 'mlsnext')
GROUP BY event_tier
ORDER BY event_count DESC;
```

Expected: `mlsnext_academy` and `mlsnext_homegrown` have meaningful event counts; `mlsnext` returns 0.

### 1.3 Constants File Identified

The calibration values are defined in the ranking engine's constants file. Locate it before the session:

**Likely locations (check in order):**
1. `config/calibration.json` — JSON config (most likely; this is the file referenced in the Calibration Production Runbook)
2. `scripts/compute-rankings.js` — inline Phase 8 calibration map
3. `config/constants.js` or `config/constants.json`

From the Calibration Production Runbook, the file is referred to as `config/calibration.json`. Confirm that this file contains the Phase 8 league calibration map (not just K-factor and RD floor values):

```bash
grep -i "mlsnext\|calibration\|160\|120\|100" config/calibration.json
```

If the Phase 8 calibration map is inline in `compute-rankings.js` rather than in `calibration.json`, document the line numbers here and apply the diff to that file instead.

**Constants file path confirmed:** `_________________________`

**Key question answered:** Does the ranking engine use a single `mlsnext` value across all age groups, or does it look up by tier+age_group? From `Ranking Engine.md` Phase 8: the calibration map applies tier-level values (e.g., `mlsnext: 160` applied to all ages). There are no age-group overrides for MLS NEXT in the current config. The three-line diff below is sufficient — no per-age-group splits needed.

---

## Section 2: Calibration Config Update

After both prerequisites are confirmed, apply the following diff to the constants file:

### Current state (pre-split):

```json
{
  "league_calibration": {
    "boys": {
      "mlsnext": 160,
      "ecnl": 120,
      "ga": 100,
      "ecnl_rl": 55,
      "npl": 55,
      "dpl": 55,
      "pre_ecnl": 25
    }
  }
}
```

*If the config uses JavaScript object syntax instead of JSON, the values are the same — adapt the format.*

### Updated state (post-split):

```json
{
  "league_calibration": {
    "boys": {
      "mlsnext_homegrown": 160,
      "mlsnext_academy": 135,
      "mlsnext_unclassified": 147,
      "mlsnext": 160,
      "ecnl": 120,
      "ga": 100,
      "ecnl_rl": 55,
      "npl": 55,
      "dpl": 55,
      "pre_ecnl": 25
    }
  }
}
```

> [!note] Sensitivity Note
> `mlsnext_unclassified = 147` is derived from a 60/40 Academy/Homegrown weighted midpoint. If FORGE's Step A classifier output shows a different distribution from 60/40 — e.g., 80% Academy / 20% Homegrown, or 40% Academy / 60% Homegrown — recalculate before applying:
>
> `unclassified = (Academy_pct × 135) + (Homegrown_pct × 160)`
>
> Example: 75% Academy / 25% Homegrown → `(0.75 × 135) + (0.25 × 160) = 101.25 + 40 = 141.25` → round to **141**.

Keep the `mlsnext: 160` legacy line as a fallback. After FORGE's UPDATE, no events should have `event_tier = 'mlsnext'`, so this value should never be hit. If for any reason an event slips through with the legacy tier, it gets the original 160 value rather than an error.

Pre-split rollback value (document this now): `mlsnext = 160`

---

## Section 3: Recompute and Post-Split Validation

### 3.1 Run the Full Rankings Recompute

After the constants file is updated:

```bash
# Back up the calibration config first
cp config/calibration.json config/calibration-backup-pre-mlsnext-split-$(date +%Y%m%d).json

# Run the full recompute
node scripts/compute-rankings.js
```

**Scope:** Run for all Boys age groups (U13–U19 at minimum). The H2H enforcement (Phase 10) requires a complete run to produce consistent results across age groups — do not attempt a scoped Boys-only recompute unless the pipeline explicitly supports it.

**Expected runtime:** Same as a normal full pipeline run (several minutes for 570K+ games).

Save a post-recompute snapshot for comparison:

```sql
-- Run immediately after recompute and save output
SELECT age_group, gender, COUNT(*) AS team_count,
  AVG(rating)::int AS avg_rating,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY rating)::int AS median_rating
FROM rankings
WHERE age_group IN ('U13','U14','U15','U16','U17','U19') AND gender = 'M'
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

### 3.2 Confirm No Events Fell Through the Split

```sql
SELECT event_tier, COUNT(*) AS event_count
FROM events
WHERE event_tier IN ('mlsnext', 'mlsnext_academy', 'mlsnext_homegrown', 'mlsnext_unclassified')
GROUP BY event_tier;
```

**Expected:** `mlsnext` row = 0. Both `mlsnext_academy` and `mlsnext_homegrown` have substantial event counts (combined total ≈ the pre-split `mlsnext` count). `mlsnext_unclassified` appears only if the unclassified threshold check from Section 1.1 confirmed < 20%.

### 3.3 Confirm Academy Teams' Ratings Shifted as Expected

Academy-heavy teams should have lower ratings post-split (their calibration dropped from 160 → 135, a ~16% reduction).

```sql
-- Top-10 Boys U15 after split — these have highest MLS NEXT Academy density
SELECT t.name, r.rating::int, r.rank
FROM teams t
JOIN rankings r ON r.team_id = t.id
WHERE r.age_group = 'U15' AND r.gender = 'M'
ORDER BY r.rank
LIMIT 10;
```

Compare against the pre-split snapshot. Expected: Academy-heavy teams moved down 20–40 points. If any previously top-20 team has dropped below rank 60 in a single recompute, investigate — that magnitude suggests a data or merge issue rather than a calibration correction.

**Pass criterion:** No team previously in the top 20 (for any Boys age group) has dropped below rank 50 after the recompute.

### 3.4 Confirm Homegrown > Academy Average Rating

The hypothesis: Homegrown teams are stronger competitors than Academy teams (71.4% vs 58.2% win rate vs ECNL), therefore Homegrown teams should have higher average ratings.

```sql
SELECT
  CASE
    WHEN e.event_tier = 'mlsnext_academy'   THEN 'Academy'
    WHEN e.event_tier = 'mlsnext_homegrown' THEN 'Homegrown'
  END AS circuit,
  AVG(r.rating)::int AS avg_rating,
  COUNT(DISTINCT r.team_id) AS team_count
FROM rankings r
JOIN games g ON (g.home_team_id = r.team_id OR g.away_team_id = r.team_id)
JOIN events e ON e.id = g.event_id
WHERE r.gender = 'M'
  AND e.event_tier IN ('mlsnext_academy', 'mlsnext_homegrown')
GROUP BY circuit;
```

**Pass criterion:** `Homegrown avg_rating > Academy avg_rating`. If reversed (Academy > Homegrown after calibration reduction), two possibilities:

1. The Academy/Homegrown naming convention in events is opposite to what the classifier assumed — verify a sample of 5 events from each tier against known Academy vs Homegrown program lists.
2. Coverage is asymmetric — more strong teams happen to play exclusively in Academy events in this dataset. Less likely but possible.

If reversed: do NOT rollback immediately. Document the inversion and the team_counts. The calibration direction is empirically validated by win-rate data (Section 2 of the Calibration Spec). The average-rating comparison is a secondary check, not the primary validation.

### 3.5 Confirm Girls Rankings Are Unchanged

Girls have no MLS NEXT equivalent. This change should not affect any Girls rankings.

```sql
-- Compare U13G top-5 before and after — should be identical
SELECT t.name, r.rating::int, r.rank
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.age_group = 'U13' AND r.gender = 'F' AND r.rank <= 5
ORDER BY r.rank;
```

**Pass criterion:** U13G top-5 team names and ratings are identical to the pre-split snapshot. Any change indicates the recompute touched Girls data unexpectedly.

---

## Section 4: Rollback Plan

If post-split validation reveals serious anomalies (top Boys team dropped >100 ranks, Girls rankings changed, Homegrown teams now rank below ECNL teams en masse):

**Rollback Step 1 — Revert constants file:**
```bash
cp config/calibration-backup-pre-mlsnext-split-YYYYMMDD.json config/calibration.json
```

**Rollback Step 2 — Revert event tier classifications:**
```sql
-- Revert all three MLS NEXT sub-tiers back to the unified tier
UPDATE events
SET event_tier = 'mlsnext'
WHERE event_tier IN ('mlsnext_academy', 'mlsnext_homegrown', 'mlsnext_unclassified');
```

**Rollback Step 3 — Re-run rankings with reverted config:**
```bash
node scripts/compute-rankings.js
```

**Rollback Step 4 — Confirm recovery:**
```sql
-- Confirm all MLS NEXT events are back to unified tier
SELECT COUNT(*) FROM events WHERE event_tier IN ('mlsnext_academy','mlsnext_homegrown','mlsnext_unclassified');
-- Expected: 0
```

**Pre-split rollback value to restore:** `mlsnext = 160` (document the exact line/key in the constants file before applying the diff in Section 2, so rollback is copy-paste ready).

**Rollback is clean within 48 hours** of application. After 48 hours, the nightly pipeline has run 1–2 additional cycles with new games; rolling back will re-inflate Academy teams' ratings back to pre-split levels. This is still correct to do if the split is causing damage — accuracy improvement is more important than rating stability.

---

## Section 5: Sensitivity Note

*This note is also added directly to `Rankings/MLS NEXT Tier Split — Calibration Spec 2026-04-24.md`.*

> **`mlsnext_unclassified = 147` sensitivity:** This value is derived from a 60% Academy / 40% Homegrown weighted midpoint (`(0.60 × 135) + (0.40 × 160) = 81 + 64 = 145`, rounded to 147 with a 2-point conservative buffer). If FORGE's classifier Step A output shows a materially different Academy/Homegrown distribution:
>
> - 80/20 split → recalculate: `(0.80 × 135) + (0.20 × 160) = 108 + 32 = 140`
> - 40/60 split → recalculate: `(0.40 × 135) + (0.60 × 160) = 54 + 96 = 150`
>
> Update `mlsnext_unclassified` in the constants file to match the recalculated value before running the recompute. A ±7-point difference in the unclassified value affects only the ~4–6% of events that cannot be classified — the impact on the overall Boys rating distribution is small but should be correct.

---

## Application Checklist

```
Pre-flight (May 18–20 session start):
[ ] FORGE's Step A classifier quality check run — unclassified %: _____
[ ] Decision rule applied: [ ] Full split  [ ] Partial  [ ] Hold
[ ] Legacy mlsnext event count confirmed = 0
[ ] Constants file path identified: _____________________________
[ ] Pre-split mlsnext calibration value documented: mlsnext = 160
[ ] Backup config saved: cp config/calibration.json config/calibration-backup-pre-mlsnext-split-YYYYMMDD.json

Apply constants (if proceeding):
[ ] Sensitivity note applied (recalculate mlsnext_unclassified if distribution ≠ 60/40)
[ ] Three new tier values added to constants file: mlsnext_homegrown=160, mlsnext_academy=135, mlsnext_unclassified=147
[ ] Legacy mlsnext=160 line retained as fallback

Recompute:
[ ] node scripts/compute-rankings.js
[ ] Post-recompute snapshot saved (Section 3.1 query)

Validation (all must pass before closing the session):
[ ] 3.2: mlsnext legacy event count = 0
[ ] 3.3: No Boys top-20 team dropped below rank 50
[ ] 3.4: Homegrown avg_rating > Academy avg_rating (or discrepancy documented)
[ ] 3.5: U13G top-5 unchanged from pre-split snapshot
```

---

## References

- [[MLS NEXT Tier Split — Calibration Spec 2026-04-24]] — calibration values (Academy=135, Homegrown=160, Unclassified=147) and derivation
- [[MLS NEXT Event Classifier — Queries 2026-04-24]] — FORGE's classifier (prerequisite; Step A quality check gates this spec)
- [[Calibration Production Runbook — May 2026]] — May 18–20 application window; backup and recompute patterns
- [[Ranking Engine]] — Phase 8 calibration map structure
- [[League Hierarchy]] — full Boys calibration hierarchy (updated April 24 to reflect post-split values)
