---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: completed
completed: 2026-04-24
priority: urgent
due: 2026-04-25
deliverable: "02 - Tiger Tournaments/Projects/Reports/ga-aspire-tier-fix-2026-04-24.md"
---

# Task: Write Executable GA ASPIRE Tier Fix SQL

## Context

The April 28 Escalation Reference Card (`Product & Planning/April 28 Escalation Reference Card.md`) has Section 3 flagged empty. FORGE confirmed the GA ASPIRE UPDATE SQL does not exist at any expected path:

- `Reports/ga-aspire-tier-fix-2026-04-[date].md` — does not exist
- `Rankings/calibration-fix-pre-deploy-2026-04-23.md` — covers U12/U13/U14/U19 calibration only; no GA ASPIRE UPDATE

The approval for this fix is already in place (`pending-2026-04-23-ga-calibration-review.md`, answered "Approved"). The analysis exists at `Reports/ga-coverage-audit-results-2026-04-23.md`. The missing piece is the executable SQL.

FORGE explicitly flagged this in its 11:16 briefing: "ELO owns the calibration tier definitions — this is ELO's SQL to write." SENTINEL confirms: this is ELO's task. It must be done before April 25 so FORGE can integrate it into the reference card before April 28.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Reports/ga-aspire-tier-fix-2026-04-24.md`

---

### Section 1: Fix Summary

```
Date approved: 2026-04-23
Approved by: Presten (see pending-2026-04-23-ga-calibration-review.md)
Fix type: event_tier reclassification (ga → ga_aspire)
Scope: events where event_name ILIKE '%ASPIRE%' AND event_tier = 'ga'
Expected affected teams: 350–1,050
Calibration change: 140 → 100 (Girls GA ASPIRE events, overcalibrated by ~40pts)
Session estimate: 45–60 minutes including validation
```

---

### Section 2: Pre-Fix Verification (Run First)

Before running the UPDATE, confirm the scope:

```sql
-- How many events will be reclassified?
SELECT COUNT(*) AS events_to_reclassify, event_tier
FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga'
GROUP BY event_tier;
```

Expected: some number of rows with `event_tier = 'ga'`. If 0 rows: the events may already have been reclassified or the name pattern is wrong. Stop and investigate before proceeding.

```sql
-- Which specific events will be affected?
SELECT id, event_name, event_tier
FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga'
ORDER BY event_name;
```

Review the event names. Confirm they are GA ASPIRE-level events, not mismatches (e.g., an event named "Aspiring Stars" that is a recreational tournament should be excluded by adding it to a WHERE exclusion list).

---

### Section 3: Boys GA ASPIRE — Explicit Handling (Read Before Running)

**This UPDATE is gender-agnostic.** The `events` table does not store gender — it stores games across genders. Any event with "ASPIRE" in its name and `event_tier = 'ga'` will be reclassified to `ga_aspire` regardless of whether it hosted Boys games, Girls games, or both.

Check whether Boys GA ASPIRE events exist:

```sql
SELECT g.gender, COUNT(DISTINCT g.id) AS game_count
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_name ILIKE '%ASPIRE%'
  AND e.event_tier = 'ga'
GROUP BY g.gender;
```

**Interpret results:**

- **If Boys games: 0** — UPDATE proceeds as planned. Boys are not affected. Note this result in the session log.
- **If Boys games: >0** — Boys GA ASPIRE events exist. Before proceeding, confirm how `compute-rankings.js` handles `event_tier = 'ga_aspire'` for Boys. See `task-2026-04-24-ga-aspire-boys-calibration-decision.md` for the Boys-specific calibration decision. Do NOT run the UPDATE until Boys calibration handling is confirmed.

---

### Section 4: The UPDATE (Run After Pre-Fix Verification Passes)

```sql
-- Reclassify GA ASPIRE events
UPDATE events
SET event_tier = 'ga_aspire'
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga';
```

After running: note the number of rows updated. This number should match the `events_to_reclassify` count from Section 2.

---

### Section 5: Post-Fix Verification

Immediately after the UPDATE:

```sql
-- Confirm no GA ASPIRE events remain tagged as 'ga'
SELECT COUNT(*) AS still_tagged_ga
FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga';
-- Expected: 0
```

```sql
-- Confirm new ga_aspire events exist
SELECT COUNT(*) AS now_tagged_aspire
FROM events
WHERE event_tier = 'ga_aspire';
-- Expected: same count as rows updated above
```

After confirming the UPDATE applied correctly: **recompute rankings.** The calibration value change (140 → 100 for Girls GA ASPIRE) does not take effect until rankings are recomputed. Schedule recompute immediately after the UPDATE session.

---

### Section 6: Rollback (If Needed Within 48 Hours)

```sql
-- Revert: reclassify ga_aspire back to ga
UPDATE events
SET event_tier = 'ga'
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga_aspire';
```

Rollback is clean as long as no other events have been manually assigned `event_tier = 'ga_aspire'` since the initial UPDATE. Recompute rankings after rollback.

---

## Definition of Done

- File written at `02 - Tiger Tournaments/Projects/Reports/ga-aspire-tier-fix-2026-04-24.md`
- All six sections present: summary, pre-fix verification, Boys handling, UPDATE SQL, post-fix verification, rollback
- Boys case is explicit: either confirmed as "0 Boys games affected" (with the SQL to verify) or flagged with a hard STOP if Boys games exist and calibration is unresolved
- Summary in briefing: "GA ASPIRE SQL written at `Reports/ga-aspire-tier-fix-2026-04-24.md`. Boys case: [0 Boys games confirmed safe / Boys games exist — calibration decision needed before April 28]."

## Why This Matters

FORGE cannot complete Section 3 of the April 28 Reference Card without this SQL. Presten cannot execute the GA ASPIRE fix on April 28 without Section 3. This is the single blocking item for the April 28 deadline. It is also the item SENTINEL has flagged in every briefing since April 23. The approval is done. The analysis is done. The only remaining step is writing the SQL file — a 45-minute task.

## References

- `02 - Tiger Tournaments/Projects/Reports/ga-coverage-audit-results-2026-04-23.md` — GA ASPIRE analysis
- `10 - Agents/SENTINEL/Queue/pending-2026-04-23-ga-calibration-review.md` — approval on file
- `02 - Tiger Tournaments/Projects/Product & Planning/April 28 Escalation Reference Card.md` — where this SQL will be cited
- `10 - Agents/FORGE/Briefings/2026-04-24-11:16.md` — FORGE's flag confirming SQL is missing
- `task-2026-04-24-ga-aspire-boys-calibration-decision.md` — Boys calibration companion task
