---
title: Post-April-28 Fix Verification
type: verification-spec
author: ELO
date: 2026-04-24
status: ready
execution-window: 2026-04-28 (last 15 minutes of session)
tags: [verification, ga-aspire, ecnl, april-28, post-fix]
---

# Post-April-28 Fix Verification

> **Run this after:** GA ASPIRE UPDATE complete + ECNL dedup fix applied + rankings recompute finished.
> **Time budget:** 15 minutes. Run all queries, check all boxes.

---

## Section 1: GA ASPIRE Fix — Calibration Verification

Confirm that GA ASPIRE events' calibration changed correctly after recompute:

```sql
-- Top 10 teams that played primarily in GA ASPIRE events (Girls)
SELECT t.name, r.rating, r.rank, r.age_group, r.gender
FROM rankings r
JOIN teams t ON t.id = r.team_id
WHERE r.gender = 'F'
  AND r.team_id IN (
    SELECT DISTINCT g.home_team_id
    FROM games g
    JOIN events e ON e.id = g.event_id
    WHERE e.event_tier = 'ga_aspire'
  )
ORDER BY r.rating DESC
LIMIT 10;
```

**Pass criteria:**
- No Girls GA ASPIRE team has a rating above the top Girls GA teams in the same age group (ga_aspire cal=100 < ga cal=140 — ASPIRE teams should rank below regular GA teams with comparable records)
- No Girls GA ASPIRE team is rated above 1500 (this would suggest the fix didn't apply and they're still getting cal=140)
- No obvious anomalies: a GA ASPIRE team should not outrank a strong ECNL team of similar win-rate in the same age group

**Fail action:** If Girls GA ASPIRE teams appear to have retained inflated ratings (>1500 for teams with modest records), check whether the rankings recompute ran after the UPDATE. The fix only takes effect post-recompute.

---

## Section 2: GA ASPIRE Fix — Tier Assignment Verification

```sql
-- Confirm zero GA events with ASPIRE in name remain tagged as 'ga'
SELECT COUNT(*) AS remaining_ga_aspire
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
-- Expected: > 0
```

**Pass criteria:** First query returns 0. Second query returns the same count as the rows updated in the pre-fix verification (from `ga-aspire-tier-fix-2026-04-24.md` Section 2A).

**Fail action:** If first query returns > 0, the UPDATE did not complete. Check for transaction errors. Re-run the UPDATE from Section 4 of the fix SQL.

---

## Section 3: ECNL Dedup Fix — Verification

```sql
-- Confirm no unverified ECNL transition merges remain
SELECT COUNT(*) AS unverified_transitions
FROM team_merges
WHERE merge_type = 'ecnl_transition'
  AND (verified IS NULL OR verified = false);
-- Expected: 0
```

**Pass criteria:** Returns 0. All ECNL transition merges are verified.

**Fail action:** If returns > 0, the ECNL dedup fix did not apply correctly. Do **not** mark this fix as complete. Investigate the merge records returned: `SELECT * FROM team_merges WHERE merge_type = 'ecnl_transition' AND (verified IS NULL OR verified = false) LIMIT 20;`

---

## Section 4: Ranking Distribution — Sanity Check

After any calibration change and recompute, confirm no unexpected structural shifts:

```sql
SELECT
  age_group,
  gender,
  ROUND(AVG(rating), 0) AS avg_rating,
  ROUND(STDDEV(rating), 0) AS rating_std,
  COUNT(*) AS team_count
FROM rankings
WHERE age_group IN ('U13', 'U14') AND rank IS NOT NULL
GROUP BY age_group, gender
ORDER BY age_group, gender;
```

**Pass criteria:**
- U13G and U14G `avg_rating` should not have shifted by more than **15 points** from the pre-fix baseline
  - The GA ASPIRE fix affects a subset of teams (~350–1,050 out of potentially tens of thousands in those age groups). The overall average should barely move.
  - If `avg_rating` shifted by > 15 points for U13G or U14G: the UPDATE scope was wider than expected. Investigate whether events outside GA ASPIRE were reclassified.
- `rating_std` should not have changed by more than 20 points — the fix narrows the range for affected teams but should not dramatically change overall spread
- `team_count` should be identical to pre-fix (no teams should have been added or removed from rankings)

**Fail action:** If avg_rating shifted >15 points, run: `SELECT COUNT(*) FROM events WHERE event_tier = 'ga_aspire';` and compare to expected event count. If much higher than expected, the name-pattern match was too broad.

---

## Section 5: Boys GA ASPIRE — Confirmation

```sql
-- Confirm Boys GA ASPIRE handling
SELECT g.gender, COUNT(DISTINCT g.id) AS game_count
FROM games g
JOIN events e ON e.id = g.event_id
WHERE e.event_tier = 'ga_aspire'
GROUP BY g.gender;
```

Cross-reference against the Boys game_count from the pre-fix Boys check (Section 3 of `ga-aspire-tier-fix-2026-04-24.md`).

**If Boys game_count here matches the pre-fix Boys count:** Boys events were reclassified as expected. Per the Boys calibration decision (`Rankings/Boys GA ASPIRE Calibration Decision — April 2026.md`), this has zero net impact on Boys ratings — both `ga` and `ga_aspire` use cal=100 for Boys.

**If Boys game_count = 0 but pre-fix count was > 0:** Something unexpected happened. Investigate before closing the session.

**If Boys game_count = 0 and pre-fix count was also 0:** Boys were not affected. Expected outcome — note and close.

---

## Section 6: Session Completion Checklist

Run all checks above, then complete:

```
□ Section 2: GA ASPIRE events remaining as 'ga' = 0
□ Section 2: ga_aspire event count matches UPDATE row count
□ Section 3: ECNL unverified transitions = 0
□ Section 4: U13G avg_rating shift ≤ 15 points
□ Section 4: U14G avg_rating shift ≤ 15 points
□ Section 5: Boys GA ASPIRE count consistent with pre-fix check
□ All checks PASS — note session completion time: ___________

Fixes confirmed complete:
  ✅ GA ASPIRE tier reclassification (Girls: 140 → 100 effective)
  ✅ ECNL dedup fix (verified = true on all ecnl_transition merges)
```

If any box is unchecked, do not mark the fix as complete. Investigate before closing.

---

## References

- [[ga-aspire-tier-fix-2026-04-24]] — fix SQL (Sections 2–4 have the pre-fix baselines to compare against)
- `ecnl-dedup-verified-fix-2026-04-24.md` — ECNL P0 fix details
- [[April 28 Escalation Reference Card]] — the session plan this spec closes
- [[Boys GA ASPIRE Calibration Decision — April 2026]] — Boys cal=100 decision (Section 5 reference)
