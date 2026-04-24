---
title: GA ASPIRE Tier Fix — Executable SQL
type: fix-spec
author: ELO
date: 2026-04-24
status: ready-for-execution
execution-window: 2026-04-28
related: "[[Boys GA ASPIRE Calibration Decision — April 2026]]"
tags: [ga-aspire, calibration, fix, sql, april-28]
---

# GA ASPIRE Tier Fix — Executable SQL

> **Approved:** 2026-04-23 by Presten (see `pending-2026-04-23-ga-calibration-review.md`)
> **Ready to execute:** April 28, 2026 session
> **Estimated session time:** 45–60 minutes including validation

---

## Section 1: Fix Summary

| Field | Value |
|-------|-------|
| Date approved | 2026-04-23 |
| Approved by | Presten (see `pending-2026-04-23-ga-calibration-review.md`) |
| Fix type | event_tier reclassification (`ga` → `ga_aspire`) |
| Scope | Events where `event_name ILIKE '%ASPIRE%' AND event_tier = 'ga'` |
| Expected affected teams | 350–1,050 |
| Calibration change (Girls) | 140 → 100 (GA ASPIRE overcalibrated by ~40 pts) |
| Calibration change (Boys) | 100 → 100 (no net change; see Section 3) |
| Session estimate | 45–60 minutes including validation |

**Why this fix is needed:** GA ASPIRE launched February 2025 as a sub-tier of GA with lower competition level. All GA ASPIRE events are currently stored as `event_tier = 'ga'`, giving participating teams a calibration weight of 140 (Girls) — identical to full GA. The correct calibration is 100. Teams playing primarily in GA ASPIRE leagues are inflated by ~40 points relative to their true competitive level. This directly affects DSS division placement accuracy.

---

## Section 2: Pre-Fix Verification (Run First)

**Step 2A — Count events to be reclassified:**

```sql
-- How many events will be reclassified?
SELECT COUNT(*) AS events_to_reclassify, event_tier
FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga'
GROUP BY event_tier;
```

Expected: some number of rows with `event_tier = 'ga'`.
- If **0 rows:** Events may already have been reclassified or the name pattern is wrong. **STOP and investigate before proceeding.**
- If result looks right: continue to Step 2B.

**Step 2B — List the specific events to be reclassified:**

```sql
-- Which specific events will be affected?
SELECT id, event_name, event_tier
FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga'
ORDER BY event_name;
```

Review the event names manually. Confirm they are GA ASPIRE-level events. If any event is clearly not a GA ASPIRE competition (e.g., an event with "Aspiring Stars" in the name that is recreational), add it to a WHERE exclusion clause in Section 4 before running the UPDATE.

---

## Section 3: Boys GA ASPIRE — Explicit Handling (Read Before Running)

**This UPDATE is gender-agnostic.** The `events` table stores events, not games — Boys and Girls games often share the same event record. Any event with "ASPIRE" in its name and `event_tier = 'ga'` will be reclassified to `ga_aspire` regardless of whether it hosted Boys games, Girls games, or both.

**First, check whether Boys GA ASPIRE events exist:**

```sql
SELECT g.gender, COUNT(DISTINCT g.id) AS game_count
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_name ILIKE '%ASPIRE%'
  AND e.event_tier = 'ga'
GROUP BY g.gender;
```

**How to interpret results:**

- **Boys game_count = 0:** The UPDATE proceeds as planned. Boys are not affected. Note this result in your session log and continue to Section 4.
- **Boys game_count > 0:** Boys GA ASPIRE events exist and will be reclassified to `ga_aspire`.

**If Boys game_count > 0 — what happens to Boys ratings:**

The Boys GA calibration value is 100. The Boys GA ASPIRE calibration value is also 100 (same value; see [[Boys GA ASPIRE Calibration Decision — April 2026]]). Reclassifying Boys GA ASPIRE events from `event_tier = 'ga'` to `event_tier = 'ga_aspire'` will **not change Boys team ratings** because both tiers use the same calibration weight (100) for Boys in `compute-rankings.js`.

**Conclusion:** Whether Boys game_count is 0 or >0, it is safe to proceed with the UPDATE. If Boys games are reclassified, there is no net rating impact for Boys teams. Note the Boys game_count in your session log for the post-fix verification check.

---

## Section 4: The UPDATE (Run After Pre-Fix Verification Passes)

```sql
-- Reclassify GA ASPIRE events from 'ga' to 'ga_aspire'
UPDATE events
SET event_tier = 'ga_aspire'
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga';
```

After running: **note the number of rows updated.** This number must match the `events_to_reclassify` count from Section 2A.

If the row count does not match:
- Less than expected: some events may have already been updated, or the WHERE clause hit a race condition. Re-run Step 2A to confirm current state.
- More than expected: a broader pattern match may have caught unintended events. Run Step 2B again to check what was reclassified. Rollback (Section 6) if needed.

---

## Section 5: Post-Fix Verification

Run all three checks immediately after the UPDATE:

**5A — Confirm no GA ASPIRE events remain tagged as `ga`:**

```sql
SELECT COUNT(*) AS still_tagged_ga
FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga';
-- Expected: 0
```

**5B — Confirm ga_aspire tier is now populated:**

```sql
SELECT COUNT(*) AS now_tagged_aspire, MIN(event_name) AS sample_event
FROM events
WHERE event_tier = 'ga_aspire';
-- Expected: > 0, matching the rows updated in Section 4
```

**5C — Confirm Boys game handling (compare to Step 3 result):**

```sql
SELECT g.gender, COUNT(DISTINCT g.id) AS game_count
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_tier = 'ga_aspire'
GROUP BY g.gender;
```

Cross-reference against the boys game_count from Section 3. If Section 3 showed Boys game_count = 500, this query should show a similar Boys count. If it doesn't, investigate before recomputing rankings.

**After all three checks pass:**

Schedule or trigger a full rankings recompute (`node compute-rankings.js`). **The calibration value change (140 → 100 for Girls GA ASPIRE) does not take effect until rankings are recomputed.** The UPDATE only changes the event tier tag; the Elo calculation uses that tag during recompute.

---

## Section 6: Rollback (If Needed Within 48 Hours)

```sql
-- Revert: reclassify ga_aspire back to ga
UPDATE events
SET event_tier = 'ga'
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga_aspire';
```

Rollback is clean as long as no other events have been manually assigned `event_tier = 'ga_aspire'` since the initial UPDATE. After rollback, recompute rankings to restore prior calibration.

**Rollback trigger:** Use if post-fix verification (Section 5) shows an unexpected event count, or if the post-fix ranking distribution sanity check (see `ga-aspire-post-fix-verification-2026-04-28.md` Section 4) shows avg_rating shifting >15 points for U13/U14.

---

## References

- [[ga-coverage-audit-results-2026-04-23]] — GA ASPIRE analysis and affected team estimates
- `pending-2026-04-23-ga-calibration-review.md` — Presten approval on file
- [[April 28 Escalation Reference Card]] — session plan (this SQL is Section 3)
- [[Boys GA ASPIRE Calibration Decision — April 2026]] — Boys calibration handling detail
- `ga-aspire-post-fix-verification-2026-04-28.md` — post-execution verification (run after recompute)
