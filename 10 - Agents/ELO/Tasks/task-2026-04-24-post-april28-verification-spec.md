---
type: agent-task
assigned_to: ELO
assigned_by: SENTINEL
date: 2026-04-24
status: pending
priority: medium
due: 2026-04-27
deliverable: "02 - Tiger Tournaments/Projects/Reports/ga-aspire-post-fix-verification-2026-04-28.md"
---

# Task: Post-April-28 GA ASPIRE Fix Verification Spec

## Context

The April 28 session will execute two fixes:
1. GA ASPIRE tier reclassification (UPDATE + ranking recompute)
2. ECNL dedup P0 fix (one-line code change)

Both fixes have pre-fix verification in their respective specs. What does not exist: a consolidated post-execution verification that Presten runs after both fixes are done and the ranking recompute completes. Without this, the April 28 session ends with the fixes applied but no formal confirmation that they worked correctly.

ELO owns the calibration verification domain. This task: write a short, executable post-fix verification spec that Presten runs in the last 15 minutes of the April 28 session.

## What to Write

Write: `02 - Tiger Tournaments/Projects/Reports/ga-aspire-post-fix-verification-2026-04-28.md`

This document is ops-mode: short, numbered steps, copy-paste queries, clear pass/fail criteria. No context, no background — Presten is running it after 75 minutes of fix work.

---

### Section 1: GA ASPIRE Fix — Calibration Verification

**Goal:** Confirm that GA ASPIRE teams' ratings moved from the overcalibrated level (~40pts high) toward the corrected level after recompute.

```sql
-- Sample: top 5 GA ASPIRE teams before vs after
-- (Run pre-fix version first if a snapshot was taken; otherwise use current rankings)
SELECT t.name, r.rating, r.rank, r.age_group, r.gender
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.team_id IN (
  SELECT DISTINCT g.home_team_id
  FROM games g
  JOIN events e ON e.id = g.event_id
  WHERE e.event_tier = 'ga_aspire'
)
ORDER BY r.rating DESC
LIMIT 10;
```

Pass criteria:
- No Girls GA ASPIRE team has a rating implying calibration > 110 (if Girls GA ASPIRE is now at 100, teams should cluster below strong GA teams, not above them)
- No obvious anomalies: a GA ASPIRE team should not be rated higher than a top ECNL team in the same age group

---

### Section 2: GA ASPIRE Fix — Tier Assignment Verification

```sql
-- Confirm zero GA events with "ASPIRE" in name remain
SELECT COUNT(*) AS remaining_ga_aspire_events
FROM events
WHERE event_name ILIKE '%ASPIRE%'
  AND event_tier = 'ga';
-- Expected: 0
```

```sql
-- Confirm ga_aspire tier is now populated
SELECT COUNT(*) AS aspire_events, MIN(event_name) AS sample_event
FROM events
WHERE event_tier = 'ga_aspire';
-- Expected: > 0 (at least as many as the UPDATE affected)
```

---

### Section 3: ECNL Dedup Fix — Verification

```sql
-- Confirm no unverified ECNL transition merges remain
SELECT COUNT(*) AS unverified_transitions
FROM team_merges
WHERE merge_type = 'ecnl_transition'
  AND (verified IS NULL OR verified = false);
-- Expected: 0
```

If this returns > 0: the ECNL fix did not apply correctly. Do not mark this fix as complete.

---

### Section 4: Ranking Distribution — Sanity Check

After any calibration change and recompute, run a distribution check to confirm no unexpected structural shifts:

```sql
SELECT
  age_group,
  gender,
  AVG(rating) AS avg_rating,
  STDDEV(rating) AS rating_std,
  COUNT(*) AS team_count
FROM rankings
WHERE age_group IN ('U13', 'U14') AND rank IS NOT NULL
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

Pass criteria:
- U13G and U14G `avg_rating` should not have changed by more than 15 points from the pre-fix baseline (the GA ASPIRE fix affects a subset of teams, not the whole distribution)
- If `avg_rating` shifted by > 15 points: investigate whether the UPDATE scope was wider than expected

---

### Section 5: Boys GA ASPIRE — Confirmation

```sql
-- Confirm Boys GA ASPIRE handling (whether they were reclassified or not)
SELECT g.gender, COUNT(DISTINCT g.id) AS game_count
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_tier = 'ga_aspire'
GROUP BY g.gender;
```

Cross-reference against the pre-fix Boys GA ASPIRE check from `task-2026-04-24-ga-aspire-update-sql.md`. If Boys game_count here matches what was in `event_tier = 'ga'` before, Boys events were reclassified. Confirm this was the intended outcome per the Boys calibration decision (`Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md`).

---

### Section 6: Session Completion Checklist

```
After all queries pass:
□ GA ASPIRE events: 0 remaining as 'ga' (Section 2)
□ ECNL transition merges: 0 unverified (Section 3)
□ Ranking distribution: avg_rating shift < 15pts for U13/U14 (Section 4)
□ Boys GA ASPIRE: handled per decision document (Section 5)
□ Session complete — note completion time in Presten Session Plans log
```

---

## Definition of Done

- Document written at `Reports/ga-aspire-post-fix-verification-2026-04-28.md`
- All six sections present with copy-paste SQL
- Pass/fail criteria stated for each check
- Completion checklist at the end (Presten should be able to check off 4 boxes and walk away)
- Document is short enough to execute in 15 minutes
- Summary in briefing: "Post-April-28 verification spec written. Covers: GA ASPIRE tier verification, ECNL dedup check, ranking distribution sanity, Boys GA ASPIRE confirmation."

## Why This Matters

The April 28 Reference Card gets Presten through the fixes. This spec gives Presten the last 15 minutes of the session: the confirmation that it worked. Without a formal post-fix check, the session ends on uncertainty. With it, Presten can mark both April 28 items as definitively closed.

## References

- `02 - Tiger Tournaments/Projects/Reports/ga-aspire-tier-fix-2026-04-24.md` — fix SQL (ELO companion task)
- `02 - Tiger Tournaments/Projects/Reports/ecnl-dedup-verified-flag-2026-04-24.md` — ECNL P0 fix spec
- `02 - Tiger Tournaments/Projects/Product & Planning/April 28 Escalation Reference Card.md` — the session this spec closes
- `02 - Tiger Tournaments/Projects/Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md` — Boys case decision
