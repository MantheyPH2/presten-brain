---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: high
due: 2026-05-02
deliverable: "02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split — Calibration Application Spec.md"
---

# Task: MLS NEXT Tier Split — Calibration Application Spec

## Context

Two pieces of the MLS NEXT tier split are complete:
1. **ELO's calibration spec** (`Rankings/MLS NEXT Tier Split — Calibration Spec 2026-04-24.md`): values are `mlsnext_academy = 135`, `mlsnext_homegrown = 160`, `mlsnext_unclassified = 147`.
2. **FORGE's classifier queries** (`Rankings/MLS NEXT Event Classifier — Queries 2026-04-24.md`): the SQL to classify events and the UPDATE statement to apply the classification.

What does not exist: the step that connects them. Once Presten runs FORGE's classifier and applies the UPDATE:
- The `event_tier` column changes for classified events
- But the calibration constant file (wherever K-factors and tier weights are stored) has not been updated to include `mlsnext_academy`, `mlsnext_homegrown`, and `mlsnext_unclassified` as distinct tiers
- And there is no validation spec to confirm the split improved calibration (or at minimum didn't break it)

This task: write the end-to-end application spec that takes FORGE's classifier output through calibration value deployment to post-split validation. Due May 2 — before the MLS NEXT classifier session is likely to run.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split — Calibration Application Spec.md`

---

### Section 1: Prerequisites Check

Before applying the calibration split, confirm:

**1.1 Classifier run completed**
- FORGE's classifier Step A query has been run and the unclassified % is documented
- If unclassified < 10% of game volume: proceed with full split application
- If unclassified ≥ 10%: do not apply the full split — apply `mlsnext_academy` and `mlsnext_homegrown` only to the classified events, leave `mlsnext_unclassified` as a literal fallback tier with value 147. Document the unclassified event names for manual review.

**1.2 UPDATE has been applied**
- FORGE's Step E UPDATE SQL has been run
- Confirm: `SELECT COUNT(*) FROM events WHERE event_tier = 'mlsnext'` should return 0 after the UPDATE (all classified events should have new tier names; if this returns > 0, some events were not reclassified)

**1.3 Calibration constants file identified**
- Locate the file where calibration values are defined (likely `ranking_config.js`, `constants.js`, `config.json`, or inline in `compute-rankings.js` — check `Infrastructure/Database Schema.md` or `Server and Frontend.md` for the constants file path)
- Document the file path here

---

### Section 2: Calibration Config Update

Once FORGE's classifier is applied and the constants file is identified:

Write the exact diff to add to the constants file:

```javascript
// Before (existing):
mlsnext: 147,  // or whatever the current unified value is

// After (new three-tier split):
mlsnext_academy: 135,    // MLS Next Academy circuits — validated per Spec 2026-04-24
mlsnext_homegrown: 160,  // MLS Next Homegrown circuits — unchanged
mlsnext_unclassified: 147, // Fallback for events not matching Academy/Homegrown pattern
```

Adapt the diff to the actual constants file syntax (JSON, JS object, YAML, etc.).

If the constants file contains per-age-group calibration overrides (e.g., `mlsnext_u13: 145`), note whether each override needs to be split into `mlsnext_academy_u13` and `mlsnext_homegrown_u13` equivalents — or whether the global split applies uniformly across age groups.

**Key question to answer:** Does the ranking engine apply a single `mlsnext` value, or does it look up by tier+age_group combination? Determine this from `Ranking Engine.md` and document the answer. If tier+age_group: more lines to add to constants. If tier only: the 3-line diff above is sufficient.

---

### Section 3: Post-Split Validation Queries

After the calibration config is updated and a full rankings recompute is run, run these validation queries:

**3.1 Confirm no events fell through the split**
```sql
SELECT event_tier, COUNT(*) AS event_count
FROM events
WHERE event_tier IN ('mlsnext', 'mlsnext_academy', 'mlsnext_homegrown', 'mlsnext_unclassified')
GROUP BY event_tier;
-- Expected: mlsnext row should be gone (0 count) or the fallback unclassified value
-- mlsnext_academy and mlsnext_homegrown should have meaningful event counts
```

**3.2 Confirm MLS NEXT teams' ratings shifted as expected**
```sql
-- Top-10 MLS NEXT Academy U13G teams after split
SELECT t.name, r.rating::int, r.rank
FROM teams t
JOIN rankings r ON r.team_id = t.id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND t.id IN (
    SELECT DISTINCT home_team_id FROM games g
    JOIN events e ON e.id = g.event_id
    WHERE e.event_tier = 'mlsnext_academy'
  )
ORDER BY r.rank
LIMIT 10;
```

Expected: Academy teams' ratings should be somewhat lower post-split (their calibration decreased from whatever unified value was in use to 135). If they increased, check whether the constants file update was applied.

**3.3 Confirm Homegrown teams' ratings relative to Academy teams**

The hypothesis in the calibration spec is that MLS NEXT Homegrown (cal=160) is harder than Academy (cal=135). Post-split, average Homegrown team ratings should be higher than average Academy team ratings if games are correctly attributed.

```sql
SELECT
  CASE
    WHEN e.event_tier = 'mlsnext_academy' THEN 'Academy'
    WHEN e.event_tier = 'mlsnext_homegrown' THEN 'Homegrown'
  END AS circuit,
  AVG(r.rating)::int AS avg_rating,
  COUNT(DISTINCT r.team_id) AS team_count
FROM rankings r
JOIN games g ON (g.home_team_id = r.team_id OR g.away_team_id = r.team_id)
JOIN events e ON e.id = g.event_id
WHERE r.age_group = 'U13' AND r.gender = 'F'
  AND e.event_tier IN ('mlsnext_academy', 'mlsnext_homegrown')
GROUP BY circuit;
```

**Pass criteria:** Homegrown avg_rating > Academy avg_rating. If reversed, the calibration values may be swapped (Academy and Homegrown definitions may be different from what the spec assumed).

---

### Section 4: Rollback Plan

If post-split ratings produce unexpected anomalies (top-ranked MLS NEXT team dropped below rank 1000, or Homegrown teams now rank significantly above ECNL teams):

**Rollback Step 1:** Revert the constants file to the pre-split unified `mlsnext` value
**Rollback Step 2:** `UPDATE events SET event_tier = 'mlsnext' WHERE event_tier IN ('mlsnext_academy', 'mlsnext_homegrown', 'mlsnext_unclassified')`
**Rollback Step 3:** Re-run `compute-rankings.js` with the reverted constants

Document the pre-split unified `mlsnext` value here so rollback doesn't require looking it up: `mlsnext = [X]` (pull from current constants file).

---

### Section 5: Sensitivity Note (Per SENTINEL Request)

Per SENTINEL's 08:52 review: add a note to the calibration spec that the `mlsnext_unclassified = 147` weighted midpoint value should be recalculated if the classifier shows a meaningfully different Academy/Homegrown distribution from the assumed 60/40 split.

Add this note directly to `Rankings/MLS NEXT Tier Split — Calibration Spec 2026-04-24.md` as well:

> "Note: `mlsnext_unclassified = 147` is derived from a 60/40 Academy/Homegrown weighted midpoint. If FORGE's classifier output shows a different distribution (e.g., 80/20 or 40/60), recalculate: `unclassified = (Academy_pct × 135) + (Homegrown_pct × 160)` and update this value before applying."

---

## Definition of Done

- Application spec written covering: prerequisites, constants file update, post-split validation, rollback
- Constants file path identified (or documented as "locate during pre-flight")
- Three validation queries are present and schema-correct
- Rollback steps are explicit and copy-paste ready
- Sensitivity note added to `MLS NEXT Tier Split — Calibration Spec 2026-04-24.md`
- Summary in briefing: "MLS NEXT calibration application spec written. Constants file: [path identified / to confirm]. Sensitivity note added to spec."

## Why This Matters

Without this spec, the MLS NEXT tier split is a design with no implementation path. FORGE's classifier changes the DB. ELO's spec has the calibration values. This document is the bridge: it tells Presten exactly what to update in the code, confirms the change worked, and provides a rollback if something breaks. The split is worthless without the application step.

## References

- `02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split — Calibration Spec 2026-04-24.md` — calibration values; add sensitivity note here
- `02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Event Classifier — Queries 2026-04-24.md` — FORGE's classifier (prerequisite); check unclassified % before applying
- `02 - Tiger Tournaments/Projects/Rankings/Ranking Engine.md` — constants file location and per-tier calibration structure
- `02 - Tiger Tournaments/Projects/Rankings/MLS NEXT Tier Split Analysis 2026-04-22.md` — original analysis and hypothesis being validated in Section 3.3
